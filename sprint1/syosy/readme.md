### 3. Design de Comunicação

#### 3.1. Análise dos Requisitos de Comunicação

Para este sistema de *Platoon Monitoring*, a arquitetura de comunicação divide-se em dois domínios principais, cada um com requisitos de rede e restrições de tempo muito distintos:

* **Comunicação Intra-veículo (Crítica e de Tempo Real):** Os módulos de tomada de decisão, particularmente o Controlo do Veículo (*Vehicle Control* - VC), necessitam de receber dados de vários sensores para decidir ações imediatas. Estas decisões ocorrem em intervalos curtos de 0.1s a 5s. Com base na representação do mundo à sua volta, o VC emite comandos diretos para os atuadores físicos do veículo (direção, travagem e *powertrain*). Este nível exige uma fiabilidade extrema e latência ultra-baixa.
* **Comunicação Inter-veículo (Monitorização e Gestão):** A aplicação exige que o veículo líder mantenha uma visão atualizada do estado de todos os veículos que compõem o *platoon*. Os veículos seguidores enviam dados do módulo de Manutenção Preditiva (*PredMaint*) para o módulo de Gestão do Platoon (*PlatMgmt*) no líder. Este nível exige tecnologias capazes de suportar comunicação sem fios adaptável a condições de rede em movimento.

---

#### 3.2. Seleção de Tecnologias de Comunicação e Justificação Física

A escolha das tecnologias baseia-se na alocação física das Unidades de Controlo Eletrónico (ECUs) e nas limitações matemáticas de largura de banda de cada barramento.

**Domínio Intra-veículo (Rede Interna)**
* **Automotive Ethernet (ECU Perceção $\rightarrow$ ECU Controlo):** O processamento de dados densos, como os provenientes de câmaras, exige uma largura de banda maciça. Se considerarmos uma única câmara a transmitir vídeo sem compressão a 720p e 30 frames por segundo (fps) em RGB (24 bits por pixel), o débito de dados necessário calcula-se da seguinte forma:
$$1280 \times 720 \text{ pixels} \times 24 \text{ bits/pixel} \times 30 \text{ fps} \approx 663.5 \text{ Mbps}$$
Como o CAN Bus standard está limitado a 1 Mbps, é fisicamente impossível transmitir estes dados por essa via. Logo, o Automotive Ethernet (ex: 1000BASE-T1 de 1 Gbps) é estritamente necessário.

* **CAN Bus (ECU Controlo $\rightarrow$ ECUs Atuadores):** A norma standard para controlo mecânico. O CAN garante determinismo com latências na ordem dos milissegundos. Embora a velocidade seja baixa (1 Mbps), o *payload* máximo de cada mensagem é de 8 bytes, o que é perfeitamente adequado e eficiente para enviar comandos simples (ex: ângulo de viragem ou pressão do travão).

**Domínio Inter-veículo (V2V)**
* **5G C-V2X (Sidelink):** Selecionado para a partilha de telemetria (JSON) entre camiões. Permite comunicação direta V2V com latências mínimas e capacidade para suportar o tamanho dinâmico dos ficheiros JSON gerados pela Manutenção Preditiva.

---

#### 3.3. Definição dos Mecanismos de Troca de Dados (Arquitetura Lógica)

Para coordenar o fluxo de dados entre os veículos do *platoon*, o paradigma **Publish-Subscribe**, implementado através do protocolo **MQTT**, foi o mecanismo escolhido. 
* **Eficiência e Assincronismo:** Os seguidores enviam dados apenas quando há atualizações, poupando largura de banda.
* **Arquitetura de Tópicos:** O módulo *PlatMgmt* no veículo líder aloja o *Broker* MQTT. O líder subscreve tópicos (ex: `platoon/+/predmaint`) para receber dados de todos os seguidores de forma unificada.

---

#### 3.4. Especificação dos Modelos de Dados (JSON via MQTT / Ethernet)

As comunicações de alto nível (entre a ECU de Manutenção Preditiva e a ECU de Gestão de Platoon) utilizam **JSON**, modelado com base nas normas *FIWARE Smart Data Models*.

**Payload JSON: Estado de Manutenção Preditiva (ECU 4 $\rightarrow$ ECU 7)**
```json
{
  "id": "urn:ngsi-ld:Vehicle:Truck-Follower-02",
  "type": "VehiclePredictiveMaintenance",
  "timestamp": "2026-06-03T21:50:00Z",
  "telemetry": {
    "engineTemperature": { "value": 92.0, "unit": "Celsius" },
    "tirePressure": { "rearLeft": 7.5, "unit": "Bar", "status": "WARNING" }
  },
  "overallHealthStatus": "WARNING"
}
```

#### 3.5. Mapeamento de Cargas Físicas Críticas (Tramas CAN)

Ao contrário das comunicações V2V, a comunicação interna de controlo (ECU de Controlo → ECUs de Atuação) **não utiliza JSON**, de forma a não exceder o limite restrito de 8 bytes (64 bits) do protocolo *CAN Bus*. O modelo de dados é mapeado ao nível do bit.

Trama CAN: Comando de Direção (ECU 5 → ECU 8 - Steering ECU)
Para caber nos 8 bytes físicos do CAN Bus, a mensagem estruturada pelo Vehicle Control distribui-se da seguinte forma:
* **Byte 0-1 (16 bits):** Ângulo de Viragem Requerido (−720º a +720º com resolução de 0.1º).
* **Byte 2-3 (16 bits):** Binário Requerido (Torque do motor de assistência à direção).
* **Byte 4 (8 bits):** Estado do Comando (0x00 = Inativo, 0x01 = Ativo, 0x02 = Emergência).
* **Byte 5 (8 bits):** Contador de Ciclo (Alive Counter para garantir que a ligação não falhou).
* **Byte 6-7 (16 bits):** Checksum (CRC) para validação da integridade da mensagem.
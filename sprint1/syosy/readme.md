### 3. Design de Comunicação

#### 3.1. Análise dos Requisitos de Comunicação

Para este sistema de *Platoon Monitoring*, a arquitetura de comunicação divide-se em dois domínios principais, cada um com requisitos de rede e restrições de tempo muito distintos:

* **Comunicação Intra-veículo (Crítica e de Tempo Real):** Os módulos de tomada de decisão, particularmente o Controlo do Veículo (*Vehicle Control* - VC), necessitam de receber dados de vários sensores para decidir ações imediatas. Estas decisões de curto prazo ocorrem em intervalos curtos de 0.1s a 5s. Com base na representação do mundo à sua volta, o VC emite comandos diretos para os atuadores físicos do veículo, nomeadamente a direção, a travagem e o motor (*powertrain*). Este nível exige uma fiabilidade extrema e latência ultra-baixa.
* **Comunicação Inter-veículo (Monitorização e Gestão):** A aplicação exige que o veículo líder mantenha uma visão atualizada do estado de todos os veículos que compõem o *platoon*. Isto obriga à partilha de informação não apenas entre os subsistemas internos de um veículo, mas também entre veículos diferentes. Os veículos seguidores têm de enviar os dados do seu módulo de Manutenção Preditiva (*PredMaint*, como a temperatura do motor, pressão dos pneus e estado dos travões) para o módulo de Gestão do Platoon (*PlatMgmt*) residente no líder. Este nível exige tecnologias capazes de suportar comunicação sem fios adaptável a condições de rede em movimento.

**Tabela de Requisitos de Fluxos de Dados (Feeds)**

| Domínio | Origem da Informação | Destino da Informação | Requisitos Operacionais |
| :--- | :--- | :--- | :--- |
| **Intra-veículo** | Sensores (Câmaras, LiDAR, GPS) | *Vehicle Control* (VC) | Latência mínima e alta largura de banda para mapeamento do mundo. |
| **Intra-veículo** | *Vehicle Control* (VC) | Atuadores (Travões, Direção) | Tempo real crítico (resposta entre 0.1s e 5s) e extremo determinismo. |
| **Inter-veículo** | *PredMaint* (Seguidores) | *Platoon Management* (Líder) | Fiabilidade na entrega e capacidade de operar em movimento contínuo. |

---

#### 3.2. Seleção de Tecnologias de Comunicação

Com base na análise de requisitos operacionais, foram avaliadas e selecionadas as seguintes tecnologias de rede e transporte de dados para garantir a segurança e a sincronização do *platoon*:

**Domínio Intra-veículo (Rede Interna)**
* **CAN Bus (Controller Area Network):** A norma standard da indústria automóvel para os sistemas de controlo. É a escolha ideal para a comunicação entre o módulo VC e os atuadores críticos. Garante determinismo (o tempo máximo de entrega da mensagem é garantido), um fator não negociável para o controlo mecânico.
* **Automotive Ethernet:** O CAN Bus tem limitações de velocidade que o tornam inadequado para transmitir vídeo. O Automotive Ethernet foi selecionado para ligar os sensores de alta densidade (como Câmaras e LiDAR) ao módulo de controlo, oferecendo a largura de banda (Gigabit) necessária para o processamento de imagem em tempo real.

**Domínio Inter-veículo (V2V - Vehicle-to-Vehicle)**
* **5G C-V2X (Cellular Vehicle-to-Everything):** Para a partilha de telemetria e avisos de manutenção entre os camiões seguidores e o líder, selecionou-se a tecnologia C-V2X. Especificamente, a funcionalidade *Sidelink* (interface PC5) permite que os camiões comuniquem diretamente uns com os outros com latências da ordem de milissegundos, sem precisarem de passar o tráfego pelas antenas das operadoras móveis locais.
* **Alternativa / Fallback - IEEE 802.11p (ITS-G5):** Uma tecnologia fiável baseada na norma Wi-Fi e desenhada especificamente para veículos em movimento. Servirá como protocolo de redundância (backup) caso haja constrangimentos na implementação do hardware 5G.

**Resumo da Avaliação Tecnológica**

| Ligação (Feed) | Tecnologia Selecionada | Justificação Principal |
| :--- | :--- | :--- |
| **Sensores $\rightarrow$ VC** | Automotive Ethernet | Elevado débito de dados (largura de banda) para *streams* de perceção espacial. |
| **VC $\rightarrow$ Atuadores** | CAN Bus | Fiabilidade comprovada, resistência a interferências e latência ultra-baixa. |
| **Seguidores $\rightarrow$ Líder**| 5G C-V2X (Sidelink) | Comunicação V2V direta, robusta a altas velocidades e baixa latência. |

#### 3.3. Definição dos Mecanismos de Troca de Dados (Arquitetura)

Para coordenar o fluxo de dados entre os veículos do *platoon*, foi avaliado o paradigma *Request-Response* (como o protocolo HTTP/REST) em contraste com o paradigma *Publish-Subscribe* (Pub/Sub). 

Para este cenário, o paradigma **Publish-Subscribe**, implementado através do protocolo **MQTT (Message Queuing Telemetry Transport)**, foi o mecanismo escolhido. Esta decisão baseia-se nos seguintes fatores:
* **Eficiência e Assincronismo:** Ao contrário do REST, que exigiria que o veículo líder fizesse perguntas constantes (*polling*) a cada seguidor ("Como está a tua pressão dos pneus?"), o MQTT permite que os seguidores enviem dados apenas quando há atualizações ou de forma assíncrona.
* **Baixo Overhead:** O MQTT possui um cabeçalho de mensagem extremamente leve, o que poupa largura de banda na rede sem fios (C-V2X / 802.11p).
* **Resiliência de Rede:** É ideal para cenários de mobilidade onde a ligação pode sofrer micro-cortes, dado que o *Broker* gere a entrega das mensagens.

**Arquitetura de Tópicos (Topic Tree):**
O módulo de Gestão do Platoon (*PlatMgmt*) no veículo líder aloja (ou está diretamente ligado a) o *Broker* MQTT. Os veículos seguidores atuam como publicadores (*Publishers*) e o líder como subscritor (*Subscriber*).
* Exemplo de tópico para publicação: `platoon/veiculo_<ID>/predmaint`
* O líder subscreve o tópico usando *wildcards* para ouvir todos os seguidores simultaneamente: `platoon/+/predmaint`

---

#### 3.4. Especificação dos Modelos de Dados (Estrutura das Mensagens)

Para garantir a interoperabilidade do sistema (permitindo que camiões de marcas diferentes integrem o mesmo *platoon*), os modelos de dados devem seguir uma linguagem comum e normalizada.

O formato escolhido para a serialização dos dados é o **JSON (JavaScript Object Notation)**. A estrutura das propriedades baseia-se nas normas da indústria, nomeadamente nos **FIWARE Smart Data Models** (focados em *Smart Aeronautics and Automotive*), adaptados para os requisitos específicos de Manutenção Preditiva do nosso projeto.

**Exemplo de Payload: Mensagem de Estado de Manutenção Preditiva**
Esta mensagem é gerada pelo módulo *PredMaint* de um veículo seguidor e enviada via MQTT para o líder, reportando o estado de parâmetros críticos como a pressão dos pneus e a temperatura do motor.

```json
{
  "id": "urn:ngsi-ld:Vehicle:Truck-Follower-02",
  "type": "VehiclePredictiveMaintenance",
  "timestamp": "2026-06-02T20:30:00Z",
  "platoonId": "Platoon-Alpha-7",
  "telemetry": {
    "speed": {
      "value": 85.5,
      "unit": "km/h"
    },
    "engineTemperature": {
      "value": 92.0,
      "unit": "Celsius",
      "status": "NORMAL"
    },
    "tirePressure": {
      "frontLeft": 8.1,
      "frontRight": 8.0,
      "rearLeft": 7.5,
      "rearRight": 7.6,
      "unit": "Bar",
      "status": "WARNING",
      "alertMessage": "Low pressure detected on rear axle."
    }
  },
  "overallHealthStatus": "WARNING"
}
```
**Mapeamento de Atributos Críticos:**
id e type: Identificadores normalizados do veículo e do tipo de mensagem.
timestamp: Carimbo de tempo exato da leitura (crucial para detetar atrasos na rede).
status: Variável de estado (NORMAL, WARNING, CRITICAL) que permite ao módulo de Platoon Management do líder processar regras rapidamente (ex: acionar uma travagem de emergência do platoon se o estado for CRITICAL).
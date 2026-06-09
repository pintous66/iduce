## 3.7 Análise Temporal das Transferências de Dados

A arquitetura do sistema exige que as restrições temporais (*deadlines*) sejam estritamente cumpridas para garantir a segurança do pelotão. O requisito fundamental da aplicação estabelece que a tomada de decisão a curto prazo do Controlo do Veículo (VC) opera num intervalo de 0,1 s a 5 s. Consequentemente, o sistema deve garantir uma latência *end-to-end* máxima inferior a 100 milissegundos (0,1 s) para ações de resposta imediata ou manobras críticas de emergência (ex.: encostar o pelotão à berma).

### 3.7.1 Latência no Domínio Intra-veículo (Caminho Crítico de Decisão)

Para ações imediatas locais, os dados fluem dos sensores para os atuadores através do módulo VC.

#### Sensores → ECU de Perceção → ECU VC

A utilização de Automotive Ethernet (ex.: 1000BASE-T1 a 1 Gbps) garante que o transporte de grandes volumes de dados densos (como vídeo de câmaras) não cria estrangulamentos. A latência de transmissão de um pacote nesta rede situa-se confortavelmente na ordem dos sub-milissegundos, suportando os requisitos rigorosos de tempo real de veículos definidos por software [Ref 1].

#### Processamento RTOS (FreeRTOS)

A adoção de um escalonamento preemptivo baseado em prioridades garante o determinismo. O tempo de processamento da decisão de emergência sofre um atraso mínimo, visto que a tarefa de Controlo do Veículo pode interromper processos menos prioritários quase de imediato e com um *overhead* residual provocado pelo núcleo do RTOS [Ref 2].

#### ECU VC → ECUs Atuadores (CAN Bus)

O barramento CAN opera na camada física (L1) e de ligação de dados (L2) a uma velocidade de 1 Mbps. Considerando que o *payload* máximo estruturado pelo VC para o CAN é de apenas 8 bytes, o tempo de transmissão física de uma trama de atuação de comando direto nos atuadores (direção, travagem, etc.) no pior cenário de transmissão (*worst-case*) é de exatamente 134 microssegundos [Ref 3].

#### Desempenho Global Intra-veículo

O tempo total desde a leitura do sensor até à receção física do comando pelo atuador resume-se a uns escassos milissegundos, cumprindo sem dificuldades o limiar inferior do *deadline* rígido de 100 ms.

### 3.7.2 Latência no Domínio Inter-veículo (Caminho de Manutenção Preditiva e Gestão)

O teste de *stress* à análise temporal ocorre no cenário em que uma anomalia (ex.: problema nos travões) detetada num veículo seguidor exige uma ação de travagem coordenada de todo o pelotão. O percurso dos dados distribui-se e orçamenta-se da seguinte forma:

#### Deteção e Envio (Seguidor)

A ECU PredMaint deteta o problema e passa a informação encapsulada em JSON para a ECU COMM via Ethernet interna (tempo estimado < 1 ms).

#### Transmissão V2V (5G NG-RAN)

O pacote é emitido para o exterior. A tecnologia 5G Next Generation Radio Access Network (NG-RAN) está otimizada para comunicações ultra-rápidas através do serviço URLLC (*Ultra-Reliable Low-Latency Communications*), oferecendo latências *end-to-end* garantidas na ordem de 1 milissegundo [Ref 4].

#### Processamento Protocolar (MQTT)

A transmissão assenta em MQTT, um protocolo da camada aplicacional desenvolvido especificamente para ser leve e altamente eficiente em cenários de IoT com recursos limitados. Apesar do *overhead* inevitável de qualquer protocolo aplicacional, a arquitetura *publish-subscribe* assíncrona encaminha a mensagem do seguidor para o tópico subscrito pelo líder sem dependências síncronas bloqueantes. O processamento pelo *Broker* adiciona um atraso residual, garantindo a baixa latência e estabilidade adequadas para o provisionamento de serviços V2X [Ref 5].

#### Decisão e Atuação (Líder)

O pacote é rececionado pela ECU PlatMgmt e retransmitido à ECU VC do líder. A lógica do VC é acionada e despacha a trama CAN de travagem/direção para a berma (< 1 ms).

#### Desempenho Global Inter-veículo

A soma acumulada dos tempos de processamento da ECU de origem, da travessia da rede de rádio 5G URLLC (~ 1 ms), do processamento do *Broker* MQTT e da atuação final via CAN Bus totaliza uma latência *end-to-end* típica que variará entre 5 ms e 15 ms.

Este ciclo completa-se amplamente dentro da janela exigida de 100 ms (0,1 s) definida para as reações de curto prazo. A arquitetura cumpre assim com folga as métricas de tempo real, viabilizando o isolamento de falhas críticas antes da ocorrência de um acidente físico no *platoon*.

### 3.7.3 Referências

**[Ref 1]** Texas Instruments. "How Ethernet accelerates the move to software-defined vehicles." Texas Instruments White Paper (SSZTD02), 2020. (Valida o uso de Single-Pair Ethernet a 1 Gbps para redes de veículos autónomos de forma a garantir largura de banda e baixíssima latência na transmissão de sensores de alta densidade).

**[Ref 2]** A. M. da Silva et al., "Analysis of FreeRTOS Overheads on Periodic Tasks," Anais do XVII Workshop em Desempenho de Sistemas Computacionais e de Comunicação (WPerformance), Sociedade Brasileira de Computação, 2018. (Comprova cientificamente a eficiência e o escalonamento preemptivo no FreeRTOS numa arquitetura ARM com atrasos irrisórios).

**[Ref 3]** Analog Devices. "iCoupler® Isolation in CAN Bus Applications (AN-770)." Analog Devices Application Note, 2012. (Mostra o cálculo matemático detalhado de que uma trama CAN de 8-bytes e identificador de 11 bits a 1 Mbps demora no pior cenário exatos 134 microssegundos).

**[Ref 4]** H. B. Yilmaz et al., "Ultra-Reliable Low-Latency Communications: Foundations, Enablers, System Design, and Evolution Towards 6G," IEEE Access, 2023. (Paper que descreve exaustivamente a métrica de 1 milissegundo de latência garantida imposta pelo serviço URLLC nas redes 5G).

**[Ref 5]** C. Campolo et al., "V2X Service Provisioning with 5G V2N2V Communications with Cross-Stakeholder Information Sharing," Proc. of the IEEE Vehicular Networking Conference (VNC), 2024. (Estuda e valida empiricamente a comunicação entre veículos apoiada por redes 5G e os delays residuais associados a corretores MQTT publish-subscribe em cenários críticos V2X).

---

## 3.8 Análise de Fiabilidade das Comunicações e Soluções de Mitigação

Num sistema de condução autónoma, a perda de pacotes ou falhas de comunicação podem ter consequências catastróficas. Para mitigar esses riscos, o desenho das comunicações incorpora múltiplas estratégias de resiliência distribuídas pelos diferentes níveis de rede.

### 3.8.1 Fiabilidade na Comunicação Inter-veículo (MQTT sobre 5G)

O canal de rádio é inerentemente instável, pelo que o uso de MQTT para a partilha de dados entre veículos e o líder (onde se aloja o Broker) recorre a mecanismos nativos de mitigação:

- **Qualidade de Serviço (QoS):** O MQTT suporta três níveis de QoS. Para alertas críticos de Manutenção Preditiva (ex.: falha iminente nos travões) serão usados QoS 1 (*At least once*) ou QoS 2 (*Exactly once*), garantindo a entrega da mensagem através de *buffers* e retransmissões.

- **Persistent Sessions:** Sessões persistentes permitem que o Broker retenha o estado de subscrição caso a ligação 5G de um seguidor caia temporariamente (ex.: passagem por um túnel). Mensagens retidas em fila de espera são entregues imediatamente quando a ligação é restabelecida.

- **Last Will and Testament (LWT):** Cada seguidor regista uma mensagem LWT ao conectar-se. Se um veículo perder a ligação de forma inesperada (avaria grave ou perda de alcance), o Broker deteta a quebra e publica imediatamente o LWT, sinalizando ao líder a falha e desencadeando a manobra de segurança.

### 3.8.2 Fiabilidade na Comunicação Intra-veículo (CAN Bus e Ethernet)

No domínio intra-veículo, o ambiente é ruidoso devido aos motores e inversores. A fiabilidade combina mecanismos físicos e lógicos adequados às restrições dos barramentos:

- **Validação de Integridade (Checksums / CRC):** Conforme o mapeamento de tramas CAN, os bytes 6 e 7 de cada comando crítico (ex.: direção) contêm um CRC-16 para detetar corrupções nos bits causadas por ruído eletromagnético. Tramas inválidas são sumariamente descartadas.

- **Monitorização de Estado (Alive Counters):** O byte 5 das tramas CAN funciona como um contador de vida (*alive counter*). Os microcontroladores dos atuadores esperam incrementos contínuos deste contador; se estagnar durante alguns ciclos, o atuador assume a perda de comunicação com o cérebro do camião e transita de forma autónoma para um Estado de Falha Segura (*Fail-Safe State*).

### 3.8.3 Matriz de Failover e Mitigação de Comunicações

A tabela seguinte resume os cenários de falha previstos na arquitetura de rede e os respetivos mecanismos de redundância (*failover*) ou mitigação aplicados:

| Meio de Comunicação | Cenário de Falha Identificado | Estratégia de Mitigação / Failover | Consequência no Sistema |
|---------------------|-------------------------------|------------------------------------|-------------------------|
| 5G C-V2X (V2V) | Perda de pacotes via rádio devido a interferências. | Injeção da mensagem crítica com MQTT QoS 1 ou 2. | O Broker exige confirmação de receção (ACK); caso contrário, a mensagem é retransmitida. |
| 5G C-V2X (V2V) | Perda temporária de sinal (ex.: zona de sombra ou túnel longo). | MQTT Persistent Sessions mantêm as subscrições ativas. | Dados são acumulados em buffer e sincronizados logo que o sinal regresse, sem necessitar de novo handshake. |
| 5G C-V2X (V2V) | Falha elétrica total do seguidor ou quebra abrupta e permanente da ligação. | Acionamento automático da mensagem MQTT Last Will and Testament (LWT). | O módulo PlatMgmt do líder interceta o aviso de "Morte" do nó e ordena ao VC a travagem/encosto imediato do pelotão. |
| CAN Bus | Corrupção de uma mensagem de atuação (ex.: Virar 15º) por ruído nos cabos. | Cálculo divergente do algoritmo CRC-16 nos bytes 6 e 7 da trama. | O microcontrolador local do atuador descarta a mensagem corrompida, prevenindo movimentos erráticos indesejados. |
| CAN Bus | Corte físico do barramento CAN ou falha fatal do Vehicle Control. | Estagnação da monitorização por Alive Counter (Watchdog) no byte 5. | *Fail-Safe Action* ativada por hardware: atuadores centram a direção e aplicam travagem controlada sem intervenção central. |

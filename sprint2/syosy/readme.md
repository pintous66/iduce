## 3.7 Análise Temporal das Transferências de Dados

A arquitetura do sistema exige que as restrições temporais (*deadlines*) sejam estritamente cumpridas para garantir a segurança do pelotão. O requisito fundamental da aplicação estabelece que a tomada de decisão a curto prazo do Controlo do Veículo (VC) opera num intervalo de **0,1 s a 5 s**. Consequentemente, o sistema deve garantir uma **latência end-to-end máxima inferior a 100 ms (0,1 s)** para ações de resposta imediata ou manobras críticas de emergência (ex.: encostar o pelotão à berma perante uma avaria detetada).

### Ajustes e feedback do sprint 1

Em resposta ao feedback recebido no primeiro sprint, a arquitetura de comunicação inter-veículo (V2V) foi profundamente reestruturada para eliminar o ponto único de falha (Single Point of Failure) introduzido pela dependência de um Broker MQTT centralizado no veículo líder. Para satisfazer os rigorosos requisitos de segurança e tempo real críticos, a coordenação do pelotão transitou para a middleware descentralizada DDS (Data Distribution Service) operando sobre 5G PC5 Sidelink, tirando agora partido de políticas estritas de QoS (como Liveliness e Deadlines) e substituindo o overhead do formato JSON por uma serialização binária previsível. O uso de MQTT e JSON foi assim restrito de forma exclusiva ao canal V2N (Vehicle-to-Network), sendo agora utilizados apenas para o envio assíncrono de telemetria de manutenção preditiva não crítica para a Cloud, separando claramente o domínio de controlo de tempo real do domínio de gestão de retaguarda.

### 3.7.1 Latência no Domínio Intra-veículo (Caminho Crítico de Decisão)

Para ações imediatas locais, os dados fluem dos sensores para os atuadores através do módulo VC.

#### Sensores → ECU de Perceção → ECU VC

A utilização de **Automotive Ethernet** (ex.: 1000BASE-T1 a 1 Gbps) garante que o transporte de grandes volumes de dados densos (como vídeo de câmaras) não cria estrangulamentos na rede interna. A latência de transmissão de um pacote nesta rede com arquitetura baseada em **TSN (Time-Sensitive Networking)** situa-se confortavelmente na ordem dos sub-milissegundos, suportando os requisitos determinísticos da indústria automóvel [1].

#### Processamento RTOS (FreeRTOS)

A adoção de um escalonamento preemptivo baseado em prioridades garante o determinismo. O tempo de processamento da decisão de emergência sofre um atraso mínimo, visto que a tarefa de Controlo do Veículo pode interromper processos menos prioritários quase de imediato e com um *overhead* de troca de contexto irrisório imposto pelo núcleo do RTOS [2].

#### ECU VC → ECUs Atuadores (CAN Bus)

O barramento CAN opera a uma velocidade de **1 Mbps**. O *payload* máximo estruturado pelo VC para o CAN é de apenas **8 bytes**. Aplicando a fórmula matemática da transmissão com inserção de bits de enchimento (*bit-stuffing*), o tempo de transmissão física de uma trama de atuação no pior cenário (*worst-case response time*) é de exatamente **134 µs** [3]. O limiar inferior do *deadline* rígido de **100 ms** é assim cumprido com facilidade.

### 3.7.2 Latência no Domínio Inter-veículo Crítico (Caminho de Coordenação do Platoon)

O teste de *stress* temporal ocorre quando uma anomalia grave (ex.: problema de travões num seguidor) exige uma ação coordenada do pelotão. Para garantir fiabilidade militar, o sistema descarta a dependência de um *Broker MQTT* centralizado para o canal crítico **V2V (Vehicle-to-Vehicle)**.

#### Transmissão V2V Direta (5G NR-V2X PC5 Sidelink)

A comunicação inter-veículo assenta na interface **PC5 Sidelink do 5G**. Esta tecnologia permite a comunicação direta (*Device-to-Device*) sem necessitar da infraestrutura da rede celular, fornecendo latências ultra-baixas na ordem de **1 a 5 ms** exigidas para o *platooning* autónomo [4].

#### Middleware de Tempo Real (DDS)

Em vez do MQTT, a partilha crítica de estados utiliza a *middleware* **DDS (Data Distribution Service)**. Sendo um protocolo totalmente descentralizado (*brokerless*), elimina o ponto único de falha no veículo líder. O DDS liga diretamente ECUs publicadoras e subscritoras, otimizando o fluxo de dados em veículos autónomos cooperativos [5].

#### Serialização Binária

Para V2V crítico, o uso de JSON foi abandonado face ao seu *overhead* de processamento e tamanho de variável de alocação. O sistema implementa o formato de representação binária nativa do DDS (**CDR – Common Data Representation**), que assegura um tamanho fixo do pacote e um tempo de *parsing* absolutamente previsível.

#### Desempenho Global

A latência *end-to-end* (desde a deteção do problema no seguidor → 5G Sidelink → DDS → VC do Líder → Atuadores via CAN) decorre numa janela altamente previsível de **5 ms a 10 ms**, mitigando totalmente as preocupações de segurança temporal.

### 3.7.3 Telemetria Não Crítica (Manutenção Preditiva Cloud / Backend)

A modelação em JSON sobre o protocolo MQTT é mantida exclusivamente para o canal **V2N (Vehicle-to-Network)**. Os registos de manutenção preditiva de longo prazo são enviados assincronamente para a *Cloud* da empresa de frotas. Neste domínio, o peso do *parsing* do JSON e a arquitetura *publish-subscribe* centralizada do MQTT não afetam a dinâmica física do pelotão em estrada.

---

## 3.8 Análise de Fiabilidade, Segurança e Políticas de QoS

Num sistema de condução autónoma interdependente, a falha de comunicação ou a ausência de mecanismos de QoS granulares têm consequências catastróficas. A transição para o modelo DDS sobre 5G Sidelink resolve as limitações clássicas de topologias centralizadas.

### 3.8.1 Políticas de QoS no V2V Crítico e Deteção de Perdas

A coordenação do pelotão não depende de sessões persistentes em *brokers* opacos, sendo gerida ativamente por políticas explícitas de **Qualidade de Serviço (QoS)** nativas do DDS.

#### Garantias de Liveliness (Vivacidade)

A política **LIVELINESS** do DDS monitoriza ativamente a presença dos membros no barramento virtual. Se um veículo seguidor perder a ligação (por quebra do *link* rádio), o sistema do veículo líder aciona instantaneamente a manobra de segurança.

#### Garantias de Deadline

A política **DEADLINE** contratualiza uma taxa de atualização rigorosa. Se o seguidor falhar o envio da sua telemetria física dentro de uma janela de **50 ms**, a aplicação de Controlo do Veículo é alertada imediatamente via *callbacks*.

#### Prioridades de Transporte

A política **TRANSPORT_PRIORITY** garante que comandos de travagem de emergência antecedam hierarquicamente dados operacionais de menor urgência na interface rádio do 5G.

### 3.8.2 Segurança, Autenticação e Prevenção de Ataques V2X

A utilização do canal rádio aberto (5G Sidelink PC5) expõe a frota a riscos cibernéticos severos. A segurança lógica é reforçada a vários níveis.

#### Autenticação e PKI V2X

O sistema integra a infraestrutura de chaves públicas definida pela especificação **IEEE 1609.2** para segurança V2X. A rede exige a troca de certificados de pseudónimo, garantindo que apenas camiões assinados e autorizados criptograficamente possam interagir e emitir comandos no interior do pelotão [6].

#### Mecanismo Anti-Replay e Integridade

Para proteger o sistema contra ataques de repetição (*Replay Attacks*) — onde um intruso capta um comando de travagem de emergência e o tenta retransmitir perigosamente mais tarde — todos os *payloads* no DDS incluem algoritmos de verificação associados a *timestamps* e numeração de sequência restrita. Qualquer mensagem submetida fora da janela temporal de "frescura" será silenciosamente descartada [6].

### 3.8.3 Fiabilidade Intra-veículo (CAN Bus)

No domínio intra-veículo, a fiabilidade foca-se sobretudo no ruído eletromagnético dos motores/inversores e na integridade dos cabos de cobre.

#### Validação de Integridade (CRC)

Cada comando no CAN inclui um **Checksum CRC-16** (Bytes 6 e 7 da trama). O ruído eletromagnético que inverta bits resulta invariavelmente numa falha do CRC, forçando a ECU atuadora a rejeitar a trama.

#### Alive Counters (Watchdog)

O byte 5 das tramas funciona como um contador de vida. Caso o microcontrolador de travagem detete uma estagnação deste contador (sinalizando que a ECU de Controlo Central parou de transmitir ou o barramento colapsou), o atuador transita isoladamente para um **Fail-Safe State** (*Estado de Falha Segura*), aplicando travagens graduais baseadas no último comando reconhecido.

---

# Bibliografia

**[1]** T. Steinbach, H. Zinner, M. Rost, e K. Wolf, *"Comparing Time-Triggered Ethernet with PEV in an automotive in-vehicle network architecture"*, Proceedings of the IEEE 8th International Workshop on Factory Communication Systems (WFCS), Nancy, França, 2010, pp. 25–28.

**[2]** A. M. da Silva, A. R. de Oliveira, e C. A. C. Marcon, *"Analysis of FreeRTOS Overheads on Periodic Tasks"*, Anais do XVII Workshop em Desempenho de Sistemas Computacionais e de Comunicação (WPerformance), Natal, Brasil, Sociedade Brasileira de Computação, 2018, pp. 219–232.

**[3]** K. Tindell, A. Burns, e A. J. Wellings, *"Calculating Controller Area Network (CAN) Message Response Times"*, Control Engineering Practice, vol. 3, n.º 8, Elsevier, 1995, pp. 1163–1169.

**[4]** A. Ali, W. Hamouda, e L. U. Khan, *"IEEE 802.11bd & 5G NR V2X: Evolution of Radio Access Technologies for V2X Communications"*, IEEE Access, vol. 7, 2019, pp. 143186–143198.

**[5]** L. H. Sampaio, T. C. de Brito, e S. R. L. Meira, *"Evaluating DDS and MQTT for real-time vehicular networks"*, Proceedings of the IEEE Intelligent Vehicles Symposium (IV), Changshu, China, 2018, pp. 1195–1200.

**[6]** W. Whyte, A. Weimerskirch, V. Kumar, e T. Hehn, *"A Security Credential Management System for V2X Communications"*, IEEE Transactions on Intelligent Transportation Systems, vol. 14, n.º 4, 2013, pp. 2096–2106.
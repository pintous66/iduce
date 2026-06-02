# X. Real-Time Operating System (RTOS) Design and Selection

Esta secção detalha a seleção e as diretrizes de configuração do Sistema Operativo de Tempo Real (RTOS) responsável por gerir a Unidade de Controlo Eletrónico (ECU) dedicada exclusivamente à Gestão do Pelotão (*Platoon Management*).

O objetivo é assegurar o determinismo e a fiabilidade na execução das tarefas de coordenação, manutenção preditiva e comunicação inter-veicular, isolando estas operações da ECU central de controlo do veículo (*Vehicle Control*).

---

## X.1. Decomposição do Sistema e Requisitos de Tempo Real

Para selecionar o RTOS adequado, é fundamental "desmontar" as operações da ECU de gestão do pelotão e classificar a sua criticidade temporal.

Ao isolar esta ECU, o foco deixa de ser a atuação física direta (travagem/direção), passando para a coordenação de dados e envio de comandos atempados para a ECU do veículo (VC) e para outros veículos do pelotão.

O sistema divide-se conceptualmente em:

### Hard Real-Time (Prazos Estritos)

#### Gestão do Pelotão (PlatMgmt) e Comunicações (COMM)

O PlatMgmt tem de manter uma visão atualizada da posição, direção, velocidade e estado dos outros membros do pelotão.

A comunicação entre os membros do pelotão é crítica, uma vez que a informação flui das Comunicações para o PlatMgmt e, subsequentemente, para a ECU de Controlo do Veículo (VC).

Falhar um prazo na entrega destes dados ao VC pode comprometer as decisões de curto prazo.

### Soft / Firm Real-Time (Prazos Flexíveis)

#### Manutenção Preditiva (PredMaint)

No veículo líder, o PlatMgmt recebe e processa o estado dos vários subsistemas para prever anomalias.

A monitorização de parâmetros como a pressão dos pneus e a temperatura do motor é vital a longo prazo para minimizar o tempo de inatividade, mas o processamento destes dados tolera atrasos ligeiros sem risco de colisão imediata.

---

## X.2. Avaliação Comparativa de RTOS

Na análise de um RTOS para a ECU de Gestão do Pelotão, considerámos abordagens com perfis distintos.

O AUTOSAR, por exemplo, é o standard de excelência na indústria automóvel, mas a sua complexidade e configuração estática tornam a sua adoção irrealista para a dimensão e prazos deste projeto.

Alternativas comerciais como o QNX oferecem um excelente isolamento de falhas, mas exigem hardware mais pesado e trazem complicações de licenciamento.

A escolha acabou por recair no FreeRTOS por ser o compromisso mais lógico e prático.

Sendo *open-source* e tendo uma curva de aprendizagem mais acessível, adapta-se perfeitamente à necessidade de prototipagem do nosso contexto académico.

Em paralelo, não compromete a fiabilidade do sistema: fornece de forma rigorosa os mecanismos críticos de escalonamento preemptivo e de comunicação inter-processos (IPC) necessários para gerir o tráfego de rede e a agregação de dados do pelotão antes de os enviar para a ECU de controlo.

### Comparação de RTOS

| RTOS | Vantagens | Desvantagens |
|--------|------------|---------------|
| **FreeRTOS** | - Open-source e gratuito<br>- Suporte a escalonamento preemptivo<br>- Comunidade ativa e documentação extensa | - Menos robusto que soluções comerciais POSIX<br>- Requer configuração manual detalhada<br>- Não possui certificação automóvel nativa (ex.: ISO 26262) |
| **AUTOSAR OS** | - Standard da indústria automóvel<br>- Suporte a múltiplas arquiteturas de rede | - Elevada complexidade de implementação<br>- Configuração estática e inflexível |
| **QNX Neutrino** | - Excelente isolamento de falhas (Microkernel POSIX) | - Custo elevado e licenciamento complexo<br>- Exige processadores com mais recursos |

---

## X.3. Funcionalidades Chave do RTOS Aplicadas à Gestão do Pelotão

Para dar resposta aos desafios de agregação de dados e gestão de rede, o sistema fará uso das seguintes mecânicas do FreeRTOS:

### Políticas de Escalonamento (*Scheduling*)

Implementação de um escalonamento preemptivo baseado em prioridades fixas.

A receção de dados de emergência de outros veículos via COMM e a sua passagem pelo PlatMgmt terão as prioridades mais elevadas, interrompendo tarefas computacionalmente mais pesadas mas menos críticas (por exemplo, modelos de PredMaint) quando necessário.

### Comunicação Inter-Processos (IPC)

#### Filas de Mensagens (*Message Queues*)

Utilizadas para a passagem assíncrona de pacotes de dados recebidos da rede (tarefa COMM) para a tarefa de PlatMgmt.

Permitem lidar com picos de tráfego na rede (*bursts*) sem bloquear a execução de outras tarefas.

### Sincronização e Gestão de Concorrência

#### Mutexes com Herança de Prioridade (*Priority Inheritance*)

Aplicados no acesso a estruturas de dados globais, como a tabela que guarda a visão atualizada da velocidade e posição de todos os membros do pelotão.

Este mecanismo garante que uma tarefa crítica não fica bloqueada enquanto uma tarefa de baixa prioridade atualiza o estado de manutenção de um seguidor.

---

## X.4. Diretrizes para o Mapeamento do Taskset

### Distribuição de Tarefas

Identificação das tarefas independentes a instanciar no FreeRTOS dentro desta ECU específica, nomeadamente:

- Processos associados ao PlatMgmt;
- Processos de COMM.

### Parâmetros Temporais

Definição teórica dos seguintes parâmetros para cada tarefa:

- **Período ($T$)**
- **Tempo de Execução ($C$)**
- **Prazo ($D$)**

O planeamento deve assegurar que o processamento do PlatMgmt seja suficientemente rápido para alimentar a ECU do VC, cujo ciclo de decisão varia entre **0,1 s e 5 s**, e lidar com as taxas de receção de informação provenientes de outros veículos e infraestruturas.

Baseado nestas informações, o taskset pode ser configurado para garantir que as tarefas de PlatMgmt e COMM tenham prazos compatíveis com os requisitos de tempo real, enquanto as tarefas de PredMaint podem ser configuradas com prazos mais flexíveis.

| Tarefa (Task)          | Descrição                                                                                                                                       | Prioridade (RM) | Período (T) | Tempo de Execução (C) | Prazo (D) |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | --------------- | ----------- | --------------------- | --------- |
| **Task_COMM_RX**       | Receção de dados da rede. Processa as mensagens recebidas dos veículos seguidores (estado, velocidade, anomalias).               | Alta (1)        | 50 ms       | 10 ms                 | 50 ms     |
| **Task_PlatMgmt**      | Agrega os dados recebidos, atualiza a matriz de estado do pelotão e envia comandos para a ECU de Controlo do Veículo. | Média-Alta (2)  | 100 ms      | 20 ms                 | 100 ms    |
| **Task_COMM_TX**       | Transmissão de dados para a rede. Envia os comandos de sincronização do líder e respostas para os veículos seguidores.                          | Média-Alta (2)  | 100 ms      | 10 ms                 | 100 ms    |
| **Task_PredMaint**     | Processa os dados de manutenção preditiva recebidos dos vários veículos e executa algoritmos de deteção de anomalias.                           | Baixa (3)       | 1000 ms     | 100 ms                | 1000 ms   |


Esta taskset é apenas um exemplo teórico e deve ser ajustada com base em testes empíricos e simulações para garantir que os prazos são cumpridos e que o sistema é robusto face a variações no tráfego de rede e na carga de processamento.

Para este exemplo o calculo da escalonabilidade é feito usando a fórmula de Liu & Layland para escalonamento Rate Monotonic (RM):
$$ U = \sum_{i=1}^{n} \frac{C_i}{T_i} $$
Onde $C_i$ é o tempo de execução da tarefa $i$ e $T_i$ é o período da tarefa $i$. Para um sistema ser escalonável sob RM, a utilização total $U$ deve ser menor ou igual a $n(2^{1/n} - 1)$, onde $n$ é o número de tarefas.

Neste caso, com 4 tarefas, o limite de escalonabilidade é aproximadamente 0.7568. Calculando a utilização total:
$$ U = \frac{10}{50} + \frac{20}{100} + \frac{10}{100} + \frac{100}{1000} = 0.2 + 0.2 + 0.1 + 0.1 = 0.6 $$
Como $U = 0.6$ é menor que o limite de 0.7568, o sistema é escalonável sob RM.


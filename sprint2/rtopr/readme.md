# Sprint 2: Real-Time OS Design - Platoon Management (RTOPR)

## Introdução: Revisão da Arquitetura do Sistema

Neste segundo sprint, a arquitetura do sistema sofreu uma revisão significativa com o objetivo de otimizar a latência e simplificar a infraestrutura de hardware. A Unidade de Controlo do Veículo (VC) e o módulo de Gestão do Pelotão (PlatMgmt) passaram a residir na mesma Unidade de Controlo Eletrónico (ECU).

Esta consolidação arquitetural altera o paradigma de comunicação: em vez de os módulos trocarem dados através de uma rede externa, a passagem de informação passa a ser gerida internamente pelo Sistema Operativo de Tempo Real (FreeRTOS) através de mecanismos de Inter-Process Communication (IPC). O foco deste desenho recai sobre o isolamento do módulo de Platoon Management, definindo como o RTOS aloca tempo de processador às tarefas de gestão do pelotão, garantindo que o processamento das comunicações e da manutenção preditiva não bloqueia os ciclos críticos de atuação física do veículo (VC).

Claro que num cenário ideal, o VC e o PlatMgmt estariam pelo menos em núcleos separados, algo que vai ser tido em conta quando for feita a análise de escalabilidade, mas para efeitos de simplificação e foco na gestão do pelotão, ambos os módulos partilham a mesma ECU neste sprint.

---

## 1. Identificação de Tarefas (PlatMgmt)

Para efeitos de escalonamento no FreeRTOS, os processos associados à Gestão do Pelotão foram divididos em tarefas independentes, separando claramente a receção, processamento e transmissão de dados.

- **Task_PlatMgmt_Update**: Tarefa central responsável por manter uma visão atualizada da posição, direção, velocidade e estado dos outros membros do pelotão. Processa os dados recebidos para criar uma matriz de estado consolidada que será consumida pela tarefa de VC.

- **Task_PlatMgmt_COMM_RX**: Lida com a receção assíncrona de dados provenientes da rede (telemetria dos seguidores). Entrega os dados brutos à tarefa principal do PlatMgmt. 

- **Task_PlatMgmt_COMM_TX**: Lida com a transmissão síncrona de dados para a rede. No veículo líder, transmite as ordens de marcha ou decisões de paragem de emergência para os veículos seguidores. No veículo seguidor, transmite a telemetria atualizada para o líder.

- **Task_PredMaint_Leader**: Presente no veículo líder, monitoriza e agrega o estado dos subsistemas dos vários veículos para fins de manutenção preditiva, visando prever anomalias antes que estas ocorram.

---

## 2. Mecanismos de Sincronização (IPC e Concorrência)

Com o PlatMgmt e o VC a partilharem a mesma ECU e espaço de memória, a troca de dados exige mecanismos de sincronização estritos para evitar a corrupção de dados e garantir determinismo:

- **Memória Partilhada protegida por Mutexes**: O estado global do pelotão é guardado numa estrutura de dados partilhada. O acesso de leitura/escrita a esta estrutura é estritamente protegido por um Mutex.

- **Protocolo de Herança de Prioridade (Priority Inheritance Protocol)**: Existindo tarefas de maior prioridade, se estas tentarem ler o estado do pelotão enquanto o PlatMgmt (que possui prioridade inferior) estiver a escrever, o FreeRTOS eleva temporariamente a prioridade do PlatMgmt. Este mecanismo previne a inversão de prioridade, garantindo que o recurso é libertado rapidamente para a atuação do veículo.

- **Filas de Mensagens (Message Queues)**: Os pacotes de rede recolhidos pelas Rotinas de Serviço de Interrupção (ISR) são colocados em filas de mensagens para acordar a Task_PlatMgmt_COMM_RX, permitindo absorver picos de tráfego na rede (jitter) sem bloquear o processador.

---

## 3. Propriedades de Escalonamento

O sistema utilizará um escalonamento preemptivo baseado em prioridades fixas, assumindo o algoritmo Rate Monotonic (RM), onde tarefas com menores períodos recebem maiores prioridades.

O planeamento temporal do PlatMgmt foi desenhado para alimentar atempadamente as decisões do VC, cujo ciclo de decisão varia entre 0.1s e 5s.

| Tarefa | Prioridade (RM) | Período (Ti) | Execução (Ci) | Prazo (Di) |
|--------|----------------|--------------|---------------|------------|
| COMM_RX | Alta | 50 ms | 10 ms | 50 ms |
| Update | Média-Alta | 100 ms | 20 ms | 100 ms |
| COMM_TX | Média | 100 ms | 15 ms | 100 ms |
| PredMaint | Baixa | 1000 ms | 50 ms | 1000 ms |

**Justificação das Propriedades Temporais:**

- Assume-se um modelo de prazos implícitos (implicit deadlines), onde o prazo (Di) é igual ao período (Ti).
- A Task_PlatMgmt_Update opera com um período de 100 ms, garantindo que, no limite inferior do ciclo do VC (0.1s), a matriz de estado do pelotão contém dados recém-calculados.
- A Task_PlatMgmt_COMM_RX corre ao dobro da frequência (50 ms) para acomodar rapidamente as altas taxas de receção da telemetria, mitigando a perda de pacotes e reduzindo a latência no tratamento de emergências.
- A Task_PlatMgmt_COMM_TX tem um período de 100 ms, alinhado com o ciclo de decisão do VC, garantindo que as ordens de marcha ou telemetria são transmitidas a tempo de serem processadas.
- A Task_PredMaint_Leader, embora importante para a manutenção preditiva, tem um período mais longo (1000 ms) e prioridade mais baixa, pois as suas decisões não são críticas para o ciclo de atuação imediata do veículo.

---

## 4. Análise de Escalonabilidade

Para comprovar que as tarefas de gestão do pelotão não excedem a capacidade de processamento e cumprem sempre os seus prazos teóricos, foi efetuada uma verificação matemática utilizando o teste de limite de utilização de processador de Liu & Layland para o algoritmo Rate Monotonic (RM).

A utilização total do processador (U) para o sub-sistema PlatMgmt é dada por:

$$
U_{PlatMgmt} = \sum_{i=1}^{n} \frac{C_i}{T_i}
$$

Substituindo com os valores teóricos da tabela:

$$
U_{PlatMgmt} = \frac{10 ms}{50 ms} + \frac{20 ms}{100 ms} + \frac{15 ms}{100 ms} + \frac{50 ms}{1000 ms}
$$

$$
U_{PlatMgmt} = 0.20 + 0.20 + 0.15 + 0.05 = 0.60
$$

O sub-sistema consome 60% da capacidade do processador.

O limite teórico superior (U_bound) para um conjunto de n = 4 tarefas, abaixo do qual o escalonamento RM é garantido independentemente das fases de chegada, é:

$$
U_{bound} = n(2^{1/n} - 1)
$$

$$
U_{bound} = 4(2^{1/4} - 1) \approx 0.756
$$

Uma vez que:

$$
0.60 \le 0.756
$$

o taskset de Gestão do Pelotão é teoricamente escalonável.

## 5. Considerações sobre a Arquitetura de Hardware (Multi-Core)

É fundamental notar que esta análise de escalonabilidade assume a utilização de um processador de, pelo menos, dois núcleos (dual-core) na ECU. Nesta arquitetura, o Controlo do Veículo (VC) e a Gestão do Pelotão (PlatMgmt) partilham a mesma ECU, mas cada módulo é alocado a um núcleo dedicado (Core Affinity).  

Se ambos os módulos partilhassem o mesmo núcleo singular, a adição das tarefas críticas de VC à utilização de 60% do PlatMgmt faria com que a carga total do processador ultrapassasse facilmente os 100%, tornando o sistema impossível de escalonar. Uma solução teórica para contornar esta limitação num sistema de núcleo único seria duplicar os períodos de todas as tarefas do PlatMgmt, reduzindo a sua utilização isolada para 30%. No entanto, isso comprometeria inaceitavelmente a capacidade de resposta e o desempenho do pelotão em tempo real.  

Uma vez que a especificação detalhada e o escalonamento do taskset de VC estão fora do âmbito (out of scope) desta fase do projeto, a adoção da premissa de uma arquitetura dual-core assegura que o sistema se mantém matematicamente escalonável e garante um isolamento de desempenho robusto entre a atuação física do veículo e a coordenação do pelotão.

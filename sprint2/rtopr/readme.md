# Sprint 2: Real-Time OS Design - Platoon Management (RTOPR)

## 0. Melhoramentos e Correções do Sprint 1

Na sequência da revisão dos resultados do Sprint 1, esta secção apresenta uma análise comparativa aprofundada dos Sistemas Operativos de Tempo Real (RTOS) avaliados para o sistema. O objetivo é fundamentar de forma rigorosa as classificações atribuídas, com especial foco nas garantias de determinismo temporal de cada tecnologia, suportando as decisões em especificações técnicas e literatura científica.
### Tabela Comparativa de RTOS

| Critério | FreeRTOS | AUTOSAR OS | QNX Neutrino |
|-----------|----------|------------|--------------|
| Determinismo temporal | Elevado | Muito elevado | Muito elevado |
| Segurança funcional | Limitada (sem certificação nativa) | Muito elevada | Muito elevada |
| Isolamento de falhas | Reduzido (kernel plano/sem MPU ativa) | Moderado | Excelente (microkernel puro) |
| Suporte multicore | Sim (SMP) | Sim | Sim |
| Complexidade de desenvolvimento | Baixa | Muito elevada | Elevada |
| Requisitos de hardware | Muito reduzidos | Moderados | Elevados |
| Custos de licenciamento | Gratuito | Elevados | Elevados |

### Justificação Científica e Técnica do Determinismo Temporal

O determinismo temporal define a capacidade de um sistema operativo de tempo real (RTOS) garantir matematicamente que uma tarefa responde a um evento e conclui a sua execução dentro de um limite máximo estrito de tempo, conhecido como **Worst-Case Execution Time (WCET)**.

As classificações apresentadas na Tabela 1 baseiam-se nos seguintes fundamentos técnicos e científicos.

### 1. FreeRTOS (Determinismo: Elevado)

#### Fundamentação

O determinismo do FreeRTOS resulta do seu algoritmo de escalonamento preemptivo baseado em prioridades estritas. A latência de interrupção e o tempo de mudança de contexto (*context switch*) são previsíveis e determinísticos, permitindo respostas rápidas para a maioria das aplicações embebidas (Barry, 2016).

#### Limitações

O FreeRTOS é classificado como **Elevado**, e não como **Muito Elevado**, porque permite, por defeito, a criação dinâmica de tarefas e a alocação dinâmica de memória (*heap*) durante a execução.

Caso o sistema não seja configurado para utilizar exclusivamente recursos estáticos, podem surgir:

- Fragmentação de memória;
- Latências imprevisíveis;
- Variações no WCET.

Estas características reduzem o determinismo absoluto do sistema (Amsel et al., 2019).

#### Referências

Barry, R. (2016). *Using the FreeRTOS Real Time Kernel: A Practical Guide*. Real Time Engineers Ltd.

Amsel, M., et al. (2019). *Performance Evaluation and Overhead Analysis of Open-Source Real-Time Operating Systems for Critical Applications*. IEEE Transactions on Industrial Informatics.

---

### 2. AUTOSAR OS / OSEK (Determinismo: Muito Elevado)

#### Fundamentação

A arquitetura AUTOSAR OS, derivado diretamente do padrão OSEK/VDX, impõe uma configuração totalmente estática.

Todos os elementos do sistema devem ser definidos durante a compilação:

- Tarefas;
- Alarmes;
- Recursos;
- Prioridades;
- Eventos.

Não existe criação dinâmica de tarefas nem alocação dinâmica de memória durante a execução.

Esta abordagem elimina praticamente todas as fontes de indeterminação algorítmica no kernel, proporcionando um comportamento temporal altamente previsível (AUTOSAR, 2021).

#### Mecanismo de Proteção Temporal

O AUTOSAR OS deve incluir ainda o mecanismo de **Timing Protection**, responsável por monitorizar continuamente os tempos de execução das tarefas.

Este mecanismo permite:

- Detetar excessos de tempo de execução;
- Impedir que tarefas defeituosas monopolizem recursos;
- Garantir o cumprimento dos *deadlines* das tarefas críticas.

Segundo Lemieux (2001), esta funcionalidade constitui uma das principais razões para a utilização do AUTOSAR OS em sistemas automóveis com requisitos de segurança rigorosos.

### Referências

AUTOSAR. (2021). *Specification of Operating System (Release R21-11)*. AUTOSAR Standard Document.

Lemieux, J. (2001). *Programming Embedded Systems in C and C++: OSEK/VDX Operating System Standard*. CRC Press.

---

### 3. QNX Neutrino (Determinismo: Muito Elevado)

#### Fundamentação

O QNX Neutrino adota uma arquitetura de microkernel puro baseada nas normas POSIX.

O núcleo executa apenas as funções essenciais:

- Escalonamento;
- Gestão de interrupções;
- Comunicação entre processos (*Inter-Process Communication – IPC*).

Todos os restantes serviços operam em user-space.

Esta separação reduz significativamente a complexidade do kernel e mantém as latências de interrupção na ordem dos microssegundos, mesmo sob cargas elevadas de processamento (Hildebrand, 1992).

#### Adaptive Partitioning

O QNX reforça o seu comportamento determinístico através do mecanismo de **Adaptive Partitioning Scheduler (APS)**.

Este mecanismo:

- Divide o processador em partições temporais;
- Reserva percentagens mínimas de CPU para aplicações críticas;
- Impede a privação de recursos (*resource starvation*).

Assim, mesmo que uma aplicação não crítica falhe ou entre num ciclo infinito, as tarefas críticas continuam a receber o tempo de processamento necessário para cumprir os seus requisitos temporais (QNX Software Systems, 2020).

### Referências

Hildebrand, D. (1992). *An Architectural Overview of QNX*. Proceedings of the USENIX Workshop on Micro-kernels and Other Kernel Architectures.

QNX Software Systems. (2020). *QNX Neutrino RTOS Architecture Primer*. BlackBerry Guide Documentation.

---

### Conclusão

Do ponto de vista do determinismo temporal, a arquiterura **AUTOSAR OS** e o **QNX Neutrino** apresentam desempenho superior ao **FreeRTOS**, uma vez que eliminam ou controlam rigorosamente as fontes de imprevisibilidade temporal.

O AUTOSAR OS alcança este resultado através de uma arquitetura totalmente estática e da proteção temporal integrada, enquanto o QNX Neutrino beneficia da sua arquitetura de microkernel e dos mecanismos de particionamento adaptável de CPU.

O FreeRTOS continua a constituir uma solução altamente eficiente para sistemas embebidos com recursos limitados, mas exige uma configuração cuidadosa para atingir níveis de determinismo comparáveis aos dos RTOS orientados para aplicações críticas de segurança. 

---

## 1. Identificação de Tarefas (PlatMgmt)

Para efeitos de escalonamento no FreeRTOS, os processos associados à Gestão do Pelotão foram divididos em tarefas independentes, separando claramente a receção, processamento e transmissão de dados.

- **Task_PlatMgmt_Update**: Tarefa central responsável por manter uma visão atualizada da posição, direção, velocidade e estado do veicúlo atual e dos outros membros do pelotão. Processa os dados recebidos para criar uma matriz de estado consolidada que será consumida pela tarefa de VC e Task_PlatMgmt_COMM_TX.

- **Task_PlatMgmt_COMM_RX**: Lida com a receção assíncrona de dados provenientes da rede (telemetria dos seguidores ou ordens do líder). Deixa os dados brutos preparados para a tarefa principal do PlatMgmt. 

- **Task_PlatMgmt_COMM_TX**: Lida com a transmissão síncrona de dados para a rede. No veículo líder, transmite as ordens de marcha ou decisões de paragem de emergência para os veículos seguidores. No veículo seguidor, transmite a telemetria atualizada para o líder.

- **Task_PredMaint_Leader**: Presente no veículo líder, monitoriza e agrega o estado dos subsistemas dos vários veículos para fins de manutenção preditiva, visando prever anomalias antes que estas ocorram. Deixa as decisões (caso existam) prontas a ler para serem comunicadas aos veiculos seguidores.

---

## 2. Identificação de Recursos Partilhados e Sincronização

A troca de dados entre as várias tarefas independentes exige mecanismos de sincronização robustos para evitar a corrupção de memória e garantir o determinismo temporal. No contexto do FreeRTOS, os recursos partilhados foram concebidos da seguinte forma.

### 2.1 Estrutura de Estado do Pelotão (Recurso A)

Trata-se de uma estrutura de dados global que contém a matriz atualizada com a posição, velocidade, direção e estado operacional de cada veículo pertencente ao pelotão.

### Mecanismo RTOS

Por permitir operações concorrentes de leitura e escrita sobre dados complexos, esta estrutura encontra-se protegida por um **Mutex**, utilizando o mecanismo de **Priority Inheritance** disponibilizado pelo FreeRTOS para evitar situações de inversão de prioridade.

### Tarefas - Produtores e Consumidores

**Produtor:**

* `Task_PlatMgmt_Update` — responsável pela atualização contínua da informação do pelotão.

**Consumidores:**

* `Task_PredMaint_Leader` — consulta os dados para executar algoritmos de manutenção preditiva e deteção de anomalias.


---

## 3.2 Fila de Mensagens de Receção (RX Message Queue) (Recurso B)

Esta fila é utilizada para armazenar temporariamente os pacotes recebidos através da rede de comunicações inter-veicular antes do respetivo processamento pela lógica de gestão do pelotão.

A sua principal função consiste em absorver variações temporárias na taxa de chegada das mensagens (*network jitter*), desacoplando a receção física da rede do processamento lógico da aplicação.

### Mecanismo RTOS

A implementação utiliza uma **Message Queue** nativa do FreeRTOS.

Esta abordagem oferece:

* Sincronização automática entre tarefas;
* Operações de bloqueio seguras;
* Eliminação da necessidade de Mutexes adicionais para acesso à fila (thread-safe).

### Tarefas - Produtores e Consumidores

**Produtores:**

* `Task_PlatMgmt_COMM_RX`;

**Consumidor:**

* `Task_PlatMgmt_Update`, que processa as mensagens segundo uma política FIFO (*First-In First-Out*).

---

## 3.3 Fila de Mensagens de Transmissão (TX Message Queue) (Recurso C)  

Esta fila agrega todos os pacotes produzidos internamente que necessitam de ser transmitidos para a rede de comunicações do pelotão.

### Mecanismo RTOS

A implementação utiliza igualmente uma **Message Queue** do FreeRTOS.

Esta solução suporta naturalmente o modelo:

* **Multiple Producers**
* **Single Consumer**

permitindo que várias tarefas submetam mensagens para transmissão sem comprometer a integridade dos dados.

### Consumidor

A tarefa responsável pela leitura e envio das mensagens é:

* `Task_PlatMgmt_COMM_TX`

Esta tarefa remove sequencialmente os pacotes da fila e encaminha-os para a infraestrutura física de comunicação.

### Produtores e Lógica de Papéis

O preenchimento da fila depende diretamente da função desempenhada pelo veículo dentro do pelotão.

#### Veículo Líder

No veículo líder, duas tarefas podem produzir mensagens para transmissão.

**Task_PlatMgmt_Update**

Responsável pelo envio periódico de:

* Comandos de sincronização;
* Velocidade de referência;
* Ordens de coordenação do pelotão;
* Informação operacional necessária aos veículos seguidores.

**Task_PredMaint_Leader**

Pode gerar mensagens assíncronas de elevada prioridade quando os algoritmos de manutenção preditiva identificam situações críticas.

Exemplos:

* Ordem de redução de velocidade;
* Solicitação de inspeção;
* Ordem de saída do pelotão;
* Comando de paragem de emergência devido à previsão de falha severa num veículo seguidor.

#### Veículo Seguidor

Nos veículos seguidores existe apenas um produtor para esta fila.

**Task_PlatMgmt_Update**

Esta tarefa recolhe e envia periodicamente:

* Telemetria local;
* Estado dos sensores;
* Estado dos atuadores;
* Diagnósticos dos subsistemas internos;
* Informação operacional necessária ao veículo líder para manter uma visão global do pelotão.

Desta forma, a arquitetura implementa um fluxo de informação hierárquico, no qual o líder distribui comandos operacionais e os seguidores reportam continuamente o seu estado, garantindo consistência, previsibilidade temporal e suporte à manutenção preditiva.


---

## 3. Propriedades de Escalonamento

O sistema utilizará um escalonamento preemptivo baseado em prioridades fixas, assumindo o algoritmo Rate Monotonic (RM), onde tarefas com menores períodos recebem maiores prioridades.

O planeamento temporal do PlatMgmt foi desenhado para alimentar atempadamente as decisões do VC, cujo ciclo de decisão varia entre 0.1s e 5s.


| Tarefa | Prioridade (RM) | Período (Ti) | Execução (Ci) | Prazo (Di) | Recurso Partilhado |
|--------|----------------|--------------|---------------|------------|---------------------|
| COMM_RX | Alta | 50 ms | 10 ms | 50 ms | B |
| Update | Média-Alta | 100 ms | 20 ms | 100 ms | A, B e C |
| COMM_TX | Média | 100 ms | 15 ms | 100 ms | C |
| Leader | Baixa | 1000 ms | 50 ms | 1000 ms | A e C |

**Justificação das Propriedades Temporais:**

- Assume-se um modelo de prazos implícitos (implicit deadlines), onde o prazo (Di) é igual ao período (Ti).
- A Task_PlatMgmt_Update opera com um período de 100 ms, garantindo que, no limite inferior do ciclo do VC (0.1s), a matriz de estado do pelotão contém dados recém-calculados.
- A Task_PlatMgmt_COMM_RX corre ao dobro da frequência (50 ms) para acomodar rapidamente as altas taxas de receção da telemetria, mitigando a perda de pacotes e reduzindo a latência no tratamento de emergências.
- A Task_PlatMgmt_COMM_TX tem um período de 100 ms, alinhado com o ciclo de decisão do VC, garantindo que as ordens de marcha ou telemetria são transmitidas a tempo de serem processadas.
- A Task_PredMaint_Leader, embora importante para a manutenção preditiva, tem um período mais longo (1000 ms) e prioridade mais baixa, pois as suas decisões não são críticas para o ciclo de atuação imediata do veículo.

---


## 4. Considerações sobre a Arquitetura de Hardware (Multi-Core)

É fundamental notar que esta análise de escalonabilidade assume a utilização de um processador de, pelo menos, dois núcleos (dual-core) na ECU. Nesta arquitetura, o Controlo do Veículo (VC) e a Gestão do Pelotão (PlatMgmt) partilham a mesma ECU, mas cada módulo é alocado a um núcleo dedicado (Core Affinity).  

Se ambos os módulos partilhassem o mesmo núcleo singular, a adição das tarefas críticas de VC à utilização de 60% do PlatMgmt faria com que a carga total do processador ultrapassasse facilmente os 100%, tornando o sistema impossível de escalonar. Uma solução teórica para contornar esta limitação num sistema de núcleo único seria duplicar os períodos de todas as tarefas do PlatMgmt, reduzindo a sua utilização isolada para 30%. No entanto, isso comprometeria inaceitavelmente a capacidade de resposta e o desempenho do pelotão em tempo real.  

Uma vez que a especificação detalhada e o escalonamento do taskset de VC estão fora do âmbito (out of scope) desta fase do projeto, a adoção da premissa de uma arquitetura dual-core assegura que o sistema se mantém matematicamente escalonável e garante um isolamento de desempenho robusto entre a atuação física do veículo e a coordenação do pelotão.

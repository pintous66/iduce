# Iduce - Sprint 1

Grupo 1 - Rodrigo Pinto, Rodrigo Bolelho, Martim Botelho

## Índice
1. [Introdução](#11-introdução)
2. [Planeamento do Sprint 1 (PMDEV)](#12-planeamento-do-sprint-1-pmdev)
3. [Design de Arquitetura (VEVCA)](#13-design-de-arquitetura-vevca)
4. [Real-Time Operating System (RTOS) Design and Selection (RTOPR)](#14-real-time-operating-system-rtos-design-and-selection-rtopr)
5. [Tecnologias de Comunicação (SYOSY)](#15-tecnologias-de-comunicação-syosy)
6. [Conclusão](#16-conclusão)


## 1.1. Introdução
Este documento apresenta a análise e o planeamento para a implementação de um sistema de gestão de pelotão de veículos autónomos no contexto da unicade curricular de IDUCE (Engenharia de Casos de Uso Focados na Indústria). Este projeto foca-se na definição da arquitetura do sistema, na escolha do Sistema Operativo de Tempo Real (RTOS) e das tecnologias de comunicação a utilizar. O objetivo é criar um sistema robusto e eficiente que permita a coordenação eficaz entre os veículos do pelotão, garantindo a segurança e a fiabilidade das operações.

## 1.2 Planeamento do Sprint 1 (PMDEV)

O planeamento do sprint 1 foi feito no primeiro dia do sprint, e teve como resultado o seguinte plano de atividades:

![Planeamento do Sprint 1](./pmdev/planning.png)

Os principais marcos do sprint 1 incluem:
- Construção do design de arquiterura;
- Estudo, análise e escolha do RTOS a utilizar e das suas funcionalidades chave;
- Estuda, análise e escolha das tecnologias de comunicação a utilizar;

## 1.3 Design de Arquitetura (VEVCA)

A seguinte secção apresenta o design de arquitetura proposto para o sistema de monitorização de *platooning*, incluindo a alocação dos componentes pelas ECUs e a justificação da mesma.

### Diagrama de Arquitetura

O diagrama apresenta a arquitetura proposta para o sistema de monitorização de *platooning*. A arquitetura representa os principais subsistemas do veículo, os sensores e atuadores físicos, e os fluxos de informação usados para trocar dados entre os componentes de software.

A arquitetura foi definida de forma a representar diretamente as dependências entre sensores, módulos de decisão, comunicação, gestão do *platoon* e atuadores. Neste caso, as ligações entre blocos representam interfaces lógicas de dados entre produtores e consumidores de informação. Assim, as setas entre sensores, módulos e atuadores indicam que dados são fornecidos e por quem são consumidos, sem introduzir camadas intermédias adicionais no diagrama.

![Diagrama de Arquitetura](vevca/arquitetura.svg)

As cores do diagrama representam a alocação lógica dos componentes pelas ECUs. Assim, o mesmo diagrama mostra simultaneamente os fluxos entre sensores, módulos e atuadores, e a distribuição proposta dos componentes pelas unidades computacionais do veículo.

Esta opção mantém a arquitetura simples e focada nos principais fluxos de dados do sistema. As interfaces são representadas diretamente pelas ligações entre componentes, em vez de serem modeladas como blocos físicos separados. Desta forma, o diagrama evidencia as relações entre sensores, módulos de decisão, comunicação, gestão do *platoon* e atuadores.

O *Vehicle Control* é responsável pelas decisões de curto prazo, como acelerar, travar ou virar, e envia comandos para os atuadores de *Steering*, *Braking* e *Powertrain*. A *Navigation* fornece informação de rota ao *Vehicle Control*. O *Platoon Management* mantém a visão do estado do *platoon* e fornece decisões de nível superior ao *Vehicle Control*. O *Predictive Maintenance* monitoriza o estado do veículo e, dependendo do papel do veículo, comunica os seus resultados ao *Platoon Management*, quando o veículo atua como líder, ou ao *Communication*, quando o veículo atua como seguidor.

No caso dos seguidores, uma potencial avaria detetada pelo *PredMaint* é enviada através do *COMM* para o veículo líder. No líder, o *COMM* encaminha a informação recebida para o *PlatMgmt*, que atualiza a visão global do *platoon*. Caso seja necessário encostar à berma, o *PlatMgmt* influencia o *VC* do líder, e os seguidores continuam a replicar a trajetória do líder de acordo com o comportamento de *platooning*.

#### Alocação Hardware/Software por ECU

| ECU | Nome | Componentes |
| --- | --- | --- |
| ECU 1 | *Perception ECU* | Cameras, LiDAR, Ultrasonic Sensors |
| ECU 2 | *Localization, Motion & Navigation ECU* | GPS, Wheel Speed Sensor, Steering Sensor, NAV |
| ECU 3 | *Health & Diagnostics ECU* | Tyre Pressure Sensors, Engine Temperature Sensors, Brake Status Sensors |
| ECU 4 | *Predictive Maintenance ECU* | PredMaint |
| ECU 5 | *Vehicle Control ECU* | VC |
| ECU 6 | *Communication ECU* | COMM |
| ECU 7 | *Platoon Management ECU* | PlatMgmt - ativo no líder |
| ECU 8 | *Steering ECU* | Steering |
| ECU 9 | *Braking ECU* | Braking |
| ECU 10 | *Powertrain ECU* | Powertrain |

#### Justificação da alocação por ECUs

A alocação dos componentes pelas ECUs foi definida com base nas responsabilidades funcionais, criticidade temporal, volume de dados processado e isolamento de falhas. Esta separação permite distribuir o processamento pelo veículo sem acoplar todos os sensores, módulos de decisão e atuadores numa única ECU.

- **ECU 1 - Perception ECU**  
  Agrupa os sensores responsáveis pela perceção do ambiente, nomeadamente câmaras, LiDAR e sensores ultrassónicos. Estes sensores têm uma responsabilidade funcional semelhante e produzem dados usados para construir a representação do mundo envolvente do veículo. A sua alocação conjunta permite concentrar os dados de perceção numa unidade dedicada.

- **ECU 2 - Localization, Motion & Navigation ECU**  
  Agrega os sensores relacionados com localização e movimento, como GPS, *wheel speed sensor* e *steering sensor*. O módulo *NAV* também é colocado nesta ECU porque depende diretamente destes dados para calcular e atualizar a rota. Esta separação permite concentrar dados cinemáticos e de posicionamento numa unidade funcionalmente coerente.

- **ECU 3 - Health & Diagnostics ECU**  
  Concentra os sensores associados ao estado físico do veículo, como *tyre pressure*, *engine temperature* e *brake status*. Esta ECU tem como responsabilidade recolher dados de diagnóstico relacionados com a condição física dos principais subsistemas do veículo.

- **ECU 4 - Predictive Maintenance ECU**  
  Executa o *PredMaint* numa ECU dedicada porque este módulo pode envolver processamento computacionalmente mais pesado, incluindo análise de tendências, correlação entre sensores, previsão de falhas e possível modelação baseada em *digital twins*. A separação evita que este processamento interfira com tarefas de controlo em tempo-real.

- **ECU 5 - Vehicle Control ECU**  
  Isola o *VC* numa ECU dedicada por ser o núcleo crítico do sistema de controlo. Este módulo executa decisões de curto prazo, como acelerar, travar e virar, com requisitos temporais mais exigentes. A sua separação reduz interferência de outros módulos e facilita uma política de escalonamento mais determinística.

- **ECU 6 - Communication ECU**  
  Contém o *COMM*, responsável pela comunicação entre veículos do *platoon*. Esta separação isola o tráfego de comunicação externa dos módulos de controlo e permite gerir receção, envio e validação de mensagens de estado de forma independente.

- **ECU 7 - Platoon Management ECU**  
  Contém o *PlatMgmt*, responsável por manter a visão atualizada do *platoon* e agregar informação recebida através do *COMM*. Embora o componente possa existir em todos os veículos, a sua função principal fica ativa no veículo que assume o papel de líder.

- **ECU 8/9/10 - Actuator ECUs**  
  *Steering*, *Braking* e *Powertrain* foram colocados em ECUs dedicadas por serem atuadores críticos para o veículo. Esta separação permite maior modularidade e isolamento de falhas, evitando que uma falha num atuador afete diretamente os restantes.

## 1.4 Real-Time Operating System (RTOS) Design and Selection (RTOPR) 

Esta secção detalha a seleção e as diretrizes de configuração do Sistema Operativo de Tempo Real (RTOS) responsável por gerir a Unidade de Controlo Eletrónico (ECU) dedicada exclusivamente à Gestão do Pelotão (*Platoon Management*).

O objetivo é assegurar o determinismo e a fiabilidade na execução das tarefas de coordenação, manutenção preditiva e comunicação inter-veicular, isolando estas operações da ECU central de controlo do veículo (*Vehicle Control*).

## 1.4.1. Avaliação Comparativa de RTOS

A seleção do sistema operativo de tempo real (RTOS) para a ECU de Gestão do Pelotão exigiu uma análise que foi além da simples disponibilidade ou facilidade de utilização. Sendo esta ECU responsável pela agregação de dados provenientes de múltiplos veículos, pela gestão das comunicações de rede e pela transmissão de informação consolidada para a ECU de controlo, o RTOS escolhido deve garantir previsibilidade temporal, fiabilidade e capacidade de evolução futura.

Foram analisadas três soluções representativas de diferentes abordagens ao desenvolvimento de sistemas embebidos críticos: FreeRTOS, AUTOSAR OS e QNX Neutrino.

O QNX Neutrino representa, do ponto de vista técnico, a solução mais robusta entre as alternativas consideradas. A sua arquitetura baseada em microkernel permite isolar componentes críticos do sistema, reduzindo significativamente a propagação de falhas. Em caso de erro numa aplicação, o restante sistema pode continuar operacional, uma característica particularmente valorizada em sistemas automóveis de elevada criticidade. Além disso, o QNX possui certificações amplamente reconhecidas para aplicações de segurança funcional e é utilizado em diversos sistemas automóveis comerciais.

Contudo, estas vantagens têm um custo significativo. O licenciamento comercial, os requisitos de hardware superiores e a maior complexidade de desenvolvimento tornam a sua utilização pouco adequada para um projeto como estes, cujo principal objetivo é validar a arquitetura proposta.

O AUTOSAR OS encontra-se no extremo oposto em termos de adoção industrial. Oferece integração com ecossistemas completos de desenvolvimento automóvel e facilita a conformidade com processos de segurança funcional. No entanto, a sua natureza fortemente configurada e estática implica uma curva de aprendizagem elevada e um esforço considerável de integração, incompatíveis com os recursos disponíveis.

Face a estas alternativas, o FreeRTOS surge como a solução de melhor compromisso entre desempenho, simplicidade e flexibilidade. Apesar de não oferecer o mesmo nível de isolamento de falhas do QNX nem a integração automóvel do AUTOSAR, disponibiliza todos os mecanismos fundamentais necessários para o sistema proposto, incluindo escalonamento preemptivo, gestão eficiente de tarefas, sincronização através de semáforos e mecanismos de comunicação inter-processos.

Outro fator determinante foi a escalabilidade da plataforma. Embora a arquitetura inicial utilize um processador single-core, o FreeRTOS suporta igualmente arquiteturas multi-core através da sua variante SMP (*Symmetric Multiprocessing*), permitindo uma futura evolução do sistema sem necessidade de substituição do RTOS.

Assim, embora o QNX represente a opção tecnicamente mais segura e o AUTOSAR a opção mais alinhada com os padrões industriais automóveis, o FreeRTOS foi considerado a escolha mais adequada para os objetivos deste projeto, oferecendo um equilíbrio favorável entre capacidade técnica, flexibilidade, custo e esforço de implementação.

## Comparação de RTOS

| Critério                            | FreeRTOS                           | AUTOSAR OS    | QNX Neutrino            |
| ----------------------------------- | ---------------------------------- | ------------- | ----------------------- |
| **Determinismo temporal**           | Elevado                            | Muito elevado | Muito elevado           |
| **Segurança funcional**             | Limitada (sem certificação nativa) | Muito elevada | Muito elevada           |
| **Isolamento de falhas**            | Reduzido (kernel monolítico)       | Moderado      | Excelente (microkernel) |
| **Suporte multicore**               | Sim (SMP)                          | Sim           | Sim                     |
| **Complexidade de desenvolvimento** | Baixa                              | Muito elevada | Elevada                 |
| **Requisitos de hardware**          | Muito reduzidos                    | Moderados     | Elevados                |
| **Custos de licenciamento**         | Gratuito                           | Elevados      | Elevados                |

## Resumo da decisão

Embora o QNX Neutrino apresente a arquitetura mais robusta e segura para sistemas automóveis críticos, os benefícios adicionais não justificam o aumento substancial de custo, complexidade e requisitos computacionais para o contexto deste projeto. Por sua vez, o AUTOSAR OS oferece uma forte integração com os processos industriais automóveis, mas a sua complexidade torna-o excessivamente pesado para um protótipo académico.

Outro motivo relevante para a escolha do FreeRTOS é o facto dos elementos deste grupo de trabalho terem forte interesse na aprendizagem deste OS e na sua aplicação em sistemas embebidos, o que torna a sua utilização particularmente adequada para um projeto académico como este.

Desta forma, o FreeRTOS foi selecionado como a solução que melhor satisfaz os requisitos funcionais da ECU de Gestão do Pelotão, garantindo previsibilidade temporal, facilidade de desenvolvimento, reduzidos requisitos de hardware e potencial de escalabilidade futura.


## 1.4.2. Funcionalidades Chave do RTOS Aplicadas à Gestão do Pelotão

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

O modelo de prioridades fixas do FreeRTOS é particularmente adequado porque a criticidade das tarefas é conhecida à partida e não varia durante a execução do sistema.



## 1.5 Tecnologias de Comunicação (SYOSY)

## 1.6. Conclusão

O sprint 1 foi focado na definição do design de arquitetura do sistema, na escolha do RTOS a utilizar e das suas funcionalidades chave, e na escolha das tecnologias de comunicação a utilizar. O sprint 2 será focado na continuação do trabalho realizado no sprint 1.


## Sprint 1 - PMDEV - Retrospectiva

O Sprint 1 estabeleceu as bases tecnológicas para o sistema de monitorização de *platooning*, centrando-se no design da arquitetura, na seleção do RTOS e na definição das tecnologias de comunicação.

**O que correu bem:**
- **Estratégia de Divisão de Tarefas:** A separação do trabalho por áreas de foco (Arquitetura, RTOS e Comunicação) permitiu um foco mais profundo de cada elemento da equipa, resultando em entregas mais detalhadas e de qualidade superior.
- **Decisões Sólidas:** Concluiu-se a seleção do RTOS e das tecnologias de comunicação que garantirão a fiabilidade e eficiência essenciais ao sistema.

**Desafios e Pontos de Melhoria:**
- **Comunicação Transversal:** O elevado foco individual alertou para a necessidade de manter um canal de comunicação constante entre todos os elementos, garantindo que as peças do projeto encaixam e integram na perfeição.
- **Lacunas na Arquitetura:** Existiram dificuldades concretas no design inicial, nomeadamente na alocação dos componentes pelas ECUs e na falta da elaboração do diagrama de estados (*state machine*).

---

## Sprint 2 - PMDEV - Planeamento 

O Sprint 2 arranca com um objetivo prioritário: o reajuste das três componentes principais do projeto (Arquitetura, RTOS e Comunicações). Tendo em conta as aprendizagens do Sprint 1, as iterações planeadas para a arquitetura exigirão uma reavaliação dos seus impactos no RTOS e nas tecnologias de comunicação. Este processo, a realizar logo no início do sprint, é fundamental para orientar as decisões posteriores e assegurar que a evolução do design do sistema se mantém coerente, robusta e alinhada com os objetivos globais.

Após esta consolidação inicial, o foco recairá sobre a segurança e a operação do sistema. Será conduzida uma análise HAZOP para identificar potenciais desvios e riscos, culminando na construção da respetiva tabela. Paralelamente, efetuar-se-á uma análise temporal das transferências de dados, com o intuito de avaliar a fiabilidade das comunicações e propor soluções de mitigação para eventuais falhas. No domínio do RTOS, o trabalho centrar-se-á na identificação das tarefas essenciais ao funcionamento do sistema, bem como na definição dos mecanismos de sincronização e das propriedades de escalonamento.

Por fim, o sprint encerrará com a elaboração de um relatório detalhado de progresso e com o planeamento das tarefas para o período 3, garantindo assim uma gestão eficaz e a continuidade fluida do trabalho da equipa.

Para visualização do planeamento deste sprint, apresentamos o seguinte diagrama de Gantt:

![Gantt](planning.png)
**Legenda: Laranja: Rodrigo Botelho | Amarelo: Martim Botelho | Verde: Rodrigo Pinto | Azul: Todos**

Através do gráfico é possível observar que foi mantida a estratégia do Sprint 1 de alocar cada módulo a um elemento específico da equipa, o que permitirá uma maior profundidade de análise e desenvolvimento em cada área. Todos os módulos começam com uma fase de revisão e reajuste em caso de necessidade, o que é fundamental para garantir que o trabalho realizado no sprint anterior é aproveitado e otimizado. Apesar de não estar formalizado no diagrama de Gantt, está previsto um *briefing* de alinhamento a meio do sprint, logo após a revisão da arquitetura, garantindo que as decisões tomadas nessa fase são bem comunicadas e integradas nas tarefas subsequentes.

---

## Sprint 2 - PMDEV - Retrospectiva

O Sprint 2 destacou-se pela consolidação robusta do projeto. A fase de reajuste inicial cumpriu o seu propósito, permitindo corrigir as lacunas da arquitetura identificadas no sprint anterior e estabelecendo um design muito mais sólido e integrado com o RTOS e as Comunicações.

Em termos de segurança e fiabilidade, o trabalho traduziu-se em resultados práticos: a análise HAZOP forneceu uma matriz clara de mitigação de riscos, enquanto a análise temporal das transferências de dados resultou em soluções concretas que aumentam substancialmente a robustez da rede. Relativamente ao RTOS, a equipa conseguiu fechar um mapeamento claro das tarefas e dos mecanismos de sincronização, garantindo a escalabilidade do sistema.

A grande vitória deste sprint, contudo, foi a evolução na dinâmica da equipa. A comunicação transversal melhorou de forma significativa, colmatando o principal desafio do Sprint 1. O *briefing* intercalar provou ser uma excelente ferramenta, assegurando que as peças desenvolvidas individualmente encaixavam com precisão. Em suma, o Sprint 2 transformou os desafios iniciais numa boa base para o futuro do projeto.

---

## Sprint 3 - PMDEV - Planeamento

VEVCA 21% - Using the result of the HAZOP procedure, perform a FMEA and then FTA
SYOSY 36% - Pseudocode/configurations/guidelines for setup and operation of the envisioned communications system
RTOPR 36% - Pseudocode/configurations/guidelines for the setup and operation of the envisioned real-time operating system support
PMDEV 7% - Project management report (reporting of Period 3)

O Sprint 3 tem como foco a concretização dos resultados obtidos no Sprint 2, com especial ênfase na análise de segurança e na operacionalização do sistema. A primeira etapa será a realização da Análise de Modos de Falha e Efeitos (FMEA), utilizando os resultados da análise HAZOP para identificar e classificar os modos de falha potenciais, bem como as suas consequências. Esta análise será complementada por uma Árvore de Falhas (FTA), que permitirá visualizar as relações entre os diferentes modos de falha e identificar os pontos críticos do sistema. Será também realizado o pseudocódigo e a configuração para a operacionalização do sistema de comunicações, garantindo que as soluções propostas no Sprint 2 são implementáveis e eficazes. Paralelamente, será desenvolvido o pseudocódigo e as diretrizes para a configuração do RTOS, assegurando que o sistema é capaz de suportar as tarefas identificadas e os mecanismos de sincronização definidos. O sprint culminará com a elaboração de um relatório detalhado de progresso, que documentará os resultados alcançados.

Para visualização do planeamento deste sprint, apresentamos o seguinte diagrama de Gantt:
![Gantt](planningsprint3.png)
**Legenda: Laranja: Rodrigo Botelho | Amarelo: Martim Botelho | Verde: Rodrigo Pinto | Azul: Todos**
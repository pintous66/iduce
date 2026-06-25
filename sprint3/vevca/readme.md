## 2. VEVCA

Neste sprint, foi realizada a Árvore de Falhas (FTA) e a Análise de Modos de Falha e Efeitos (FMEA), com base nos resultados da análise HAZOP do sprint anterior. A FTA foi usada para representar, de forma top-down, como os desvios identificados podem contribuir para uma operação incorreta do platoon. De seguida, a FMEA foi usada para analisar esses modos de falha, identificando causas, efeitos, mecanismos de deteção, severidade, probabilidade, dificuldade de deteção, RPN e ações recomendadas.

### Fault Tree Analysis (FTA)

A FTA foi construída a partir dos dez desvios identificados no HAZOP. Para evitar uma árvore demasiado grande e difícil de interpretar, a análise foi dividida numa árvore geral e em dez subárvores, uma para cada desvio.

A árvore geral apresenta o Top Event, definido como operação incorreta do platoon. Como os desvios H-01 a H-10 estão ligados a este evento através de uma gate OR, qualquer um deles pode contribuir para uma operação incorreta caso não seja devidamente detetado ou mitigado.

Para esta análise, + representa um gate OR e . representa um gate AND. Assim, para a árvore geral:

Operação incorreta do platoon = H-01 + H-02 + H-03 + H-04 + H-05 + H-06 + H-07 + H-08 + H-09 + H-10

Os minimal cut sets da árvore geral são:

{H-01}, {H-02}, {H-03}, {H-04}, {H-05}, {H-06}, {H-07}, {H-08}, {H-09}, {H-10}

A expansão completa da árvore geral ao nível dos eventos básicos não é apresentada como uma única lista, porque seria demasiado complexa. Em vez disso, os minimal cut sets são apresentados separadamente em cada subárvore H-01 a H-10.

#### H-01

![FTA H-01](./H-01/FTA_1.png)

H-01 = A01 . B01

A01 = A011 + A012 + A013 + A014 + A015

B01 = B011 + B012 + B013 + B014

H-01 = (A011 + A012 + A013 + A014 + A015) . (B011 + B012 + B013 + B014)

Minimal cut sets: {A011,B011}, {A011,B012}, {A011,B013}, {A011,B014}, {A012,B011}, {A012,B012}, {A012,B013}, {A012,B014}, {A013,B011}, {A013,B012}, {A013,B013}, {A013,B014}, {A014,B011}, {A014,B012}, {A014,B013}, {A014,B014}, {A015,B011}, {A015,B012}, {A015,B013}, {A015,B014}

#### H-02

![FTA H-02](./H-02/FTA_2.png)

H-02 = A02 . B02

A02 = A021 + A022 + A023 + A024

B02 = B021 + B022 + B023 + B024

H-02 = (A021 + A022 + A023 + A024) . (B021 + B022 + B023 + B024)

Minimal cut sets: {A021,B021}, {A021,B022}, {A021,B023}, {A021,B024}, {A022,B021}, {A022,B022}, {A022,B023}, {A022,B024}, {A023,B021}, {A023,B022}, {A023,B023}, {A023,B024}, {A024,B021}, {A024,B022}, {A024,B023}, {A024,B024}

#### H-03

![FTA H-03](./H-03/FTA_3.png)

H-03 = A03 . B03

A03 = A031 + A032 + A033 + A034

B03 = B031 + B032 + B033 + B034

H-03 = (A031 + A032 + A033 + A034) . (B031 + B032 + B033 + B034)

Minimal cut sets: {A031,B031}, {A031,B032}, {A031,B033}, {A031,B034}, {A032,B031}, {A032,B032}, {A032,B033}, {A032,B034}, {A033,B031}, {A033,B032}, {A033,B033}, {A033,B034}, {A034,B031}, {A034,B032}, {A034,B033}, {A034,B034}

#### H-04

![FTA H-04](./H-04/FTA_4.png)

H-04 = A04 . B04

A04 = A041 + A042 + A043 + A044

B04 = B041 + B042 + B043 + B044

H-04 = (A041 + A042 + A043 + A044) . (B041 + B042 + B043 + B044)

Minimal cut sets: {A041,B041}, {A041,B042}, {A041,B043}, {A041,B044}, {A042,B041}, {A042,B042}, {A042,B043}, {A042,B044}, {A043,B041}, {A043,B042}, {A043,B043}, {A043,B044}, {A044,B041}, {A044,B042}, {A044,B043}, {A044,B044}

#### H-05

![FTA H-05](./H-05/FTA_5.png)

H-05 = A05 . B05

A05 = A051 + A052 + A053 + A054

B05 = B051 + B052 + B053 + B054

H-05 = (A051 + A052 + A053 + A054) . (B051 + B052 + B053 + B054)

Minimal cut sets: {A051,B051}, {A051,B052}, {A051,B053}, {A051,B054}, {A052,B051}, {A052,B052}, {A052,B053}, {A052,B054}, {A053,B051}, {A053,B052}, {A053,B053}, {A053,B054}, {A054,B051}, {A054,B052}, {A054,B053}, {A054,B054}

#### H-06

![FTA H-06](./H-06/FTA_6.png)

H-06 = A06 . B06 . C06

B06 = B061 + B062 + B063 + B064

C06 = C061 + C062 + C063 + C064

H-06 = A06 . (B061 + B062 + B063 + B064) . (C061 + C062 + C063 + C064)

Minimal cut sets: {A06,B061,C061}, {A06,B061,C062}, {A06,B061,C063}, {A06,B061,C064}, {A06,B062,C061}, {A06,B062,C062}, {A06,B062,C063}, {A06,B062,C064}, {A06,B063,C061}, {A06,B063,C062}, {A06,B063,C063}, {A06,B063,C064}, {A06,B064,C061}, {A06,B064,C062}, {A06,B064,C063}, {A06,B064,C064}

#### H-07

![FTA H-07](./H-07/FTA_7.png)

H-07 = A07 . B07

A07 = A071 + A072 + A073 + A074

B07 = B071 + B072 + B073 + B074

H-07 = (A071 + A072 + A073 + A074) . (B071 + B072 + B073 + B074)

Minimal cut sets: {A071,B071}, {A071,B072}, {A071,B073}, {A071,B074}, {A072,B071}, {A072,B072}, {A072,B073}, {A072,B074}, {A073,B071}, {A073,B072}, {A073,B073}, {A073,B074}, {A074,B071}, {A074,B072}, {A074,B073}, {A074,B074}

#### H-08

![FTA H-08](./H-08/FTA_8.png)

H-08 = A08 . B08

A08 = A081 + A082 + A083 + A084

B08 = B081 + B082 + B083 + B084 + B085

H-08 = (A081 + A082 + A083 + A084) . (B081 + B082 + B083 + B084 + B085)

Minimal cut sets: {A081,B081}, {A081,B082}, {A081,B083}, {A081,B084}, {A081,B085}, {A082,B081}, {A082,B082}, {A082,B083}, {A082,B084}, {A082,B085}, {A083,B081}, {A083,B082}, {A083,B083}, {A083,B084}, {A083,B085}, {A084,B081}, {A084,B082}, {A084,B083}, {A084,B084}, {A084,B085}

#### H-09

![FTA H-09](./H-09/FTA_9.png)

H-09 = A09 . B09

A09 = A091 + A092 + A093 + A094

B09 = B091 + B092 + B093 + B094 + B095

H-09 = (A091 + A092 + A093 + A094) . (B091 + B092 + B093 + B094 + B095)

Minimal cut sets: {A091,B091}, {A091,B092}, {A091,B093}, {A091,B094}, {A091,B095}, {A092,B091}, {A092,B092}, {A092,B093}, {A092,B094}, {A092,B095}, {A093,B091}, {A093,B092}, {A093,B093}, {A093,B094}, {A093,B095}, {A094,B091}, {A094,B092}, {A094,B093}, {A094,B094}, {A094,B095}

#### H-10

![FTA H-10](./H-10/FTA_10.png)

H-10 = A10 . B10

A10 = A101 + A102 + A103 + A104 + A105 + A106

B10 = B101 + B102 + B103 + B104

H-10 = (A101 + A102 + A103 + A104 + A105 + A106) . (B101 + B102 + B103 + B104)

Minimal cut sets: {A101,B101}, {A101,B102}, {A101,B103}, {A101,B104}, {A102,B101}, {A102,B102}, {A102,B103}, {A102,B104}, {A103,B101}, {A103,B102}, {A103,B103}, {A103,B104}, {A104,B101}, {A104,B102}, {A104,B103}, {A104,B104}, {A105,B101}, {A105,B102}, {A105,B103}, {A105,B104}, {A106,B101}, {A106,B102}, {A106,B103}, {A106,B104}

### Failure Mode and Effects Analysis (FMEA)

#### Escalas usadas na FMEA

Como não existem dados reais de operação para o sistema, os valores de **Severidade**, **Probabilidade** e **Deteção** foram atribuídos de forma qualitativa, usando uma escala de 1 a 10.

O **RPN** é calculado por:

```text
RPN = Severidade × Probabilidade × Deteção
```

##### Escala de Severidade

| Valor | Classificação      | Descrição                                                                                       |
| ----: | ------------------ | ----------------------------------------------------------------------------------------------- |
|    10 | Perigosamente alta | A falha pode provocar colisão, perda de controlo ou impossibilidade de parar o veículo/platoon. |
|     9 | Extremamente alta  | A falha pode comprometer diretamente a segurança do platoon e exigir reação imediata.           |
|     8 | Muito alta         | A falha pode tornar o comportamento do veículo inseguro ou imprevisível.                        |
|     7 | Alta               | A falha pode causar degradação significativa da operação ou da coordenação do platoon.          |
|     6 | Moderada           | A falha pode afetar parcialmente a operação, mas ainda existe alguma capacidade de controlo.    |
|     5 | Baixa              | A falha reduz o desempenho do sistema, mas não causa imediatamente uma situação crítica.        |
|     4 | Muito baixa        | A falha pode ser compensada pelo sistema, com impacto limitado na operação.                     |
|     3 | Menor              | A falha causa apenas perturbações menores no funcionamento.                                     |
|     2 | Muito menor        | A falha tem impacto reduzido e facilmente controlável.                                          |
|     1 | Nenhuma            | A falha não tem impacto relevante na operação do sistema.                                       |

##### Escala de Probabilidade

| Valor | Classificação      | Descrição                                                  |
| ----: | ------------------ | ---------------------------------------------------------- |
|    10 | Quase certa        | A falha é esperada com muita frequência.                   |
|     9 | Muito alta         | A falha é muito provável em condições normais de operação. |
|     8 | Alta               | A falha pode ocorrer com alguma frequência.                |
|     7 | Moderadamente alta | A falha pode ocorrer em vários cenários operacionais.      |
|     6 | Moderada           | A falha pode ocorrer ocasionalmente.                       |
|     5 | Baixa              | A falha pode ocorrer, mas não é esperada com frequência.   |
|     4 | Muito baixa        | A falha é pouco provável.                                  |
|     3 | Remota             | A falha é improvável, mas possível.                        |
|     2 | Muito remota       | A falha é muito improvável.                                |
|     1 | Quase impossível   | A falha é extremamente improvável.                         |

##### Escala de Deteção

| Valor | Classificação      | Descrição                                                                           |
| ----: | ------------------ | ----------------------------------------------------------------------------------- |
|    10 | Incerta            | A falha dificilmente será detetada antes de afetar o sistema.                       |
|     9 | Muito remota       | A falha tem baixa probabilidade de ser detetada a tempo.                            |
|     8 | Remota             | A falha pode passar despercebida em vários cenários.                                |
|     7 | Muito baixa        | A deteção depende de verificações limitadas ou pouco frequentes.                    |
|     6 | Baixa              | A falha pode ser detetada, mas não de forma garantida.                              |
|     5 | Moderada           | Existem mecanismos de deteção, mas estes podem falhar em alguns casos.              |
|     4 | Moderadamente alta | A falha é normalmente detetada por verificações automáticas ou validações cruzadas. |
|     3 | Alta               | A falha tem grande probabilidade de ser detetada antes de causar impacto.           |
|     2 | Muito alta         | A falha é quase sempre detetada automaticamente.                                    |
|     1 | Quase certa        | A falha é detetada de forma praticamente imediata e fiável.                         |

A FMEA foi realizada com base nos eventos básicos identificados nas subárvores FTA. Cada linha corresponde a um modo de falha detalhado da árvore, mantendo a ligação entre os nós da FTA e a análise de risco.

A seguinte tabela apresenta uma seleção dos modos de falha com maior RPN, correspondendo aproximadamente ao top 20% dos eventos mais críticos identificados nas FTAs.

| Componente                        | Modo de falha                                                        | Efeito                                                                                                    | Causa                                                                | Severidade | Probabilidade | Deteção | RPN | Ação recomendada                                                              |
| --------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- | ---------- | ------------- | -------- | --- | ----------------------------------------------------------------------------- |
| Processamento de perceção         | [A021] Mau tempo                                              | Obstáculos ou limites da via podem ser detetados incorretamente | Chuva, nevoeiro ou baixa visibilidade                            | 10         | 5             | 5       | 250 | Validar perceção com sensores redundantes                    |
| Processamento de perceção         | [A024] Falha no algoritmo de perceção                         | Obstáculos podem ser classificados ou detetados incorretamente  | Erro de software, modelo ou caso não previsto                    | 10         | 4             | 6       | 240 | Testar o algoritmo com cenários variados                     |
| Processamento de perceção         | [A022] Ruído nos sensores                                     | Objetos podem ser detetados incorretamente                      | Interferência, leituras instáveis ou ruído externo               | 9          | 5             | 5       | 225 | Filtrar interferências e comparar sensores                   |
| PredMaint / política de segurança | [C062] Dados incertos não são classificados como mais graves  | Sistema pode continuar quando deveria parar                     | Política fail-safe inexistente                                   | 10         | 3             | 7       | 210 | Tratar incerteza como condição mais grave                    |
| PlatMgmt                          | [A091] Dados desatualizados                                   | Visão do platoon pode ficar incorreta                           | Mensagens antigas ou atraso de atualização                       | 10         | 4             | 5       | 200 | Rejeitar dados com timestamp expirado                        |
| Validação de sensores             | [B012] Ausência de dados não é detetada                       | VC pode usar dados inexistentes                                 | Timeout ou validação dos dados mal definida                      | 10         | 3             | 6       | 180 | Definir timeout e estado inválido para dados ausentes        |
| VC                                | [B014] Decisões com dados inválidos não são bloqueadas        | Comandos podem ser calculados com dados inválidos               | Falta de condição de bloqueio ou fallback no VC                  | 10         | 3             | 6       | 180 | Bloquear decisões quando a perceção estiver inválida         |
| Processamento de perceção         | [A023] Erro de calibração                                     | Posição de objetos ou limites da via pode ficar incorreta       | Calibração incorreta ou desvio após uso                          | 9          | 4             | 5       | 180 | Implementar validação periódica de calibração                |
| Validação de perceção             | [B021] Comparação entre sensores falha                        | Informação incorreta pode ser aceite                            | Erro na lógica de comparação de sensores                         | 10         | 3             | 6       | 180 | Validar a comparação entre sensores                          |
| VC                                | [B024] Informação incorreta é enviada ao VC                   | VC pode tomar uma decisão insegura                              | Falha no filtro antes do envio ao VC                             | 10         | 3             | 6       | 180 | Rejeitar dados inconsistentes antes do envio                 |
| VC / validação de movimento       | [B044] Medições incoerentes não são rejeitadas                | VC pode gerar comandos inseguros                                | Falta de regra de rejeição de incoerências                       | 10         | 3             | 6       | 180 | Rejeitar medições incoerentes antes do controlo              |
| PredMaint                         | [B061] Regra de classificação mal definida                    | Avaria crítica pode ser tratada como não crítica                | Critérios de severidade incorretos                               | 10         | 3             | 6       | 180 | Rever regras de classificação                                |
| PredMaint                         | [B062] Falha no algoritmo                                     | Classificação de severidade pode ficar errada                   | Erro de implementação ou caso não previsto                       | 10         | 3             | 6       | 180 | Testar algoritmo com casos críticos                          |
| COMM / líder                      | [C063] Líder não é informado                                  | Platoon pode não reagir à condição crítica                      | Falha de envio ou notificação ao líder                           | 10         | 3             | 6       | 180 | Informar o líder em qualquer condição crítica ou incerta     |
| PlatMgmt                          | [B084] Seguidor não é marcado como não fiável                 | Líder pode confiar num veículo sem comunicação                  | Falha na gestão de estado do seguidor                            | 10         | 3             | 6       | 180 | Marcar seguidor como não fiável após timeout                 |
| PlatMgmt / validação              | [B091] Timestamps não são verificados                          | Mensagens antigas podem ser aceites                             | Validação temporal ausente                                       | 10         | 3             | 6       | 180 | Verificar timestamps antes da atualização                    |
| Atuadores                         | [B101] Confirmação dos atuadores não é recebida               | Falha de execução pode não ser detetada                         | Mecanismo de confirmação ausente ou falha                        | 10         | 3             | 6       | 180 | Exigir confirmação após comandos críticos                    |

Os modos de falha com maior RPN concentram-se na perceção, validação de dados, classificação de avarias, comunicação com o líder e confirmação dos atuadores. Isto mostra que os principais riscos não resultam apenas das falhas técnicas iniciais, mas também da ausência de deteção ou validação perante essas falhas.

### Apêndice A - Tabela FMEA

| Componente                        | Modo de falha                                                 | Efeito                                                          | Causa                                                            | Severidade | Probabilidade | Deteção | RPN | Ação recomendada                                             |
| --------------------------------- | ------------------------------------------------------------- | --------------------------------------------------------------- | ---------------------------------------------------------------- | ---------- | ------------- | ------- | --- | ------------------------------------------------------------ |
| Sensores de perceção              | [A011] Falha de sensor                                        | Dados de perceção ficam indisponíveis ou incompletos            | Avaria física, alimentação instável ou falha no sensor           | 9          | 4             | 3       | 108 | Detetar ausência de dados e avisar o VC                      |
| Sensores de perceção              | [A012] Câmara obstruída                                       | Obstáculos ou limites da via podem não ser detetados            | Condições atmosféricas adversas ou obstáculo a bloquear o sensor | 9          | 5             | 3       | 135 | Verificar visibilidade e obstrução da câmara                 |
| Sensores de perceção              | [A013] Falha do LiDAR                                         | Capacidade de deteção de obstáculos fica reduzida               | Avaria do LiDAR ou alimentação instável                          | 9          | 3             | 4       | 108 | Validar LiDAR com câmara e ultrassons                        |
| Sensores de perceção              | [A014] Falha de cablagem                                      | Dados dos sensores podem não chegar à ECU                       | Cabo danificado ou ligação instável                              | 8          | 3             | 4       | 96  | Adicionar diagnóstico de ligação                             |
| ECU / entrada de sensores         | [A015] Falha de entrada na ECU                                | Dados dos sensores podem não chegar corretamente ao VC          | Falha de interface ou porta de entrada da ECU                    | 9          | 3             | 5       | 135 | Validar entradas da ECU e gerar erro quando não houver dados |
| Validação de sensores             | [B011] Verificação automática dos sensores falha              | Ausência de dados pode não ser detetada                         | Erro na lógica de diagnóstico dos sensores                       | 10         | 3             | 5       | 150 | Validar a verificação automática dos sensores                |
| Validação de sensores             | [B012] Ausência de dados não é detetada                       | VC pode usar dados inexistentes                                 | Timeout ou validação dos dados mal definida                      | 10         | 3             | 6       | 180 | Definir timeout e estado inválido para dados ausentes        |
| VC                                | [B013] VC não é avisado                                       | VC pode continuar a decidir sem perceção válida                 | Falha na comunicação interna entre diagnóstico e VC              | 10         | 3             | 5       | 150 | Garantir notificação obrigatória ao VC                       |
| VC                                | [B014] Decisões com dados inválidos não são bloqueadas        | Comandos podem ser calculados com dados inválidos               | Falta de condição de bloqueio ou fallback no VC                  | 10         | 3             | 6       | 180 | Bloquear decisões quando a perceção estiver inválida         |
| Processamento de perceção         | [A021] Mau tempo                                              | Obstáculos ou limites da via podem ser detetados incorretamente | Chuva, nevoeiro ou baixa visibilidade                            | 10         | 5             | 5       | 250 | Validar perceção com sensores redundantes                    |
| Processamento de perceção         | [A022] Ruído nos sensores                                     | Objetos podem ser detetados incorretamente                      | Interferência, leituras instáveis ou ruído externo               | 9          | 5             | 5       | 225 | Filtrar interferências e comparar sensores                   |
| Processamento de perceção         | [A023] Erro de calibração                                     | Posição de objetos ou limites da via pode ficar incorreta       | Calibração incorreta ou desvio após uso                          | 9          | 4             | 5       | 180 | Implementar validação periódica de calibração                |
| Processamento de perceção         | [A024] Falha no algoritmo de perceção                         | Obstáculos podem ser classificados ou detetados incorretamente  | Erro de software, modelo ou caso não previsto                    | 10         | 4             | 6       | 240 | Testar o algoritmo com cenários variados                     |
| Validação de perceção             | [B021] Comparação entre sensores falha                        | Informação incorreta pode ser aceite                            | Erro na lógica de comparação de sensores                         | 10         | 3             | 6       | 180 | Validar a comparação entre sensores                          |
| Validação de perceção             | [B022] Objeto não confirmado por mais do que um sensor        | Objeto falso ou incorreto pode ser aceite                       | Regra de confirmação inexistente ou mal definida                 | 10         | 3             | 5       | 150 | Exigir confirmação por sensores independentes                |
| Validação de perceção             | [B023] Limite da via não confirmado por mais do que um sensor | Trajetória pode ser calculada com limite errado                 | Falta de validação cruzada dos limites da via                    | 9          | 3             | 5       | 135 | Confirmar limites da via com múltiplos sensores              |
| VC                                | [B024] Informação incorreta é enviada ao VC                   | VC pode tomar uma decisão insegura                              | Falha no filtro antes do envio ao VC                             | 10         | 3             | 6       | 180 | Rejeitar dados inconsistentes antes do envio                 |
| Localização                       | [A031] Erro de GPS                                            | Posição do veículo pode ficar incorreta                         | Erro do recetor GPS ou erro de sinal                             | 8          | 4             | 4       | 128 | Comparar GPS com movimento do veículo                        |
| Localização                       | [A032] Perda de sinal GPS                                     | Localização fica indisponível ou instável                       | Túnel, interferência ou zona sem cobertura                       | 8          | 4             | 3       | 96  | Usar dados de movimento como fallback                        |
| Localização                       | [A033] Inconsistência com o mapa                              | Rota ou posição pode ficar incoerente                           | Mapa desatualizado ou erro de correspondência                    | 8          | 3             | 5       | 120 | Validar posição com mapa e sensores                          |
| Localização                       | [A034] Falha no algoritmo de localização                      | Posição calculada pode ficar errada                             | Erro de software ou caso não previsto                            | 9          | 3             | 6       | 162 | Testar algoritmo de localização em cenários adversos         |
| Validação de localização          | [B031] Comparação GPS/velocidade falha                        | Erro de posição pode não ser detetado                           | Falha na validação cruzada                                       | 9          | 3             | 5       | 135 | Comparar continuamente GPS e velocidade das rodas            |
| Validação de localização          | [B032] Comparação com movimento falha                         | Posição incoerente pode ser aceite                              | Dados de movimento não usados na validação                       | 9          | 3             | 5       | 135 | Usar movimento recente para confirmar localização            |
| NAV / PlatMgmt                    | [B033] Dados de movimento recentes não são usados             | Sistema perde fallback quando o GPS não é fiável                | Fallback não implementado ou não ativado                         | 8          | 3             | 5       | 120 | Ativar fallback com dados recentes                           |
| PlatMgmt                          | [B034] PlatMgmt não é avisado da posição não fiável           | Estado do platoon pode ser atualizado com posição errada        | Falha na notificação ao PlatMgmt                                 | 9          | 3             | 5       | 135 | Avisar PlatMgmt quando a posição não for fiável              |
| Sensores de movimento             | [A041] Falha no sensor de velocidade das rodas                | Velocidade estimada pode ficar incorreta                        | Avaria do sensor ou sinal inválido                               | 9          | 4             | 4       | 144 | Validar velocidade com limites e histórico                   |
| Sensores de movimento             | [A042] Falha no sensor de direção                             | Ângulo de direção pode ficar incorreto                          | Avaria do sensor de direção                                      | 9          | 3             | 4       | 108 | Validar direção com leituras anteriores                      |
| Sensores de movimento             | [A043] Erro de calibração                                     | Estado de movimento pode ficar enviesado                        | Calibração incorreta dos sensores                                | 8          | 4             | 5       | 160 | Executar calibração e validação periódica                    |
| Sensores de movimento             | [A044] Leituras desatualizadas                                | VC pode usar estado antigo do veículo                           | Timestamp expirado ou atraso na leitura                          | 9          | 4             | 4       | 144 | Rejeitar leituras com timestamp expirado                     |
| Validação de movimento            | [B041] Verificação de limites falha                           | Medições impossíveis podem ser aceites                          | Limites mal definidos ou não aplicados                           | 9          | 3             | 5       | 135 | Definir limites aceitáveis para velocidade e direção         |
| Validação de movimento            | [B042] Timestamps não são verificados                         | Leituras antigas podem ser usadas                               | Ausência de validação temporal                                   | 9          | 3             | 5       | 135 | Verificar timestamps antes de usar medições                  |
| Validação de movimento            | [B043] Comparação com leituras anteriores falha               | Variações incoerentes podem ser aceites                         | Falha na lógica de comparação temporal                           | 8          | 3             | 5       | 120 | Comparar com histórico recente                               |
| VC                                | [B044] Medições incoerentes não são rejeitadas                | VC pode gerar comandos inseguros                                | Falta de regra de rejeição de incoerências                       | 10         | 3             | 6       | 180 | Rejeitar medições incoerentes antes do controlo              |
| Veículo / diagnóstico             | [A051] Problema na pressão dos pneus                          | Veículo pode operar com aderência ou estabilidade reduzida      | Perda de pressão ou furo                                         | 8          | 4             | 4       | 128 | Monitorizar pressão em cada ciclo                            |
| Veículo / diagnóstico             | [A052] Temperatura anormal do motor                           | Powertrain pode degradar ou falhar                              | Sobreaquecimento ou falha de arrefecimento                       | 8          | 3             | 4       | 96  | Monitorizar temperatura e gerar aviso                        |
| Veículo / diagnóstico             | [A053] Problema no estado dos travões                         | Capacidade de paragem pode ficar reduzida                       | Desgaste, falha hidráulica ou sensor de travão                   | 10         | 3             | 4       | 120 | Monitorizar estado dos travões                               |
| Veículo / diagnóstico             | [A054] Falha física do veículo                                | Veículo pode continuar em condição insegura                     | Falha mecânica ou elétrica não especificada                      | 9          | 3             | 5       | 135 | Executar diagnóstico periódico completo                      |
| PredMaint                         | [B051] Falha no PredMaint                                     | Avaria pode não ser detetada                                    | Erro no módulo ou falha de execução                              | 9          | 3             | 5       | 135 | Validar execução do PredMaint                                |
| PredMaint                         | [B052] Falta de dados de diagnóstico                          | PredMaint pode não conseguir avaliar o veículo                  | Sensores de diagnóstico indisponíveis                            | 9          | 4             | 4       | 144 | Gerar erro quando faltarem dados de diagnóstico              |
| PredMaint                         | [B053] Falha de sensor                                        | Estado de saúde pode ser avaliado incorretamente                | Avaria nos sensores de diagnóstico                               | 8          | 4             | 4       | 128 | Comparar sensores e detetar leituras inválidas               |
| PredMaint                         | [B054] Verificações periódicas não detetam a falha            | Falha pode permanecer ativa sem mitigação                       | Período de verificação inadequado ou regra incompleta            | 9          | 3             | 6       | 162 | Rever periodicidade e regras de diagnóstico                  |
| Veículo / diagnóstico             | [A06] Existe uma avaria crítica                               | Platoon deve iniciar reação segura                              | Falha crítica real no veículo                                    | 10         | 3             | 4       | 120 | Detetar e sinalizar avarias críticas rapidamente             |
| PredMaint                         | [B061] Regra de classificação mal definida                    | Avaria crítica pode ser tratada como não crítica                | Critérios de severidade incorretos                               | 10         | 3             | 6       | 180 | Rever regras de classificação                                |
| PredMaint                         | [B062] Falha no algoritmo                                     | Classificação de severidade pode ficar errada                   | Erro de implementação ou caso não previsto                       | 10         | 3             | 6       | 180 | Testar algoritmo com casos críticos                          |
| PredMaint                         | [B063] Dados desatualizados                                   | Classificação pode usar estado antigo                           | Timestamp expirado ou atraso nos dados                           | 9          | 4             | 5       | 180 | Rejeitar dados desatualizados                                |
| PredMaint                         | [B064] Entrada incorreta de sensores                          | Severidade pode ser calculada com dados errados                 | Sensor com leitura inválida                                      | 9          | 4             | 5       | 180 | Validar entradas antes da classificação                      |
| Validação de diagnóstico          | [C061] Valores dos sensores não são verificados               | Dados inválidos podem ser usados na classificação               | Falha na verificação de plausibilidade                           | 10         | 3             | 6       | 180 | Verificar valores contra limites esperados                   |
| PredMaint / política de segurança | [C062] Dados incertos não são classificados como mais graves  | Sistema pode continuar quando deveria parar                     | Política fail-safe inexistente                                   | 10         | 3             | 7       | 210 | Tratar incerteza como condição mais grave                    |
| COMM / líder                      | [C063] Líder não é informado                                  | Platoon pode não reagir à condição crítica                      | Falha de envio ou notificação ao líder                           | 10         | 3             | 6       | 180 | Informar o líder em qualquer condição crítica ou incerta     |
| PlatMgmt                          | [C064] Estado do platoon não é atualizado                     | Decisão de paragem pode não ser iniciada                        | Falha na atualização do estado                                   | 10         | 3             | 6       | 180 | Atualizar estado do platoon após classificação crítica       |
| COMM                              | [A071] Atraso na rede                                         | Estado de avaria pode chegar tarde ao líder                     | Latência elevada na comunicação                                  | 8          | 5             | 3       | 120 | Definir limite máximo de atraso                              |
| COMM                              | [A072] Perda de pacotes                                       | Mensagem de avaria pode chegar tarde ou incompleta              | Instabilidade da ligação                                         | 8          | 5             | 4       | 160 | Usar reenvio e confirmação                                   |
| COMM                              | [A073] Congestionamento de comunicação                        | Mensagens críticas podem sofrer atraso                          | Excesso de tráfego V2V ou na ECU                                 | 8          | 4             | 4       | 128 | Priorizar mensagens críticas                                 |
| ECU                               | [A074] Sobrecarga da ECU                                      | Processamento ou envio de mensagens pode atrasar                | Carga elevada ou tarefas concorrentes                            | 8          | 3             | 5       | 120 | Monitorizar carga e priorizar tarefas críticas               |
| COMM / validação temporal         | [B071] Timestamps não são verificados                         | Mensagem atrasada pode ser aceite                               | Ausência de validação temporal                                   | 9          | 3             | 5       | 135 | Verificar timestamps nas mensagens                           |
| COMM / validação temporal         | [B072] Atraso não é detetado                                  | Líder pode usar informação atrasada                             | Timeout inexistente ou mal definido                              | 9          | 3             | 5       | 135 | Detetar atraso com timeout                                   |
| COMM / PlatMgmt                   | [B073] Tempo máximo de receção não está definido              | Sistema pode não saber quando assumir perda de informação       | Requisito temporal incompleto                                    | 9          | 3             | 6       | 162 | Definir tempo máximo de receção                              |
| PlatMgmt                          | [B074] Líder não assume perda de informação                   | Decisão de paragem pode ser atrasada                            | Fallback não implementado                                        | 9          | 3             | 6       | 162 | Assumir perda de informação quando o limite for excedido     |
| COMM                              | [A081] Falha de comunicação                                   | Mensagem do seguidor pode não chegar ao líder                   | Falha no canal V2V                                               | 9          | 4             | 4       | 144 | Monitorizar ligação V2V                                      |
| COMM / hardware                   | [A082] Falha da antena                                        | Comunicação V2V pode ficar indisponível                         | Avaria física da antena                                          | 9          | 3             | 5       | 135 | Diagnosticar antena e estado da ligação                      |
| COMM                              | [A083] Perda de pacotes                                       | Mensagem de estado pode não ser recebida                        | Interferência ou instabilidade da rede                           | 8          | 5             | 4       | 160 | Usar ACK e reenvio                                           |
| ECU de comunicação                | [A084] Falha da ECU de comunicação                            | Seguidor pode deixar de comunicar o estado                      | Falha de hardware ou software na ECU                             | 9          | 3             | 5       | 135 | Monitorizar saúde da ECU de comunicação                      |
| COMM                              | [B081] Mensagens periódicas de presença falham                | Perda de comunicação pode não ser detetada                      | Heartbeat não implementado ou falha no envio                     | 9          | 3             | 5       | 135 | Enviar mensagens periódicas de presença                      |
| COMM                              | [B082] ACK não é recebido ou processado                       | Sistema pode não confirmar receção de mensagens críticas        | Falha no mecanismo de ACK                                        | 9          | 3             | 5       | 135 | Usar ACK obrigatório para mensagens críticas                 |
| COMM                              | [B083] Reenvio de mensagens críticas falha                    | Mensagem perdida pode não ser recuperada                        | Lógica de reenvio fraca ou inexistente                           | 9          | 3             | 6       | 162 | Implementar reenvio com limite temporal                      |
| PlatMgmt                          | [B084] Seguidor não é marcado como não fiável                 | Líder pode confiar num veículo sem comunicação                  | Falha na gestão de estado do seguidor                            | 10         | 3             | 6       | 180 | Marcar seguidor como não fiável após timeout                 |
| PlatMgmt                          | [B085] Estado do platoon não é atualizado                     | Visão do platoon pode ficar incompleta                          | Falha de atualização após perda de comunicação                   | 10         | 3             | 6       | 180 | Atualizar estado do platoon após falha de comunicação        |
| PlatMgmt                          | [A091] Dados desatualizados                                   | Visão do platoon pode ficar incorreta                           | Mensagens antigas ou atraso de atualização                       | 10         | 4             | 5       | 200 | Rejeitar dados com timestamp expirado                        |
| PlatMgmt                          | [A092] Dados atribuídos ao veículo errado                     | Estado de um veículo pode ser aplicado a outro                  | Erro de identificação ou origem                                  | 10         | 3             | 6       | 180 | Validar origem e ID da mensagem                              |
| PlatMgmt                          | [A093] Erro de sincronização                                  | Estados dos veículos podem ficar incoerentes                    | Falha de sincronização temporal                                  | 9          | 3             | 6       | 162 | Sincronizar e validar timestamps                             |
| PlatMgmt                          | [A094] Mensagem corrompida                                    | Estado do platoon pode ser atualizado com dados inválidos       | Corrupção de dados na comunicação                                | 9          | 3             | 5       | 135 | Validar integridade da mensagem                              |
| PlatMgmt / validação              | [B091] Timestamps não são verificados                         | Mensagens antigas podem ser aceites                             | Validação temporal ausente                                       | 10         | 3             | 6       | 180 | Verificar timestamps antes da atualização                    |
| PlatMgmt / validação              | [B092] Origem das mensagens não é validada                    | Dados podem ser atribuídos ao veículo errado                    | Validação de origem ausente                                      | 10         | 3             | 6       | 180 | Validar origem das mensagens                                 |
| PlatMgmt / validação              | [B093] Coerência entre estados não é analisada                | Estado inconsistente pode ser aceite                            | Falha de plausibilidade global                                   | 10         | 3             | 6       | 180 | Comparar estados entre veículos                              |
| PlatMgmt / validação              | [B094] Mensagens antigas não são rejeitadas                   | Visão do platoon pode ficar desatualizada                       | Regra de rejeição inexistente                                    | 10         | 3             | 6       | 180 | Rejeitar mensagens antigas                                   |
| PlatMgmt / validação              | [B095] Mensagens corrompidas não são rejeitadas               | Dados inválidos podem afetar decisões                           | Validação de integridade insuficiente                            | 10         | 3             | 6       | 180 | Rejeitar mensagens corrompidas                               |
| VC                                | [A101] Falha no VC                                            | Comando de paragem ou controlo pode não ser gerado              | Falha de software no VC                                          | 10         | 3             | 5       | 150 | Monitorizar execução do VC e usar fallback                   |
| ECU do atuador                    | [A102] Falha na ECU do atuador                                | Atuador pode não executar o comando                             | Falha de hardware ou software na ECU                             | 10         | 3             | 5       | 150 | Diagnosticar ECU do atuador                                  |
| Interface VC-atuador              | [A103] Falha de interface                                     | Comando pode não chegar ao atuador                              | Erro de comunicação interna ou interface                         | 10         | 3             | 5       | 150 | Validar interface e confirmação de comando                   |
| Atuador de direção                | [A104] Falha na direção                                       | Manobra segura pode não ser executada corretamente              | Avaria no sistema de direção                                     | 10         | 2             | 5       | 100 | Monitorizar feedback da direção                              |
| Atuador de travagem               | [A105] Falha na travagem                                      | Veículo pode não conseguir parar                                | Avaria no sistema de travagem                                    | 10         | 2             | 5       | 100 | Monitorizar feedback da travagem                             |
| Powertrain                        | [A106] Falha no powertrain                                    | Veículo pode não reduzir ou ajustar movimento corretamente      | Avaria no powertrain                                             | 9          | 2             | 5       | 90  | Monitorizar feedback do powertrain                           |
| Atuadores                         | [B101] Confirmação dos atuadores não é recebida               | Falha de execução pode não ser detetada                         | Mecanismo de confirmação ausente ou falha                        | 10         | 3             | 6       | 180 | Exigir confirmação após comandos críticos                    |
| Atuadores                         | [B102] Feedback dos atuadores não é validado                  | Sistema pode assumir execução sem confirmação real              | Validação de feedback insuficiente                               | 10         | 3             | 6       | 180 | Validar feedback de direção, travagem e powertrain           |
| VC                                | [B103] Sistema não repete o comando                           | Falha temporária pode continuar sem recuperação                 | Lógica de repetição não implementada                             | 10         | 3             | 5       | 150 | Repetir comando quando não houver confirmação                |
| COMM / líder                      | [B104] Líder não é alertado                                   | Platoon pode não reagir à falha de execução                     | Falha de alerta ao líder                                         | 10         | 3             | 6       | 180 | Alertar líder se comando crítico falhar                      |

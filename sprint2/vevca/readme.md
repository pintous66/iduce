# VEVCA

## Platoon Monitoring

O Platoon Monitoring é um sistema de monitorização para veículos autónomos em platoon. O veículo líder mantém uma visão atualizada do estado de todos os veículos e, ao identificar uma potencial avaria, coordena a manobra de paragem segura na berma da estrada. Cada veículo é composto por sensores, atuadores e um conjunto de módulos de controlo (VC, NAV, PredMaint, PlatMgmt e COMM) que comunicam entre si e com os restantes veículos do platoon.

No primeiro sprint foi definida a arquitetura do sistema, identificados os requisitos e modelado o comportamento através de um diagrama de atividades. Neste segundo sprint, com o design consolidado e algumas melhorias no trabalho já realizado, foi feito um estudo HAZOP para identificar possíveis perigos e falhas.

## Melhorias Sprint 1

### Descrição dos componentes

- **Sensores** recolhem dados do ambiente e do estado físico do veículo - perceção (câmara, LiDAR, ultrassons), localização e movimento (GPS, velocidade, direção) e diagnóstico (pressão dos pneus, temperatura, travões).
- **NAV** recebe os dados de localização e movimento e calcula/atualiza a rota do veículo.
- **PredMaint** recebe os dados de diagnóstico e movimento, avalia o estado de saúde do veículo e classifica-o como normal, aviso ou crítico.
- **VC** é o núcleo de decisão de cada veículo. Recebe os dados de perceção, a rota do NAV, o estado do PredMaint e o estado do platoon, toma decisões de controlo de curto prazo e envia comandos para os atuadores (direção, travagem, powertrain).
- **COMM** gere a comunicação V2V. No seguidor, transmite o estado de saúde ao líder. No líder, partilha as decisões de manobra entre os seguidores.
- **PlatMgmt** mantém uma visão atualizada da posição, velocidade e estado de cada veículo do platoon, e quando deteta uma condição crítica, faz com que o platoon se encaminhe para a berma da estrada.

### Fluxo de dados

| Origem | Dados | Destino |
| --- | --- | --- |
| Câmaras, LiDAR, ultrassons | Dados de perceção | VC |
| GPS | Localização | VC, NAV |
| Sensor de velocidade das rodas | Velocidade das rodas | VC, NAV, COMM, PredMaint |
| Sensor de direção | Ângulo de direção | VC, COMM |
| Sensores de pressão, temperatura e travões | Dados de diagnóstico | PredMaint |
| NAV | Rota calculada | VC |
| PredMaint | Classificação do estado do veículo | VC |
| PredMaint | Estado de saúde (aviso/crítico) | COMM (seguidor) |
| PredMaint | Estado de saúde (aviso/crítico) | PlatMgmt (líder) |
| COMM (seguidor) | Mensagem de estado V2V | COMM (líder) |
| COMM (líder) | Estado recebido dos seguidores | PlatMgmt |
| PlatMgmt | Decisão de manobra segura | VC (líder) |
| VC (líder) | Comando de manobra a difundir | COMM (líder) |
| COMM (líder) | Comando de manobra V2V | COMM (seguidor) |
| COMM (seguidor) | Comando de manobra recebido | VC (seguidor) |
| VC | Comandos de direção, travagem, powertrain | Atuadores |

### Requisitos

Os requisitos foram identificados e classificados em requisitos funcionais (F) e requisitos de qualidade (Q). Os requisitos funcionais descrevem comportamentos esperados do sistema, enquanto os requisitos de qualidade descrevem propriedades relevantes como desempenho, fiabilidade e isolamento de falhas.

| ID | Tipo | Texto |
| --- | --- | --- |
| F-01 | Funcional | O sistema deve recolher dados de câmaras, LiDAR e sensores ultrassónicos para suportar a perceção do ambiente envolvente do veículo. |
| F-02 | Funcional | O sistema deve recolher dados de GPS, velocidade das rodas e direção para representar o estado de localização e movimento do veículo. |
| F-03 | Funcional | O sistema deve recolher dados de pressão dos pneus, temperatura do motor e estado dos travões para monitorizar o estado físico do veículo. |
| F-04 | Funcional | O módulo NAV deve calcular ou atualizar a rota do veículo com base em dados de localização e movimento. |
| F-05 | Funcional | O módulo VC deve tomar decisões de curto prazo com base nos dados de perceção, localização, movimento, direção, rota e estado do platoon. |
| F-06 | Funcional | O módulo VC deve enviar comandos para os atuadores de direção, travagem e powertrain. |
| F-07 | Funcional | O módulo PredMaint deve avaliar o estado do veículo com base nos dados de diagnóstico e movimento. |
| F-08 | Funcional | O módulo PredMaint deve classificar o estado do veículo como normal, aviso ou crítico. |
| F-09 | Funcional | Quando um veículo seguidor detetar uma condição de aviso ou crítico, o sistema deve enviar essa informação ao veículo líder através do módulo COMM. |
| F-10 | Funcional | O módulo PlatMgmt deve manter uma visão atualizada da posição, direção, velocidade e estado de cada veículo do platoon. |
| F-11 | Funcional | Quando o PlatMgmt identificar uma condição crítica, deve decidir a interrupção da operação normal e enviar essa decisão ao VC do líder. |
| F-12 | Funcional | Quando a operação normal for interrompida pelo líder, o sistema deve comunicar essa decisão aos veículos seguidores através do módulo COMM. |
| F-13 | Funcional | Cada veículo seguidor deve executar a manobra recebida, ajustando direção, travagem e powertrain através do seu próprio VC. |
| Q-01 | Qualidade | O módulo VC deve executar decisões de curto prazo dentro da janela temporal definida para o controlo do veículo. |
| Q-02 | Qualidade | O sistema deve ignorar dados de sensores ou mensagens inter-veículo com timestamp expirado. |
| Q-03 | Qualidade | O sistema deve comparar os dados de sensores entre si para detetar valores incorretos e evitar que decisões sejam tomadas com base em leituras inválidas. |

### Diagrama de arquitetura do sistema

[Diagrama de arquitetura do sistema](./diagrama_arquitetura.png)

### Diagrama de atividades

![Diagrama de atividades](./diagrama_atividades.png)

## HAZOP (Hazard and Operability Study)

Depois de ter o design do sistema quase fechado, realizámos um estudo HAZOP para identificar possíveis perigos e falhas no sistema. Este estudo tem como objetivo identificar desvios face ao comportamento esperado do sistema, analisar as suas causas e consequências, definir ações para controlar ou reduzir os riscos identificados, e garantir que esses problemas são registados e acompanhados ao longo do desenvolvimento.

### Etapas do HAZOP

1. Dividir o sistema em secções para análise, como por exemplo, sensores, comunicação, controlo do veículo e atuadores.
1. Escolher o nó para o estudo, como por exemplo, o atuador de travagem.
1. Descrever a função do nó, como por exemplo, o atuador de travagem deve executar o comando enviado pelo VC para reduzir a velocidade ou parar o veículo.
1. Escolher um parâmetro para o nó, como por exemplo, o comando de travagem ou a força de travagem aplicada.
1. Escolher uma palavra-guia para o parâmetro, como por exemplo, "Não", "Errado" ou "Tarde".
1. Identificar o desvio resultante da combinação entre o parâmetro e a palavra-guia, como por exemplo, o comando de travagem não é executado.
1. Determinar as causas do desvio, como por exemplo, falha na ECU de travagem, falha de interface, avaria no atuador ou perda do comando enviado pelo VC.
1. Avaliar os problemas/consequências do desvio, como por exemplo, o veículo pode não conseguir reduzir a velocidade ou parar após uma avaria crítica.
1. Definir a ação recomendada (o que fazer, quando e quem).
1. Guardar a informação.
1. Repetir a partir do segundo passo.

### Desvios identificados

Com base nos principais nós da arquitetura proposta, foram identificados alguns desvios relevantes para análise. Estes desvios incluem:

- H-01: Perda de dados de perceção provenientes das câmaras, LiDAR ou sensores ultrassónicos.
- H-02: Deteção incorreta de obstáculos ou limites da via pelo subsistema de perceção.
- H-03: Localização incorreta do veículo devido a erros de GPS ou posicionamento.
- H-04: Estimativa incorreta do estado de movimento do veículo devido a falhas nos sensores de velocidade das rodas ou direção.
- H-05: Falha do PredMaint na deteção de uma avaria do veículo.
- H-06: Classificação incorreta de uma avaria crítica como não crítica.
- H-07: Transmissão tardia do estado de avaria de um veículo seguidor para o veículo líder.
- H-08: Perda de comunicação entre veículos seguidores e o veículo líder.
- H-09: Estado incorreto do platoon mantido pelo PlatMgmt.
- H-10: Falha na execução de comandos de paragem ou controlo através da direção, travagem ou powertrain.

### HAZOP

#### Palavras-guia

| Palavra-guia   | Significado                                          | Exemplo no sistema                                                      |
| -------------- | ---------------------------------------------------- | ----------------------------------------------------------------------- |
| Não            | A intenção de funcionamento não é cumprida           | O comando de travagem não é executado                                   |
| Mais           | Existe um aumento quantitativo num parâmetro         | A velocidade estimada é superior à velocidade real                      |
| Menos          | Existe uma diminuição quantitativa num parâmetro     | A força de travagem aplicada é inferior à necessária                    |
| Além de        | Ocorre uma atividade adicional não esperada          | O veículo envia uma mensagem duplicada ou adicional ao líder            |
| Parte de       | Apenas parte da intenção de funcionamento é cumprida | Apenas alguns sensores de perceção enviam dados válidos                 |
| Inverso        | Ocorre o oposto da intenção de funcionamento         | O atuador acelera quando deveria reduzir velocidade                     |
| Outro / Errado | Ocorre uma substituição ou interpretação incorreta   | A posição, estado ou severidade da avaria é interpretada incorretamente |
| Cedo           | A ação ocorre antes do momento correto               | O seguidor aplica uma manobra antes da decisão do líder                 |
| Tarde          | A ação ocorre depois do momento correto              | O estado de avaria do seguidor chega demasiado tarde ao líder           |

#### Análise HAZOP

| Nó/Função | Parâmetro | Palavra-guia | Desvio | Possíveis causas | Possíveis consequências | Salvaguardas existentes/propostas | Recomendações |
|---|---|---|---|---|---|---|---|
| Sensores de perceção | Dados de câmaras, LiDAR e sensores ultrassónicos | Não | Sem dados de perceção disponíveis | Falha de sensor, câmara obstruída, falha do LiDAR, falha de cablagem, falha de entrada na ECU | O veículo pode não detetar obstáculos, limites da via ou veículos próximos | Verificação automática dos sensores e deteção de ausência de dados | Avisar o VC quando os dados de perceção não estiverem disponíveis e impedir decisões baseadas nesses dados |
| Processamento de perceção | Objetos / limites da via detetados | Errado | Obstáculos ou limites da via são detetados incorretamente | Mau tempo, ruído nos sensores, erro de calibração, falha no algoritmo de perceção | Decisão incorreta de trajetória, distância insegura entre veículos, possível colisão | Comparação entre câmara, LiDAR e sensores ultrassónicos | Confirmar objetos e limites da via com mais do que um sensor antes de enviar a informação ao VC |
| Localização | Posição do veículo | Errado | A posição do veículo é estimada incorretamente | Erro de GPS, perda de sinal, inconsistência com o mapa, falha no algoritmo de localização | Seguimento incorreto da trajetória ou estimativa incorreta da posição do veículo no platoon | Comparação entre GPS, velocidade das rodas e dados de movimento | Se a posição não for fiável, usar dados de movimento recentes e avisar o PlatMgmt |
| Sensores de movimento | Velocidade / ângulo de direção | Errado | O estado de movimento do veículo é estimado incorretamente | Falha no sensor de velocidade das rodas, falha no sensor de direção, erro de calibração, leituras desatualizadas | O VC pode calcular comandos inseguros de aceleração, travagem ou direção | Verificação de limites aceitáveis, timestamps e comparação com leituras anteriores | Rejeitar medições fora dos valores esperados ou incoerentes com o estado anterior do veículo |
| Manutenção preditiva | Estado de saúde do veículo | Não | A avaria não é detetada | Falha no PredMaint, falta de dados de diagnóstico, falha de sensor | O veículo continua a operar com uma falha não detetada | Verificações periódicas do estado do veículo | Verificar pressão dos pneus, temperatura do motor e estado dos travões em cada "ciclo" de monitorização |
| Manutenção preditiva | Severidade da avaria | Errado | Avaria crítica classificada como não crítica | Regra de classificação mal definida, falha no algoritmo, dados desatualizados, entrada incorreta de sensores | O platoon pode continuar a operar quando deveria parar | Verificação dos valores dos sensores e comparação com valores esperados | Quando os dados forem incertos ou incompletos, classificar a situação como mais grave e informar o líder |
| Comunicação seguidor-líder | Tempo da mensagem de estado | Tarde | O estado de avaria do seguidor chega demasiado tarde ao líder | Atraso na rede, perda de pacotes, congestionamento de comunicação, sobrecarga da ECU | O líder atualiza o estado do platoon tarde demais e pode atrasar a decisão de paragem | Timestamps nas mensagens e deteção de atraso | Definir um tempo máximo para receber mensagens de avaria, se esse tempo for excedido, o líder deve assumir perda de informação |
| Comunicação seguidor-líder | Mensagem de estado | Não | A mensagem de estado do seguidor não é recebida pelo líder | Falha de comunicação, falha da antena, perda de pacotes, falha da ECU de comunicação | O líder fica com uma visão incompleta do platoon e pode não reagir à avaria do seguidor | Mensagens periódicas de presença, confirmação de receção (ACK) e reenvio de mensagens críticas | Se o líder deixar de receber mensagens de um seguidor, deve marcar esse veículo como não fiável e atualizar o estado do platoon |
| Gestão do platoon | Visão do estado do platoon | Errado | O líder mantém uma visão incorreta do platoon | Dados desatualizados, dados atribuídos ao veículo errado, erro de sincronização, mensagem corrompida | O líder pode tomar uma decisão insegura ao nível do platoon | Verificação de timestamps, origem das mensagens e coerência entre estados dos veículos | Rejeitar mensagens antigas, corrompidas ou incoerentes antes de atualizar a visão do platoon |
| Controlo do veículo / atuadores | Comando de paragem / controlo | Não | O comando de paragem ou controlo não é executado | Falha no VC, falha na ECU do atuador, falha de interface, falha na direção, travagem ou powertrain | O veículo ou o platoon pode não conseguir parar após uma avaria crítica | Confirmação de comandos e feedback dos atuadores | Exigir confirmação dos atuadores, se a confirmação não for recebida, o sistema deve repetir o comando e alertar o líder |

Com base nos desvios identificados e nas recomendações definidas, alguns requisitos existentes foram refinados e novos requisitos de qualidade foram adicionados para garantir que o sistema responde adequadamente às falhas identificadas.
# VEVCA

## Requisitos

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
| F-09 | Funcional | Quando um veículo seguidor detetar uma condição warning ou critical, o sistema deve enviar essa informação ao veículo líder através do módulo COMM. |
| F-10 | Funcional | O módulo PlatMgmt deve manter uma visão atualizada da posição, direção, velocidade e estado de cada veículo do platoon. |
| F-11 | Funcional | Quando o PlatMgmt identificar uma condição crítica, deve fornecer ao VC do líder uma decisão de manobra segura. |
| Q-01 | Qualidade | O módulo VC deve executar decisões de curto prazo dentro da janela temporal definida para o controlo do veículo. |
| Q-02 | Qualidade | O sistema deve ignorar dados de sensores ou mensagens inter-veículo com timestamp expirado. |
| Q-03 | Qualidade | O sistema deve ser capaz de lidar com falhas de sensores ou comunicação sem comprometer a segurança do veículo. |

## Máquina de Estados

![Máquina de Estados](./maquina_estado.png)

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

## Desvios identificados

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

| Nó/Função                       | Parâmetro                                        | Palavra-guia | Desvio                                                        | Possíveis causas                                                                                                 | Possíveis consequências                                                                     | Salvaguardas existentes/propostas                                                                | Recomendações                                                                                                                    |
| ------------------------------- | ------------------------------------------------ | ------------ | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| Sensores de perceção            | Dados de câmaras, LiDAR e sensores ultrassónicos | Não          | Sem dados de perceção disponíveis                             | Falha de sensor, câmara obstruída, falha do LiDAR, falha de cablagem, falha de entrada na ECU                    | O veículo pode não detetar obstáculos, limites da via ou veículos próximos                  | Diagnóstico dos sensores, redundância entre sensores de perceção, deteção por timeout            | Adicionar monitorização do estado dos sensores e mudar para modo degradado quando os dados de perceção não estiverem disponíveis |
| Processamento de perceção       | Objetos / limites da via detetados               | Errado       | Obstáculos ou limites da via são detetados incorretamente     | Mau tempo, ruído nos sensores, erro de calibração, falha no algoritmo de perceção                                | Decisão incorreta de trajetória, distância insegura entre veículos, possível colisão        | Verificação cruzada entre câmara, LiDAR e sensores ultrassónicos; verificações de plausibilidade | Exigir validação multi-sensor antes de usar dados de perceção em decisões de controlo                                            |
| Localização                     | Posição do veículo                               | Errado       | A posição do veículo é estimada incorretamente                | Erro de GPS, perda de sinal, inconsistência com o mapa, falha no algoritmo de localização                        | Seguimento incorreto da trajetória ou estimativa incorreta da posição do veículo no platoon | Validação do GPS, comparação com velocidade das rodas, estimação de confiança                    | Adicionar monitorização da confiança da posição e estratégia alternativa de localização                                          |
| Sensores de movimento           | Velocidade / ângulo de direção                   | Errado       | O estado de movimento do veículo é estimado incorretamente    | Falha no sensor de velocidade das rodas, falha no sensor de direção, erro de calibração, leituras desatualizadas | O VC pode calcular comandos inseguros de aceleração, travagem ou direção                    | Verificações de gama, verificação de timestamps, verificações de plausibilidade                  | Comparar dados de movimento com valores anteriores e rejeitar medições inconsistentes                                            |
| Manutenção preditiva            | Estado de saúde do veículo                       | Não          | A avaria não é detetada                                       | Falha de software no PredMaint, falta de dados de diagnóstico, falha de sensor                                   | O veículo continua a operar com uma falha não detetada                                      | Monitorização de diagnóstico, verificações periódicas de saúde, watchdog                         | Adicionar verificações obrigatórias para pressão dos pneus, temperatura do motor e estado dos travões                            |
| Manutenção preditiva            | Severidade da avaria                             | Errado       | Avaria crítica classificada como não crítica                  | Limiar incorreto, falha no algoritmo, dados desatualizados, entrada incorreta de sensores                        | O platoon pode continuar a operar quando deveria parar                                      | Limiares de severidade, verificação de frescura dos dados, regras de validação                   | Definir regras conservadoras de classificação de severidade para dados incertos ou inconsistentes                                |
| Comunicação seguidor-líder      | Tempo da mensagem de estado                      | Tarde        | O estado de avaria do seguidor chega demasiado tarde ao líder | Atraso na rede, perda de pacotes, congestionamento de comunicação, sobrecarga da ECU                             | O líder atualiza o estado do platoon tarde demais e pode atrasar a decisão de paragem       | Timestamps nas mensagens, timeout de comunicação, mensagens prioritárias                         | Definir um atraso máximo aceitável para mensagens de avaria                                                                      |
| Comunicação seguidor-líder      | Mensagem de estado                               | Não          | A mensagem de estado do seguidor não é recebida pelo líder    | Falha de comunicação, falha da antena, perda de pacotes, falha da ECU de comunicação                             | O líder fica com uma visão incompleta do platoon e pode não reagir à avaria do seguidor     | Mensagens heartbeat, deteção por timeout, retransmissão                                          | Ativar modo degradado se o estado de um seguidor estiver ausente durante um período definido                                     |
| Gestão do platoon               | Visão do estado do platoon                       | Errado       | O líder mantém uma visão incorreta do platoon                 | Dados desatualizados, ID de veículo incorreto, erro de sincronização, mensagem corrompida                        | O líder pode tomar uma decisão insegura ao nível do platoon                                 | Validação de timestamps, verificação de IDs dos veículos, verificações de consistência           | Forçar atualizações sincronizadas do estado do platoon e descartar mensagens desatualizadas                                      |
| Controlo do veículo / atuadores | Comando de paragem / controlo                    | Não          | O comando de paragem ou controlo não é executado              | Falha no VC, falha na ECU do atuador, falha de interface, falha na direção, travagem ou powertrain               | O veículo ou o platoon pode não conseguir parar após uma avaria crítica                     | Feedback dos atuadores, watchdog, confirmação de comandos                                        | Exigir confirmação da execução dos comandos pelos atuadores e entrar em modo fail-safe se a execução falhar                      |

A análise HAZOP mostra que os principais riscos do sistema estão associados à perda ou incorreção de dados sensoriais, atrasos na comunicação entre veículos, classificação incorreta de avarias e falha na execução de comandos críticos. As recomendações propostas focam-se em validação de dados, deteção por timeout, redundância, confirmação de comandos e entrada em modo degradado ou fail-safe quando a informação disponível não é fiável.

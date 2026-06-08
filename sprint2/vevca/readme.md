### Requisitos

Os requisitos foram identificados e classificados em requisitos funcionais (F) e requisitos de qualidade (Q). Os requisitos funcionais descrevem comportamentos esperados do sistema, enquanto os requisitos de qualidade descrevem propriedades relevantes como desempenho, fiabilidade, isolamento de falhas e verificabilidade.

| ID | Tipo | Texto |
| --- | --- | --- |
| F-01 | Funcional | O sistema deve recolher dados de câmaras, LiDAR e sensores ultrassónicos para suportar a perceção do ambiente envolvente do veículo. |
| F-02 | Funcional | O sistema deve recolher dados de GPS, velocidade das rodas e direção para representar o estado de localização e movimento do veículo. |
| F-03 | Funcional | O sistema deve recolher dados de pressão dos pneus, temperatura do motor e estado dos travões para monitorizar o estado físico do veículo. |
| F-04 | Funcional | O módulo NAV deve calcular ou atualizar a rota do veículo com base em dados de localização e movimento. |
| F-05 | Funcional | O módulo VC deve tomar decisões de curto prazo com base nos dados de perceção, localização, movimento, direção, rota e estado do platoon. |
| F-06 | Funcional | O módulo VC deve enviar comandos para os atuadores de direção, travagem e powertrain. |
| F-07 | Funcional | O módulo PredMaint deve avaliar o estado do veículo com base nos dados de diagnóstico e movimento. |
| F-08 | Funcional | O módulo PredMaint deve classificar o estado do veículo como normal, warning ou critical. |
| F-09 | Funcional | Quando um veículo seguidor detetar uma condição warning ou critical, o sistema deve enviar essa informação ao veículo líder através do módulo COMM. |
| F-10 | Funcional | O módulo PlatMgmt deve manter uma visão atualizada da posição, direção, velocidade e estado de cada veículo do platoon. |
| F-11 | Funcional | Quando o PlatMgmt identificar uma condição crítica, deve fornecer ao VC do líder uma decisão de manobra segura. |
| Q-01 | Qualidade | O módulo VC deve executar decisões de curto prazo dentro da janela temporal definida para o controlo do veículo. |
| Q-02 | Qualidade | O sistema deve ignorar dados de sensores ou mensagens inter-veículo com timestamp expirado. |
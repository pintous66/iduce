# Sprint 3: Setup e operação do FreeRTOS - Platoon Management (RTOPR)

## 1. Recap técnico para configuração FreeRTOS

### Arquitetura assumida

| Core | Módulo |
|---|---|
| Core 0 | VC |
| Core 1 | PlatMgmt |

Apesar de neste sprint as tarefas implementadas sejam apenas as do `PlatMgmt`, a configuração considera uma ECU dual-core com FreeRTOS SMP. Desta forma, as tarefas do `PlatMgmt` são atribuídas ao `Core 1` através de core affinity, enquanto o `Core 0` fica disponível para uma futura implementação das tarefas do `VC`.

### Tarefas do PlatMgmt

| Tarefa | Prioridade | Período | Execução | Prazo | Recursos |
|---|---:|---:|---:|---:|---|
| `Task_PlatMgmt_COMM_RX` | Alta | 50 ms | 10 ms | 50 ms | B |
| `Task_PlatMgmt_Update` | Média-Alta | 100 ms | 20 ms | 100 ms | A, B, C |
| `Task_PlatMgmt_COMM_TX` | Média | 100 ms | 15 ms | 100 ms | C |
| `Task_PredMaint_Leader` | Baixa | 1000 ms | 50 ms | 1000 ms | A, C |

### Recursos partilhados

| ID | Recurso | Tipo FreeRTOS | Produtor(es) | Consumidor(es) |
|---|---|---|---|---|
| A | Estado do Pelotão / `PlatoonState` | Estrutura global + `Mutex` | `Task_PlatMgmt_Update` | `Task_PredMaint_Leader` |
| B | `RX_MessageQueue` | `Queue` | `Task_PlatMgmt_COMM_RX` | `Task_PlatMgmt_Update` |
| C | `TX_MessageQueue` | `Queue` | `Task_PlatMgmt_Update`, `Task_PredMaint_Leader` | `Task_PlatMgmt_COMM_TX` |

## 2. Configuração do FreeRTOS

A configuração do FreeRTOS foi definida no ficheiro `FreeRTOSConfig.h`, de acordo com a documentação oficial do FreeRTOS: [FreeRTOS Customization](https://www.freertos.org/Documentation/02-Kernel/03-Supported-devices/02-Customization).

Este ficheiro permite parametrizar o comportamento do kernel para a aplicação desenvolvida, sem alterar diretamente o código-fonte do FreeRTOS. Neste projeto, a configuração foi escolhida tendo em conta a utilização de escalonamento preemptivo, tarefas periódicas, sincronização entre tarefas e alocação estática de memória.

Para este projeto, foi considerada a seguinte configuração:

```c

#define configUSE_PREEMPTION 1 
#define configUSE_TIME_SLICING 0 
#define configTICK_RATE_HZ 1000 
#define configMAX_PRIORITIES 6 
#define configSUPPORT_STATIC_ALLOCATION 1 
#define configSUPPORT_DYNAMIC_ALLOCATION 0 
#define configUSE_MUTEXES 1 
#define configUSE_TICKLESS_IDLE 0 
#define configCHECK_FOR_STACK_OVERFLOW 2 
#define INCLUDE_vTaskDelayUntil 1 
#define INCLUDE_xTaskGetTickCount 1
#define configNUMBER_OF_CORES 2
#define configUSE_CORE_AFFINITY 1

```

| Parâmetro                          |    Valor | Função                                                                  | Aplicação no projeto                                                                                                                   |
| ---------------------------------- | -------: | ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `configUSE_PREEMPTION`             |      `1` | Ativa o escalonamento preemptivo.                                       | Permite que tarefas de maior prioridade interrompam tarefas menos prioritárias, garantindo resposta rápida da `Task_PlatMgmt_COMM_RX`. |
| `configUSE_TIME_SLICING`           |      `0` | Desativa a alternância automática entre tarefas com a mesma prioridade. | Torna o comportamento temporal mais previsível, uma vez que o sistema usa prioridades fixas e períodos bem definidos.                  |
| `configTICK_RATE_HZ`               |   `1000` | Define a frequência do tick do sistema.                                 | Garante uma resolução temporal de 1 ms, adequada para os períodos de 50 ms, 100 ms e 1000 ms.                                          |
| `configMAX_PRIORITIES`             |      `6` | Define o número máximo de níveis de prioridade disponíveis.             | Permite separar as tarefas do PlatMgmt por níveis de prioridade distintos.                                                             |
| `configSUPPORT_STATIC_ALLOCATION`  |      `1` | Permite a criação de recursos com alocação estática.                    | Suporta uma configuração mais determinística, com recursos definidos no arranque do sistema.                                           |
| `configSUPPORT_DYNAMIC_ALLOCATION` |      `0` | Desativa a alocação dinâmica de memória.                                | Evita fragmentação de memória e variações imprevisíveis no tempo de execução.                                                          |
| `configUSE_MUTEXES`                |      `1` | Ativa o uso de mutexes.                                                 | Permite proteger o acesso ao `PlatoonState`, evitando acessos concorrentes incorretos.                                                 |
| `configUSE_TICKLESS_IDLE`          |      `0` | Desativa o modo tickless idle.                                          | Mantém o tick periódico ativo, tornando a temporização do sistema mais previsível.                                                     |
| `configCHECK_FOR_STACK_OVERFLOW`   |      `2` | Ativa a verificação de overflow da stack.                               | Ajuda a detetar falhas de configuração ou consumo excessivo de stack pelas tarefas.                                                    |
| `INCLUDE_vTaskDelayUntil`          |      `1` | Disponibiliza a função `vTaskDelayUntil()`.                             | Necessário para implementar tarefas periódicas com períodos fixos.                                                                  |
| `INCLUDE_xTaskGetTickCount`        |      `1` | Disponibiliza a função `xTaskGetTickCount()`.                           | Permite obter o tempo atual do sistema em ticks, útil para controlo temporal e timestamps internos.                                    |
| `configNUMBER_OF_CORES`             |      `2` | Define o número de cores disponíveis no sistema.                        | Permite configurar o FreeRTOS para suportar a execução em múltiplos núcleos, com tarefas atribuídas a cores específicas.              |
| `configUSE_CORE_AFFINITY`          |      `1` | Ativa a afinidade de tarefas a cores específicos.                       | Permite associar as tarefas do `PlatMgmt` ao `Core 1`, deixando o `Core 0` disponível para uma futura implementação das tarefas do `VC`.                                   |

Como a alocação dinâmica está desativada, os recursos RTOS são criados estaticamente durante a inicialização, antes do arranque do escalonador.

## 3. Pseudo-código

Esta secção apresenta o pseudo-código operacional do suporte RTOS previsto para o módulo `PlatMgmt`. O objetivo é mostrar como as tarefas usam as primitivas principais do FreeRTOS para criar recursos, comunicar através de filas de mensagens, proteger recursos partilhados com mutexes e manter uma execução periódica.

---

### 3.1 Parâmetros usados no pseudo-código

Antes da criação dos recursos e das tarefas, são definidos os principais parâmetros usados pelo pseudo-código. Estes valores incluem o tamanho das filas, o tamanho das stacks, as prioridades das tarefas, os períodos de execução e os parâmetros de afinidade aos cores usados durante a operação.

Os valores apresentados correspondem a uma configuração inicial coerente com a análise temporal definida anteriormente. Numa implementação real, estes valores deveriam ser validados através de testes de carga, monitorização do uso de stack e análise do comportamento temporal do sistema.

```text id="4r04r1"
// Tamanho das filas
TAMANHO_FILA_RX = 16
TAMANHO_FILA_TX = 8

// Tamanho das stacks das tarefas
// Nota: em FreeRTOS, estes valores são normalmente expressos em palavras de StackType_t,
// e não diretamente em bytes.
STACK_COMM_RX = 512
STACK_PLAT_UPDATE = 768
STACK_COMM_TX = 512
STACK_PREDMAINT = 1024

// Prioridades das tarefas
PRIORIDADE_ALTA = 4
PRIORIDADE_MEDIA_ALTA = 3
PRIORIDADE_MEDIA = 2
PRIORIDADE_BAIXA = 1

// Períodos das tarefas
PERIODO_COMM_RX_MS = 50
PERIODO_UPDATE_MS = 100
PERIODO_COMM_TX_MS = 100
PERIODO_PREDMAINT_MS = 1000

// Cores e máscaras de afinidade
CORE_VC = 0
CORE_PLATMGMT = 1

AFFINITY_CORE_0 = 1 << CORE_VC
AFFINITY_CORE_1 = 1 << CORE_PLATMGMT
```

O `TAMANHO_FILA_RX` é maior do que o `TAMANHO_FILA_TX` porque a receção de mensagens V2V pode sofrer pequenos picos de tráfego ou variações temporais causadas pela rede. Assim, a fila de receção consegue armazenar temporariamente várias mensagens antes de estas serem processadas pela tarefa `Task_PlatMgmt_Update`.

O `TAMANHO_FILA_TX` pode ser menor porque existem menos mensagens a transmitir do que mensagens recebidas. Além disso, mensagens críticas podem ser colocadas na frente da fila através da primitiva `xQueueSendToFront()`.

As stacks das tarefas foram dimensionadas de acordo com a complexidade esperada de cada tarefa. A `Task_PlatMgmt_Update` usa uma stack maior porque processa mensagens, atualiza o estado global do pelotão e pode gerar comandos de paragem. A `Task_PredMaint_Leader` também tem uma stack maior porque executa análise periódica sobre uma cópia do estado do pelotão. As tarefas `Task_PlatMgmt_COMM_RX` e `Task_PlatMgmt_COMM_TX` usam stacks menores, pois executam operações mais diretas de receção e transmissão.

As prioridades seguem a política Rate Monotonic definida anteriormente. Assim, tarefas com menor período recebem maior prioridade. A tarefa `Task_PlatMgmt_COMM_RX`, com período de 50 ms, recebe a prioridade mais elevada, enquanto a tarefa `Task_PredMaint_Leader`, com período de 1000 ms, recebe a prioridade mais baixa.

O acesso ao `estadoPlatoon` é feito através de mutexes do FreeRTOS. Como estes mutexes suportam priority inheritance, uma tarefa de menor prioridade que esteja temporariamente a utilizar o mutex pode herdar a prioridade de uma tarefa mais crítica que esteja à espera desse recurso. Para reduzir o impacto temporal do bloqueio, as secções protegidas pelo mutex devem ser curtas, limitando-se à atualização ou cópia do estado partilhado.

Finalmente, as máscaras de afinidade indicam em que core uma tarefa pode executar. Neste caso, `AFFINITY_CORE_0 = 1 << CORE_VC` corresponde ao `Core 0`, enquanto `AFFINITY_CORE_1 = 1 << CORE_PLATMGMT` corresponde ao `Core 1`.

### 3.2 Inicialização dos Recursos RTOS

Antes de inicializar o escalonador, são criados os recursos partilhados usados pelas tarefas. A criação é feita com primitivas estáticas do FreeRTOS, evitando alocação dinâmica durante a execução.

```text
// Buffers estáticos das filas
bufferFilaRececao
bufferFilaTransmissao

memoriaFilaRececao[TAMANHO_FILA_RX * tamanho(MensagemV2V)]
memoriaFilaTransmissao[TAMANHO_FILA_TX * tamanho(MensagemV2V)]

// Buffer estático do mutex
bufferMutexEstadoPlatoon

// Handles dos recursos RTOS
filaRececao
filaTransmissao
mutexEstadoPlatoon

// Handles das tarefas
handleCommRx
handleUpdate
handleCommTx
handlePredMaint

// Estado global partilhado
estadoPlatoon

CriarRecursosRTOS()
{
    estadoPlatoon = inicializarEstadoPlatoon()

    filaRececao = xQueueCreateStatic(
        TAMANHO_FILA_RX,
        tamanho(MensagemV2V),
        memoriaFilaRececao,
        &bufferFilaRececao
    )

    filaTransmissao = xQueueCreateStatic(
        TAMANHO_FILA_TX,
        tamanho(MensagemV2V),
        memoriaFilaTransmissao,
        &bufferFilaTransmissao
    )

    mutexEstadoPlatoon = xSemaphoreCreateMutexStatic(
        &bufferMutexEstadoPlatoon
    )

    se filaRececao == NULL ou
       filaTransmissao == NULL ou
       mutexEstadoPlatoon == NULL:

        registarErro("falha na criação dos recursos RTOS")
        entrarModoSeguro()
}
```

A `filaRececao` armazena mensagens recebidas da comunicação V2V antes de serem processadas pela tarefa de atualização. Esta fila desacopla a receção da rede do processamento lógico do `PlatMgmt`, permitindo absorver pequenos picos de tráfego.

A `filaTransmissao` armazena mensagens que devem ser enviadas para a rede V2V. Esta fila permite que várias tarefas produzam mensagens, como `Task_PlatMgmt_Update` e `Task_PredMaint_Leader`, mantendo apenas uma tarefa responsável pela transmissão, a `Task_PlatMgmt_COMM_TX`.

O `mutexEstadoPlatoon` protege a estrutura global `estadoPlatoon`, evitando acessos concorrentes incorretos entre tarefas que leem ou atualizam o estado do pelotão.

---

### 3.3 Inicialização das Tarefas

As tarefas são criadas antes do início do escalonador através da primitiva `xTaskCreateStatic()`. As prioridades seguem a política Rate Monotonic definida anteriormente: tarefas com menor período recebem maior prioridade.

```text
// Stacks estáticas das tarefas
stackCommRx[STACK_COMM_RX]
stackUpdate[STACK_PLAT_UPDATE]
stackCommTx[STACK_COMM_TX]
stackPredMaint[STACK_PREDMAINT]

// Control blocks estáticos
tcbCommRx
tcbUpdate
tcbCommTx
tcbPredMaint

CriarTarefasRTOS()
{
    handleCommRx = xTaskCreateStatic(
        Task_PlatMgmt_COMM_RX,
        "COMM_RX",
        STACK_COMM_RX,
        NULL,
        PRIORIDADE_ALTA,
        stackCommRx,
        &tcbCommRx
    )

    handleUpdate = xTaskCreateStatic(
        Task_PlatMgmt_Update,
        "PLAT_UPDATE",
        STACK_PLAT_UPDATE,
        NULL,
        PRIORIDADE_MEDIA_ALTA,
        stackUpdate,
        &tcbUpdate
    )

    handleCommTx = xTaskCreateStatic(
        Task_PlatMgmt_COMM_TX,
        "COMM_TX",
        STACK_COMM_TX,
        NULL,
        PRIORIDADE_MEDIA,
        stackCommTx,
        &tcbCommTx
    )

    handlePredMaint = xTaskCreateStatic(
        Task_PredMaint_Leader,
        "PRED_LEADER",
        STACK_PREDMAINT,
        NULL,
        PRIORIDADE_BAIXA,
        stackPredMaint,
        &tcbPredMaint
    )

    vTaskCoreAffinitySet(handleCommRx, AFFINITY_CORE_1)
    vTaskCoreAffinitySet(handleUpdate, AFFINITY_CORE_1)
    vTaskCoreAffinitySet(handleCommTx, AFFINITY_CORE_1)
    vTaskCoreAffinitySet(handlePredMaint, AFFINITY_CORE_1)
}
```

A tarefa `Task_PlatMgmt_COMM_RX` tem prioridade mais alta porque recebe mensagens externas e pode influenciar rapidamente o estado global do pelotão. A tarefa `Task_PredMaint_Leader` tem prioridade mais baixa porque executa análise periódica e não pertence ao ciclo imediato de atuação.

---

### 3.4 Sequência de Arranque do RTOS

```text
RTOS_Inicializar()
{
    CriarRecursosRTOS()

    CriarTarefasRTOS()

    vTaskStartScheduler()
}
```

Após a chamada de `vTaskStartScheduler()`, o FreeRTOS passa a gerir a execução das tarefas de acordo com as prioridades e os períodos definidos.

---

### 3.5 Tarefa `Task_PlatMgmt_COMM_RX`

```text
Task_PlatMgmt_COMM_RX()
{
    ultimoAcordar = xTaskGetTickCount()
    mensagem

    enquanto verdadeiro:

        vTaskDelayUntil(&ultimoAcordar, pdMS_TO_TICKS(PERIODO_COMM_RX_MS))

        se receberDaRedeV2V(&mensagem) == SUCESSO:

            se validarIntegridade(mensagem) e não timestampExpirado(mensagem):

                se xQueueSend(filaRececao, &mensagem, 0) != pdPASS:
                    registarErro("filaRececao cheia")

            senão:

                descartarMensagem(mensagem)
                registarErro("mensagem V2V inválida ou expirada")
}
```

Esta tarefa usa `vTaskDelayUntil()` para manter um período fixo de 50 ms. As mensagens válidas são colocadas na `filaRececao` com `xQueueSend()`. A tarefa não atualiza diretamente o `estadoPlatoon`, evitando bloqueios desnecessários.

---

### 3.6 Tarefa `Task_PlatMgmt_Update`

```text
Task_PlatMgmt_Update()
{
    ultimoAcordar = xTaskGetTickCount()
    mensagem
    comandoParagem
    condicaoCritica

    enquanto verdadeiro:

        vTaskDelayUntil(&ultimoAcordar, pdMS_TO_TICKS(PERIODO_UPDATE_MS))

        enquanto xQueueReceive(filaRececao, &mensagem, 0) == pdPASS:

            se xSemaphoreTake(mutexEstadoPlatoon, portMAX_DELAY) == pdPASS:

                se mensagemCoerente(mensagem, estadoPlatoon):

                    atualizarEstadoVeiculo(&estadoPlatoon, mensagem)

                senão:

                    descartarMensagem(mensagem)
                    registarErro("mensagem incoerente com o estado atual")

                xSemaphoreGive(mutexEstadoPlatoon)

        condicaoCritica = falso

        se xSemaphoreTake(mutexEstadoPlatoon, portMAX_DELAY) == pdPASS:

            condicaoCritica = verificarCondicaoCritica(estadoPlatoon)

            xSemaphoreGive(mutexEstadoPlatoon)

        se condicaoCritica == verdadeiro:

            comandoParagem = criarComandoParagem()

            se xQueueSendToFront(filaTransmissao, &comandoParagem, 0) != pdPASS:

                registarErro("filaTransmissao cheia para comando crítico")
}
```

Esta tarefa lê as mensagens disponíveis na `filaRececao` usando `xQueueReceive()`. O acesso ao `estadoPlatoon` é protegido com `xSemaphoreTake()` e `xSemaphoreGive()`. Como a receção da fila é feita sem bloqueio, a tarefa apenas processa mensagens já disponíveis no momento da sua execução periódica.

Quando é detetada uma condição crítica, o comando de paragem é inserido na frente da `filaTransmissao` através de `xQueueSendToFront()`. Isto dá prioridade a mensagens críticas face a mensagens periódicas.

---

### 3.7 Tarefa `Task_PlatMgmt_COMM_TX`

```text
Task_PlatMgmt_COMM_TX()
{
    ultimoAcordar = xTaskGetTickCount()
    mensagem

    enquanto verdadeiro:

        vTaskDelayUntil(&ultimoAcordar, pdMS_TO_TICKS(PERIODO_COMM_TX_MS))

        se xQueueReceive(filaTransmissao, &mensagem, 0) == pdPASS:

            enviarParaRedeV2V(mensagem)

            se mensagem.prioridade == CRITICA:

                pedirConfirmacaoRececao(mensagem)
                gerirReenvioMensagemCritica(mensagem)
}
```

Esta tarefa é a única consumidora da `filaTransmissao`. Isto evita que várias tarefas tentem aceder diretamente à interface de comunicação ao mesmo tempo. Quando uma mensagem crítica é enviada, a tarefa solicita confirmação de receção e ativa o mecanismo de reenvio caso a confirmação não seja obtida.

---

### 3.8 Tarefa `Task_PredMaint_Leader`

```text
Task_PredMaint_Leader()
{
    ultimoAcordar = xTaskGetTickCount()
    copiaEstado
    mensagemSaude
    estadoSaude

    enquanto verdadeiro:

        vTaskDelayUntil(&ultimoAcordar, pdMS_TO_TICKS(PERIODO_PREDMAINT_MS))

        xSemaphoreTake(mutexEstadoPlatoon, portMAX_DELAY)

        copiaEstado = copiarEstadoPlatoon(estadoPlatoon)

        xSemaphoreGive(mutexEstadoPlatoon)

        estadoSaude = avaliarSaudeDoPlatoon(copiaEstado)

        se estadoSaude == CRITICO:

            mensagemSaude = criarMensagemCriticaSaude()

            se xQueueSendToFront(filaTransmissao, &mensagemSaude, 0) != pdPASS:

                registarErro("filaTransmissao cheia para mensagem crítica")

        senão se estadoSaude == AVISO:

            mensagemSaude = criarMensagemAvisoSaude()

            se xQueueSend(filaTransmissao, &mensagemSaude, 0) != pdPASS:

                registarErro("filaTransmissao cheia para mensagem de aviso")
}
```

Esta tarefa copia rapidamente o `estadoPlatoon` dentro de uma secção crítica e depois liberta o mutex. A análise de manutenção preditiva é feita sobre a cópia local, evitando manter o recurso partilhado bloqueado durante processamento mais pesado.

Se o estado for crítico, a mensagem é enviada para a frente da `filaTransmissao`. Se for apenas aviso, é colocada normalmente no fim da fila.

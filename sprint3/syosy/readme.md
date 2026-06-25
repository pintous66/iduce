# 5. Especificação e Configuração do Sistema de Comunicações (SYOSY)

De forma a converter o desenho concetual numa arquitetura analisável e rigorosa, esta secção detalha as configurações exatas do middleware DDS, as interfaces de rede V2V e os protocolos intra-veículo (CAN). Todas as políticas de resiliência e restrições temporais apresentadas são tecnicamente mensuráveis e implementáveis.

---

## 5.1 Especificação DDS (Comunicação Inter-veículo Crítica)

Para o caminho crítico V2V, o sistema utiliza o **Data Distribution Service (DDS)** sobre **5G PC5 (Sidelink)**.

### 5.1.1 Definição de Tipos em IDL (Interface Definition Language)

Para evitar o overhead do JSON e garantir tempos de parsing previsíveis, os dados são estruturados através de IDL e serializados em CDR (*Common Data Representation*).

```idl
struct PlatoonEmergencyAlert {
    string<16> vehicle_id;           // Identificador do nó
    unsigned long long timestamp_ms; // Prevenção Anti-Replay
    unsigned long sequence_number;   // Prevenção Anti-Replay
    unsigned short alert_code;       // Ex: 0x01 = BRAKE_FAULT
};

struct PlatoonManeuverCmd {
    unsigned long long timestamp_ms;
    unsigned short command_type;     // Ex: 0x02 = EMERGENCY_STOP
    float target_deceleration;       // m/s^2
};
```

### 5.1.2 Topologia de Tópicos e Atores

#### Tópico: `Platoon_Emergency_Alert`

| Papel      | Entidade                            |
| ---------- | ----------------------------------- |
| Publisher  | ECU PredMaint (Veículos Seguidores) |
| Subscriber | ECU PlatMgmt (Veículo Líder)        |

#### Tópico: `Platoon_Maneuver_Cmd`

| Papel      | Entidade                     |
| ---------- | ---------------------------- |
| Publisher  | ECU VC (Veículo Líder)       |
| Subscriber | ECU VC (Veículos Seguidores) |

### 5.1.3 Ficheiro de Configuração QoS (QoS Profile)

Os valores concretos aplicados aos tópicos críticos garantem a entrega, a vivacidade e o descarte de dados obsoletos.

| Política QoS      | Valor Configurado           | Justificação Técnica                                                                                              |
| ----------------- | --------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Reliability       | `RELIABLE`                  | Garante a retransmissão no caso de perda de pacotes rádio. Essencial para comandos de travagem.                   |
| Deadline          | `50 ms`                     | Se o VC não receber uma atualização do líder ou de um seguidor crítico em 50 ms, um callback de falha é acionado. |
| Liveliness        | `AUTOMATIC (Lease: 100 ms)` | Monitoriza a presença física na rede. O sistema considera o nó inativo se não houver prova de vida em 100 ms.     |
| History           | `KEEP_LAST (Depth: 1)`      | O VC apenas necessita da leitura ou comando mais recente. Filas longas criariam reações desatualizadas.           |
| Durability        | `VOLATILE`                  | Veículos que entrem na rede tardiamente não precisam (nem devem) receber comandos de emergência antigos.          |
| TransportPriority | `HIGHEST (1)`               | Garante primazia no mapeamento para as filas do hardware de rádio.                                                |

---

## 5.2 Adaptador DDS-PC5 e Mapeamento de Prioridades

O 5G PC5 não compreende nativamente as prioridades do DDS. É necessário um adaptador na camada de rede para mapear o `TransportPriority` do DDS para o parâmetro **PPPP (ProSe Per-Packet Priority)** do 5G Sidelink.

### Interface e Filas

O adaptador mantém duas filas de transmissão (*TX Queues*):

* **Fila Urgente:** Mensagens de controlo (`PPPP = 1`)
* **Fila Normal:** Mensagens operacionais ou de telemetria base (`PPPP = 5`)

### Regra de Mapeamento

O adaptador inspeciona o cabeçalho do pacote RTPS (protocolo subjacente ao DDS).

Se:

```text
TransportPriority == HIGHEST
```

o pacote é encapsulado numa trama MAC do 5G com:

```text
PPPP = 1
```

obtendo acesso imediato aos *Resource Blocks* de rádio (*preempting* outros pacotes).

---

## 5.3 Pseudocódigo de Segurança e Lógica de Controlo V2V

O processamento das mensagens subscritas no módulo VC inclui validações rigorosas de segurança temporal e gestão de falhas de comunicação.

```cpp
void on_data_available(DataReader* reader) {
    PlatoonManeuverCmd cmd;
    SampleInfo info;
    reader->take_next_sample(&cmd, &info);

    if (info.valid_data) {
        uint64_t current_time = get_system_time_ms();

        // 1. Anti-Replay e Proteção de Frescura (Freshness)
        if (current_time - cmd.timestamp_ms > 20) {
            log_error("Aviso: Mensagem antiga detetada. Possível Replay Attack.");
            return; // Descarta pacotes com atraso superior a 20 ms
        }

        // 2. Aceitação do comando e atuação
        execute_maneuver(cmd.target_deceleration);
    }
}

// 3. Callbacks de Liveliness e Deadline
void on_liveliness_changed() {
    log_critical("Falha de Liveliness: Nó perdeu comunicação V2V!");
    transition_to_safe_mode();
}

void on_requested_deadline_missed() {
    log_critical("Falha de Deadline: Janela de 50 ms excedida.");
    transition_to_safe_mode();
}

void transition_to_safe_mode() {
    // Modo degradado: desativa platoon, aciona travagem gradual local
    apply_local_braking(2.0); // 2 m/s^2
}
```

---

## 5.4 Especificação CAN Bus (Comunicação Intra-veículo)

Para o comando final dos atuadores (**VC → Atuadores**), o barramento CAN (1 Mbps) exige a maximização dos seus 8 bytes de payload.

### Configuração da Mensagem CAN

| Parâmetro              | Valor                                    |
| ---------------------- | ---------------------------------------- |
| ID da Mensagem         | `0x105` (ID standard de alta prioridade) |
| Período de Transmissão | `10 ms`                                  |
| Frequência             | `100 Hz`                                 |

### Layout de Bits (64 bits / 8 bytes)

| Byte(s) | Campo                                                      |
| ------- | ---------------------------------------------------------- |
| 0–1     | Ângulo de Direção (`Int16`)                                |
| 2–3     | Pressão de Travagem (`UInt16`)                             |
| 4       | Status de Comando (`0x01=Normal`, `0x02=Emergência`)       |
| 5       | Alive Counter (`0–255`, incrementado pelo VC a cada ciclo) |
| 6–7     | CRC-16 (calculado sobre os bytes 0 a 5)                    |

### Lógica do Watchdog (ECU do Atuador)

O microcontrolador do atuador garante a segurança física caso o barramento CAN seja cortado ou o VC bloqueie.

```cpp
void can_rx_interrupt(CAN_Frame frame) {
    if (crc16(frame.data, 6) != frame.crc) {
        return; // Falha de integridade: descarta trama corrompida
    }

    if (frame.alive_counter == last_alive_counter) {
        stagnation_count++;
    } else {
        stagnation_count = 0;
        last_alive_counter = frame.alive_counter;
    }
}

// Executado num timer de hardware a cada 10 ms
void hardware_watchdog_timer() {
    // Se o contador estagnar por 3 ciclos (30 ms)
    if (stagnation_count >= 3) {
        engage_hardware_fail_safe(); // Desliga controlo central e trava autonomamente
    }
}
```

---

## 5.5 Revisão da Análise de Latência End-to-End

Respondendo à necessidade de traçar o fluxo completo de uma avaria, a tabela seguinte apresenta o percurso desde a deteção da falha no seguidor até à atuação mecânica em todos os veículos do *platoon*.

O Controlo do Veículo (VC) necessita de agir em intervalos de **0,1 s a 5 s**.

| Etapa do Fluxo Crítico        | Processo Físico / Lógico                                                                       | Tempo Estimado  |
| ----------------------------- | ---------------------------------------------------------------------------------------------- | --------------- |
| 1. Amostragem / Deteção       | ECU PredMaint do seguidor deteta anomalia (ex: pressão travões) e publica no DDS               | 2 ms            |
| 2. Fila Adaptador (Seguidor)  | Serialização CDR e colocação na fila PPPP=1 do adaptador PC5                                   | 1 ms            |
| 3. Transmissão V2V (1)        | Rádio 5G Sidelink transporta o alerta do Seguidor para o Líder                                 | 3 ms            |
| 4. Processamento do Líder     | ECU PlatMgmt recebe e informa o VC; VC calcula a manobra de paragem e publica o comando no DDS | 4 ms            |
| 5. Fila Adaptador (Líder)     | Colocação do comando de paragem na fila PPPP=1                                                 | 1 ms            |
| 6. Transmissão V2V (2)        | Rádio 5G Sidelink difunde o comando do Líder para todos os Seguidores                          | 3 ms            |
| 7. Reação do Seguidor         | ECU VC dos seguidores recebe, valida Anti-Replay e processa                                    | 2 ms            |
| 8. Transmissão CAN Bus        | VC envia trama de 8 bytes para a ECU de travagem                                               | < 1 ms (134 µs) |
| 9. Processamento Atuador      | ECU de travagem valida CRC e converte em PWM para os travões                                   | 2 ms            |
| **Latência End-to-End Total** | Desde a avaria até à resposta mecânica coordenada do pelotão                                   | **~18 ms**      |

### Conclusão da Análise Temporal

Mesmo considerando o circuito completo com dupla passagem no ar (duas transmissões V2V), filas de espera do adaptador de rádio e os atrasos computacionais das ECUs, a latência total estimada fixa-se em aproximadamente **18 ms**.

Este valor cumpre de forma extremamente folgada e provável o *deadline* de **100 ms** deduzido dos requisitos de operação de curto prazo (**0,1 s**), permitindo ao sistema reagir a emergências de forma determinística.

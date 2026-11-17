# Catálogo de métricas

Fonte da verdade para as **chaves dos itens**. Coletores, template do Zabbix e dashboard do Grafana
devem usar exatamente estas chaves. Mudou aqui → mude nos três.

Legenda de viabilidade (sem root): ✅ esperado funcionar · ⚠️ depende do aparelho/versão · ❌ bloqueado.
A coluna **Moto G54** é preenchida na Fase 2, testando no aparelho.

## Itens

| Chave | Tipo | Unid. | Fonte | Viável | Moto G54 |
|---|---|---|---|---|---|
| `agent.heartbeat` | Unsigned | — | timestamp de cada ciclo | ✅ | |
| `agent.version` | Text | — | versão do agente | ✅ | |
| `agent.collect.duration` | Float | s | tempo do ciclo de coleta | ✅ | |
| `agent.collect.errors` | Unsigned | — | coletores que falharam no ciclo | ✅ | |
| `android.battery.level` | Unsigned | % | `termux-battery-status` → `percentage` | ✅ | |
| `android.battery.temperature` | Float | °C | `termux-battery-status` → `temperature` | ✅ | |
| `android.battery.status` | Text | — | `CHARGING`, `DISCHARGING`, `FULL`… | ✅ | |
| `android.battery.plugged` | Text | — | `UNPLUGGED`, `PLUGGED_AC`, `PLUGGED_USB`… | ✅ | |
| `android.battery.health` | Text | — | `GOOD`, `OVERHEAT`… | ✅ | |
| `android.battery.current` | Float | mA | `termux-battery-status` → `current` (µA na maioria) | ⚠️ | |
| `android.storage.total` | Unsigned | B | `os.statvfs` na partição `/data` | ✅ | |
| `android.storage.free` | Unsigned | B | `os.statvfs` na partição `/data` | ✅ | |
| `android.memory.total` | Unsigned | B | `/proc/meminfo` → `MemTotal` | ⚠️ | |
| `android.memory.available` | Unsigned | B | `/proc/meminfo` → `MemAvailable` | ⚠️ | |
| `android.uptime` | Unsigned | s | `/proc/uptime` | ⚠️ | |
| `android.wifi.rssi` | Integer (Numeric signed) | dBm | `termux-wifi-connectioninfo` → `rssi` | ✅ | |
| `android.wifi.link_speed` | Unsigned | Mbps | `termux-wifi-connectioninfo` → `link_speed_mbps` | ✅ | |
| `android.wifi.ip` | Text | — | `termux-wifi-connectioninfo` → `ip` | ✅ | |
| `android.server.latency` | Float | ms | tempo de conexão TCP até o Zabbix :10051 | ✅ | |
| `android.cpu.temperature` | Float | °C | `/sys/class/thermal/thermal_zone*/temp` | ⚠️ | |
| `android.cpu.freq` | Unsigned | Hz | `/sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq` | ⚠️ | |
| `android.cpu.util` | Float | % | `/proc/stat` | ❌ | |
| `android.net.rx/tx` | Unsigned | B | `/proc/net/dev` | ❌ | |

Observações:
- `android.wifi.ssid` foi deixado de fora de propósito: exige permissão de localização e é dado
  pessoal. Adicionar só se o dono do aparelho concordar.
- Itens de texto que mudam pouco (status, health, IP) podem usar *Discard unchanged with heartbeat*
  no pré-processamento do Zabbix para não encher o histórico.
- Unidades: armazene bytes/segundos/°C "crus" e deixe Zabbix/Grafana formatarem (`B`, `s`, `%`).

## Triggers do template

| Nome | Expressão (ideia) | Severidade |
|---|---|---|
| Agente sem dados | `nodata(/<template>/agent.heartbeat,10m)=1` | High |
| Bateria baixa | `last(/<template>/android.battery.level)<15 and last(/<template>/android.battery.plugged)="UNPLUGGED"` | Warning |
| Bateria quente | `min(/<template>/android.battery.temperature,5m)>42` | Average |
| Armazenamento quase cheio | `last(/<template>/android.storage.free)/last(/<template>/android.storage.total)<0.1` | Warning |
| Agente com erros de coleta | `min(/<template>/agent.collect.errors,15m)>0` | Info |

Os limites são pontos de partida; ajuste em *macros* do template (ex.: `{$BATTERY.LOW}`) para poder
mudar por aparelho sem editar o template.

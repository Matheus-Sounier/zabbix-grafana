# Celular: preparar o Android

## 1. Apps

Instale **os três da mesma fonte** — F-Droid (<https://f-droid.org/packages/com.termux/>) ou
releases do GitHub. Não misture com a versão da Play Store: as assinaturas são diferentes e o
Termux:API para de responder (comandos `termux-*` ficam travados).

- **Termux**
- **Termux:API** — dá acesso a bateria, Wi-Fi, telefonia etc.
- **Termux:Boot** — roda scripts quando o celular liga. Abra o app uma vez depois de instalar.

## 2. Pacotes no Termux

```bash
pkg update && pkg upgrade
pkg install python git termux-api termux-services
```

Depois de instalar `termux-services`, feche e abra o Termux (o runit só sobe numa sessão nova).

## 3. Testes manuais (Fase 2)

```bash
termux-battery-status            # JSON com percentage, temperature, status, health…
termux-wifi-connectioninfo       # rssi, ip, link_speed (SSID só com permissão de localização)
cat /proc/meminfo                # memória
df -h /data                      # armazenamento interno
cat /proc/stat                   # esperado: Permission denied (sem root)
nc -zv <ip-do-debian> 10051      # o celular alcança o Zabbix? (pkg install netcat-openbsd)
```

Na primeira chamada de cada `termux-*`, o Android pede permissão — aceite. Anote o resultado de
cada linha em [metricas.md](metricas.md) e salve as saídas reais em `tests/fixtures/` (Fase 3).

## 4. Não deixar o Android matar o agente

Isso precisa ser feito **em cada aparelho**:

1. **Bateria**: *Configurações → Apps → Termux → Bateria →* **Sem restrições**.
   Repita para Termux:API e Termux:Boot.
2. **Wake lock**: o agente chama `termux-wake-lock` (aparece uma notificação persistente do Termux —
   é esperado).
3. **Phantom process killer** (Android 12+): o sistema mata processos-filhos do Termux acima de um
   limite. No Android 14+: *Opções do desenvolvedor →* **Desativar restrições de processos filhos**.
   Em versões anteriores, via ADB a partir do PC:
   ```bash
   adb shell "settings put global settings_enable_monitor_phantom_procs false"
   ```
4. **Motorola** costuma respeitar as configurações acima. Outras marcas (Samsung, Xiaomi) têm
   gerenciadores próprios — consulte <https://dontkillmyapp.com> para o modelo.

## 5. Rede

- O celular precisa estar na **mesma rede** do Debian (ou na mesma VPN).
- Se o roteador isola clientes Wi-Fi ("AP isolation" / rede de convidados), o celular não alcança o
  servidor mesmo estando "na mesma rede".
- Para monitorar fora de casa: Tailscale no Debian e no celular, e o agente aponta para o IP
  `100.x.y.z` do Debian. Nunca abra a 10051 no roteador.

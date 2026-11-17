# Roadmap

Cada fase termina com algo funcionando e verificável. Não pule para a próxima sem fechar a anterior —
a maioria dos problemas desse projeto é de rede/Android, não de código.

## Fase 0 — Documentação ✅
- [x] Estrutura da pasta, README, CLAUDE.md
- [x] Arquitetura, guias de servidor e celular, catálogo de métricas

## Fase 1 — Servidor (Debian 13)
- [ ] Instalar Zabbix Server + frontend + banco ([docs/servidor.md](docs/servidor.md))
- [ ] Instalar Zabbix Agent 2 no próprio Debian (monitorar o servidor também é aprendizado)
- [ ] Instalar Grafana + plugin Zabbix, configurar data source
- [ ] Criar host de teste `ANDROID-TESTE` com um item trapper e enviar valor com `zabbix_sender`
- [ ] **Pronto quando:** o valor enviado manualmente aparece em *Monitoring → Latest data* e num painel do Grafana

## Fase 2 — Exploração no celular (Moto G54)
- [ ] Instalar Termux + Termux:API + Termux:Boot (mesma fonte — ver [docs/celular.md](docs/celular.md))
- [ ] Testar à mão cada fonte de métrica e preencher a coluna "Moto G54" em [docs/metricas.md](docs/metricas.md)
- [ ] Confirmar que o celular alcança o servidor na porta 10051 (`nc -zv <ip> 10051`)
- [ ] **Pronto quando:** a matriz de métricas está validada no aparelho real

## Fase 3 — Agente mínimo
- [ ] `agent/` com coletor de bateria + sender + `config/agent.json`
- [ ] Execução manual: `python -m agent --once` envia uma coleta
- [ ] Testes dos parsers rodando no PC (sem celular)
- [ ] **Pronto quando:** a bateria do Moto G54 aparece no Zabbix

## Fase 4 — Template, triggers e dashboard
- [ ] Template `Android Termux by Zabbix trapper` com todos os itens do catálogo
- [ ] Triggers: bateria baixa, temperatura alta, armazenamento cheio, **agente sem dados (`nodata`)**
- [ ] Dashboard Grafana com variável `$host` (um dashboard para todos os celulares)
- [ ] Exportar template (YAML) e dashboard (JSON) para `server/`

## Fase 5 — Execução contínua
- [ ] Loop de coleta com intervalo configurável + log rotativo
- [ ] Serviço `termux-services` (runit reinicia se cair) + `termux-wake-lock`
- [ ] Script em `~/.termux/boot/` para subir após reboot
- [ ] Exceções de bateria no Android / phantom process killer
- [ ] Fila offline: se o servidor estiver fora, guardar com timestamp e reenviar depois
- [ ] **Pronto quando:** 24 h sem buracos no gráfico, inclusive após reiniciar o celular

## Fase 6 — Multi-dispositivo
- [ ] `install.sh`: instala pacotes, clona o repo, gera `agent.json`, ativa o serviço
- [ ] Cadastro automático do host via API do Zabbix (token) — dispensa criar host à mão
- [ ] Adicionar o segundo celular só seguindo [docs/novo-dispositivo.md](docs/novo-dispositivo.md)

## Fase 7 — Opcional
- [ ] Acesso fora de casa via Tailscale/WireGuard
- [ ] Criptografia PSK entre agente e Zabbix
- [ ] Alertas no Telegram via media type do Zabbix

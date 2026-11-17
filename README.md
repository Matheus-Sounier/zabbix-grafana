# train — Laboratório de monitoramento Android com Zabbix + Grafana

Projeto de estudo para aprender **Zabbix** e **Grafana** monitorando celulares Android de verdade.

Cada celular roda um pequeno agente em Python dentro do **Termux**, que coleta métricas (bateria,
armazenamento, memória, Wi-Fi…) e as envia para um **Zabbix Server** num Debian 13. O **Grafana**
lê os dados do Zabbix e mostra tudo num dashboard único, com um seletor de dispositivo.

```
 ┌──────────────── Celular (Termux) ────────────────┐          ┌──────────── Debian 13 ────────────┐
 │ termux-services (runit) → agente Python           │  TCP     │ Zabbix Server (trapper :10051)     │
 │   ├─ coletores: bateria, storage, memória, wifi…  │ ───────▶ │   └─ Host por celular + Template   │
 │   └─ sender  ─────────────────────────────────────┼──────────┤ Banco (PostgreSQL/MariaDB)          │
 │ Termux:Boot inicia após reboot · wake-lock        │          │ Grafana :3000 ◀── plugin Zabbix    │
 └───────────────────────────────────────────────────┘          └────────────────────────────────────┘
```

Primeiro dispositivo: **Moto G54** (host `ANDROID-MOTO-G54`). O projeto já nasce pensando em vários
celulares: todos usam o mesmo código e o mesmo template; só muda o arquivo de configuração.

## Status

📄 **Fase 0 — Documentação.** Ainda não há código. Veja o [ROADMAP](ROADMAP.md).

## Documentação

| Documento | Conteúdo |
|---|---|
| [docs/arquitetura.md](docs/arquitetura.md) | Análise da ideia, como os dados fluem, decisões e limitações |
| [docs/servidor.md](docs/servidor.md) | Instalar Zabbix + Grafana no Debian 13 e testar sem celular |
| [docs/celular.md](docs/celular.md) | Preparar o Android: Termux, permissões, bateria, segundo plano |
| [docs/metricas.md](docs/metricas.md) | Catálogo de métricas (chaves Zabbix) e o que é viável sem root |
| [docs/novo-dispositivo.md](docs/novo-dispositivo.md) | Checklist para adicionar mais um celular |
| [ROADMAP.md](ROADMAP.md) | Fases do projeto, do servidor ao multi-dispositivo |

## Avisos importantes

- O monitoramento só funciona enquanto **o celular consegue rodar o agente** e **alcançar o servidor**.
  Servidor desligado, celular fora da rede ou Android matando o processo = sem dados (e o Zabbix
  deve alertar isso — ver trigger `nodata` em [docs/metricas.md](docs/metricas.md)).
- Celulares de outras pessoas só com **autorização do dono**. Métricas como nome da rede Wi-Fi e
  dados de operadora são informação pessoal.
- Não exponha a porta 10051 do Zabbix na internet. Para acesso fora de casa, use VPN (ex.: Tailscale).

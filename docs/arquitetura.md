# Arquitetura e análise da ideia

## Veredito

A ideia é boa e o plano original está no caminho certo: um host por celular, um template
reutilizável, um dashboard compartilhado e um agente Python igual em todos os aparelhos. É um
ótimo laboratório porque passa pelos conceitos reais de monitoramento (hosts, templates, triggers,
disponibilidade, coleta distribuída).

Os pontos que o plano original **subestimava** e que este projeto trata explicitamente:

1. **Nem toda métrica é acessível sem root.** Desde o Android 8, apps comuns não leem `/proc/stat`
   (uso de CPU) e, desde o Android 10, não leem `/proc/net/dev` (tráfego de rede). Ou seja,
   `cpu.py` e `network.py` não são triviais como parecem. Bateria, armazenamento e Wi-Fi funcionam
   bem. Ver a matriz em [metricas.md](metricas.md) — ela é validada no aparelho na Fase 2.
2. **"Cadastrar host" não precisa ser manual.** Com um token da API do Zabbix, o `install.sh` pode
   criar o host e vincular o template sozinho (Fase 6).
3. **O Zabbix precisa saber quando o celular sumiu.** Itens trapper não têm "disponibilidade" como
   um agente. A solução é uma trigger `nodata()` sobre um item de heartbeat.
4. **Rede é o ponto que mais quebra.** O celular precisa chegar no IP do Debian na porta 10051. Se o
   Debian for VM com NAT, não chega — precisa de rede em modo *bridge*. Fora de casa, só com VPN.
5. **Android mata processos em segundo plano.** Otimização de bateria, *phantom process killer*
   (Android 12+) e Doze. Exige configuração manual em cada aparelho — ver [celular.md](celular.md).
6. **Fontes do Termux.** Termux, Termux:API e Termux:Boot têm que vir da mesma loja (F-Droid ou
   GitHub), senão as assinaturas não batem e o Termux:API não responde.

MDM (gerenciamento de dispositivos móveis) é exagero para um laboratório pessoal; fica fora do escopo.

## Fluxo dos dados

```
coletor (lê fonte) → parser (normaliza) → lista de {host, key, value, clock}
      → sender (protocolo Zabbix, JSON "sender data") → Zabbix Server :10051
      → item trapper no host com o MESMO nome técnico → histórico/trends no banco
      → triggers avaliam → Grafana consulta via API do Zabbix (plugin)
```

## Decisões

| Decisão | Escolha | Por quê |
|---|---|---|
| Como enviar dados | Itens **Zabbix trapper** (push) | Não existe Zabbix Agent oficial para Android; o celular não tem IP fixo e fica atrás de NAT, então o servidor não consegue fazer *pull*. |
| Biblioteca de envio | [`zabbix_utils`](https://github.com/zabbix/python-zabbix-utils) (oficial, Python puro) | Implementa o protocolo do sender e a API. O `sender.py` do projeto é um invólucro fino, então dá para trocar por implementação própria do protocolo (bom exercício) sem mexer nos coletores. |
| Onde roda | Python no Termux | Sem root, sem compilar APK, acesso às APIs Android via Termux:API. |
| Como fica rodando | `termux-services` (runit) + `termux-wake-lock` + Termux:Boot | runit reinicia o agente se ele cair; Boot sobe tudo após reiniciar o celular. `termux-job-scheduler` foi descartado porque o intervalo mínimo é 15 min. |
| Configuração por aparelho | `config/agent.json` (fora do git) gerado a partir de `agent.example.json` | Mesmo código em todos os celulares; só muda nome do host e servidor. |
| Visualização | Grafana + plugin Zabbix (`alexanderzobnin-zabbix-app`) | Um dashboard com variável `$host` atende todos os celulares. |

## Invariantes que atravessam o projeto

- O **nome técnico do host** no Zabbix (ex.: `ANDROID-MOTO-G54`) tem que ser **idêntico**
  (maiúsculas inclusive) ao campo `host` do `agent.json`. Se não bater, o Zabbix descarta o valor
  em silêncio ("processed: 0; failed: N").
- As **chaves dos itens** (ex.: `android.battery.level`) existem em três lugares: coletores, template
  exportado e dashboard do Grafana. O catálogo em [metricas.md](metricas.md) é a fonte da verdade.
- O tipo do valor enviado precisa casar com o *Type of information* do item (numérico inteiro,
  float, texto). Enviar `"78%"` para um item numérico falha.

## Estrutura planejada

```
train/
├── docs/                    # guias e decisões (este arquivo)
├── agent/                   # roda no celular (Termux)
│   ├── collectors/          # um módulo por fonte: battery, storage, memory, wifi, system
│   ├── sender.py            # envio ao Zabbix (+ fila offline na Fase 5)
│   └── __main__.py          # loop de coleta, `--once`, `--dry-run`
├── config/agent.example.json
├── service/                 # run script do termux-services e script do Termux:Boot
├── install.sh               # instalação num celular novo
├── server/
│   ├── zabbix/              # template exportado (YAML)
│   └── grafana/             # dashboard exportado (JSON)
└── tests/                   # testes dos parsers com saídas reais salvas (fixtures)
```

Cada coletor separa **ler** (chama `termux-*` ou lê `/proc`, `/sys`) de **interpretar** (função
pura que recebe texto/JSON e devolve métricas). Assim os parsers são testados no PC usando saídas
reais copiadas do celular para `tests/fixtures/`.

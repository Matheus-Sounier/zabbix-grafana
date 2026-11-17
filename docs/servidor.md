# Servidor: Zabbix + Grafana no Debian 13

> Os comandos exatos de repositório mudam a cada versão. Use sempre o assistente oficial em
> <https://www.zabbix.com/download> (escolha: versão **LTS** mais recente, Debian 13, Server +
> Frontend + Agent 2, banco e servidor web de sua preferência). Este guia diz **o que** instalar e
> **o que conferir**.

## 0. Onde o Debian está rodando?

| Cenário | O que observar |
|---|---|
| Máquina física / notebook | IP fixo na rede (reserva DHCP no roteador) para o celular sempre achar o servidor. |
| VM (VirtualBox, VMware, Hyper-V) | Rede em modo **bridge**, não NAT. Com NAT o celular não alcança a porta 10051. |
| WSL | Não recomendado: rede em NAT por padrão e sem systemd confiável para serviços longos. |

## 1. Zabbix

Pacotes (via repositório oficial do Zabbix):

- `zabbix-server-pgsql` **ou** `zabbix-server-mysql` — o servidor
- `zabbix-frontend-php` + `zabbix-nginx-conf` **ou** `zabbix-apache-conf` — interface web
- `zabbix-sql-scripts` — schema inicial do banco
- `zabbix-agent2` — monitorar o próprio Debian
- `zabbix-sender` — **para testar itens trapper sem celular**

Portas:

| Porta | Uso | Quem acessa |
|---|---|---|
| 80/443 (ou 8080) | Frontend web | Seu PC |
| 10051/TCP | Trapper (recebe dados dos celulares) | Celulares |
| 3000/TCP | Grafana | Seu PC |

Se houver firewall (`nftables`/`ufw`), libere 10051 **só para a rede local** (ou faixa da VPN).

## 2. Teste sem celular (fim da Fase 1)

1. No frontend: *Data collection → Hosts → Create host*
   - Host name: `ANDROID-TESTE` · Host group: `Android`
   - Não precisa de interface (trapper não usa).
2. Criar item no host:
   - Type: **Zabbix trapper** · Key: `android.battery.level` · Type of information: *Numeric (unsigned)*
   - *Allowed hosts*: deixe vazio no teste; depois restrinja à rede local (ex.: `192.168.0.0/24`).
3. No Debian:
   ```bash
   zabbix_sender -z 127.0.0.1 -s "ANDROID-TESTE" -k android.battery.level -o 77 -vv
   ```
   Esperado: `processed: 1; failed: 0`. Se `failed: 1`, o nome do host, a chave ou o tipo não batem.
4. Ver em *Monitoring → Latest data*.

Repita o teste a partir de **outro computador da rede** apontando para o IP do Debian — isso valida
o caminho de rede que o celular vai usar.

## 3. Grafana

1. Instalar pelo repositório oficial: <https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/>
2. Plugin do Zabbix:
   ```bash
   sudo grafana-cli plugins install alexanderzobnin-zabbix-app
   sudo systemctl restart grafana-server
   ```
3. Em *Administration → Plugins*, habilitar **Zabbix**.
4. No Zabbix, criar um usuário só-leitura para o Grafana e gerar um **API token** para ele
   (*Users → API tokens*).
5. No Grafana, data source Zabbix:
   - URL: `http://<ip-do-debian>/zabbix/api_jsonrpc.php` (ou sem `/zabbix` se o frontend estiver na raiz)
   - Auth: API token
6. Criar painel com o item `android.battery.level` do host `ANDROID-TESTE`.

## 4. Token para cadastro automático (Fase 6)

Para o `install.sh` criar hosts sozinho, gere um token de um usuário com permissão de escrita
**apenas** no host group `Android`. Esse token nunca vai para o git.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Contexto

Projeto de **aprendizado** de Zabbix + Grafana: celulares Android rodam um agente Python no Termux
que envia métricas (push) para itens *Zabbix trapper* num Zabbix Server em Debian 13; o Grafana lê
do Zabbix via plugin `alexanderzobnin-zabbix-app`. Primeiro aparelho: Moto G54 (`ANDROID-MOTO-G54`).

O usuário está aprendendo: explique o **porquê** das decisões de Zabbix/Grafana/Android quando
introduzir algo novo, e prefira passos que ele execute e verifique a soluções mágicas. Documentação
e comentários em **português (pt-BR)**.

**Fase atual:** ver `ROADMAP.md`. Até a Fase 3 não há código — não invente comandos de build/teste
que ainda não existem; ao criar o código, atualize esta seção com os comandos reais.

## Mapa

- `docs/arquitetura.md` — decisões e invariantes (ler antes de mudar estrutura)
- `docs/metricas.md` — **fonte da verdade das chaves de itens** e da viabilidade sem root
- `docs/servidor.md`, `docs/celular.md`, `docs/novo-dispositivo.md` — guias operacionais
- Estrutura de código planejada (`agent/`, `config/`, `service/`, `server/`, `tests/`) está em
  `docs/arquitetura.md#estrutura-planejada`

## Invariantes (quebram em silêncio se violados)

- **Chaves de item** existem em três lugares: coletores em `agent/`, template exportado em
  `server/zabbix/`, dashboard em `server/grafana/`. Ao adicionar/renomear uma métrica, atualize
  `docs/metricas.md` e os três.
- O campo `host` do `config/agent.json` precisa ser **idêntico** ao nome técnico do host no Zabbix;
  divergência resulta em `processed: 0; failed: N` sem outro erro.
- O tipo Python do valor enviado tem que casar com o *Type of information* do item (unsigned,
  float, text).
- `config/agent.json` é por aparelho e fica fora do git; o versionado é `config/agent.example.json`.
  Tokens da API do Zabbix nunca entram no repositório.

## Restrições do ambiente Android/Termux

- Sem root: `/proc/stat` (CPU) e `/proc/net/dev` (tráfego) são bloqueados; `/proc/meminfo`,
  `/proc/uptime` e `/sys/class/thermal` variam por aparelho. Não proponha métricas marcadas ❌ em
  `docs/metricas.md` sem uma fonte alternativa comprovada.
- Comandos `termux-*` podem travar indefinidamente (Termux:API ausente, de outra loja, ou permissão
  negada). Toda chamada via `subprocess` precisa de `timeout`, e a falha de um coletor não pode
  derrubar o ciclo — ela incrementa `agent.collect.errors`.
- O agente roda no Python do Termux: só stdlib + `zabbix_utils`. Evite dependências com extensões
  nativas (compilar no Termux é lento e frágil).
- Coletores separam **leitura** (I/O no aparelho) de **parsing** (função pura). Testes cobrem o
  parsing com saídas reais em `tests/fixtures/`, para rodar no PC sem celular.

## Ambiente de desenvolvimento

- A pasta está dentro do **OneDrive** (`C:\Users\mathe\OneDrive\Documentos\train`), acessada pelo
  WSL (Ubuntu) em `/mnt/c/...`. Evite criar `.venv` aqui (sincronização do OneDrive); use um venv
  fora da pasta, ex.: `~/.venvs/train`.
- O servidor Debian 13 não é este WSL (que é Ubuntu). Para testar o envio sem
  celular, use `zabbix_sender` no Debian (ver `docs/servidor.md#2-teste-sem-celular-fim-da-fase-1`).

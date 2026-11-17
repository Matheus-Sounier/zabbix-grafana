# Adicionar um novo celular

Pré-requisitos: servidor da Fase 1 funcionando e template da Fase 4 importado.
Até a Fase 6 existir, os passos marcados com 🔧 são manuais; depois, o `install.sh` faz.

## Uma vez por dispositivo

- [ ] **Autorização do dono** do aparelho (se não for seu).
- [ ] Escolher o **nome técnico do host**: `ANDROID-<MARCA>-<MODELO>` (ex.: `ANDROID-SAMSUNG-A54`).
      Se houver dois do mesmo modelo, acrescente sufixo: `ANDROID-MOTO-G54-02`.
      No Zabbix, use o *Visible name* para o nome amigável ("Celular da Ana").
- [ ] Instalar Termux, Termux:API e Termux:Boot da **mesma fonte** ([celular.md](celular.md#1-apps)).
- [ ] Pacotes no Termux ([celular.md](celular.md#2-pacotes-no-termux)).
- [ ] Exceções de bateria e phantom process killer ([celular.md](celular.md#4-não-deixar-o-android-matar-o-agente)).
- [ ] Testar alcance: `nc -zv <ip-do-debian> 10051`.
- [ ] 🔧 Criar o host no Zabbix: nome técnico acima, grupo `Android`, template vinculado.
- [ ] 🔧 Clonar o projeto no Termux e criar `config/agent.json` a partir do exemplo.
- [ ] 🔧 Rodar uma coleta manual (`--once`) e conferir em *Latest data*.
- [ ] 🔧 Ativar o serviço e o script de boot.
- [ ] Reiniciar o celular e confirmar que os dados voltam sozinhos.

## Não precisa refazer

- Itens, triggers, macros → vêm do **template**.
- Dashboard → o Grafana já lista o novo host na variável `$host`.

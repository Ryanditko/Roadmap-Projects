# Script de Deploy Simples

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Escreva um script que pega uma nova versão da sua app e a coloca em um servidor — de forma confiável, repetível e sem um checklist que você precisa lembrar à meia-noite. O script conecta ao host, posiciona o novo código, reinicia o serviço e verifica que ele realmente voltou. A parte interessante não é o caminho feliz, mas as garantias ao redor dele: faça um backup antes de tocar em qualquer coisa, verifique a saúde depois e faça rollback para a versão anterior se a nova se recusar a subir. Ao terminar, fazer deploy deve ser um comando que ou tem sucesso limpo ou deixa o servidor exatamente como estava.

## Pré-requisitos

- Uma app que roda em um host Linux remoto (ou virtual) que você acessa via SSH
- Acesso por chave SSH a esse host
- Um gerenciador de serviços para (re)iniciar a app (systemd, Docker ou um gerenciador de processos)
- Scripting shell básico (variáveis, condicionais, códigos de saída)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Automatizar um deploy remoto via SSH a partir de um único comando
- Tirar um snapshot da release atual para poder restaurá-la
- Reiniciar um serviço e confirmar que está saudável antes de declarar sucesso
- Fazer rollback automático quando um deploy falha sua verificação de saúde
- Tornar o script idempotente e seguro para reexecutar

## Requisitos Funcionais

1. O script deve conectar a um host de destino e transferir a nova release de forma não-interativa.
2. Ele deve fazer backup da release atual antes de substituí-la.
3. Ele deve reiniciar o serviço após posicionar o novo código.
4. Ele deve rodar uma verificação de saúde e tratar uma verificação falha como deploy falho.
5. Em uma verificação de saúde falha, ele deve fazer rollback para a release em backup e reiniciar.
6. Ele deve sair com código diferente de zero em qualquer falha, para que um chamador (ou CI) possa detectá-la.
7. Ele deve logar cada passo com um timestamp para diagnóstico posterior.

## Marcos Sugeridos

1. **Marco 1 — Enviar e reiniciar:** Copie a release para o host e reinicie o serviço via SSH.
2. **Marco 2 — Backup e verificação:** Tire um snapshot da release atual primeiro, depois verifique a saúde após o restart.
3. **Marco 3 — Rollback e logging:** Restaure a release anterior em caso de falha e logue cada passo com códigos de saída.

## Esboço de Dados e Interface

```text
Uso
  deploy.sh <ambiente> <ref-da-release>

Config (variáveis de ambiente, não hardcoded)
  DEPLOY_HOST      user@host
  DEPLOY_PATH      /srv/app
  HEALTH_URL       http://localhost:8080/health
  SERVICE_NAME     app.service

Passos (estrutura, não o script completo)
  1. resolve config + valida entradas
  2. ssh: snapshot da release atual -> releases/<timestamp>
  3. transfere nova release -> DEPLOY_PATH
  4. ssh: reinicia SERVICE_NAME
  5. faz polling em HEALTH_URL (N tentativas, backoff)
     ok      -> loga sucesso, sai 0
     falha   -> restaura snapshot, reinicia, loga falha, sai 1
```

## Desafios Extras

- Mantenha as últimas N releases e suporte um subcomando `rollback` para qualquer uma delas.
- Use uma estratégia de troca de symlink (`current -> releases/<ts>`) para uma virada com downtime quase zero.
- Envie uma notificação (ex.: para um webhook de chat) em sucesso e falha.
- Adicione uma flag `--dry-run` que imprime cada ação sem executá-la.

## Definição de Pronto

- [ ] Um deploy completo roda a partir de um comando sem prompts interativos.
- [ ] Um backup da release anterior existe no host após o deploy.
- [ ] Uma release deliberadamente quebrada dispara um rollback automático.
- [ ] O serviço é confirmado saudável antes de o script reportar sucesso.
- [ ] O script sai com código diferente de zero em falha e loga cada passo.

## Armadilhas Comuns

- Assumir que o restart teve sucesso sem verificar — um serviço pode "iniciar" e travar em seguida.
- Sobrescrever a release em execução antes de fazer backup, deixando nada para o rollback.
- Fixar host, caminho ou credenciais no script em vez de lê-los da config.
- Verificar a saúde uma vez sem retries, fazendo uma app de início lento ser lida como quebrada.

## Recursos

- [DigitalOcean: Como usar chaves SSH](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server) — acesso remoto não-interativo.
- [Google SRE Book: Release Engineering](https://sre.google/sre-book/release-engineering/) — os princípios por trás de deploys seguros e repetíveis.
- [Manual do rsync](https://download.samba.org/pub/rsync/rsync.1) — transferência de arquivos eficiente e retomável para releases.
- [systemd: Gerenciando serviços](https://www.freedesktop.org/software/systemd/man/systemctl.html) — iniciar, reiniciar e checar o estado do serviço.

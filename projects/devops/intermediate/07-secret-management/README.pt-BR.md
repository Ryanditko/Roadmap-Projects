# Sistema de Gerenciamento de Segredos

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Pare de espalhar senhas, chaves de API e tokens por arquivos `.env` e variáveis de CI. Construa um fluxo em torno de um gerenciador de segredos de verdade — HashiCorp Vault, AWS Secrets Manager ou equivalente — que armazena segredos criptografados, controla quem pode lê-los, mantém uma trilha de auditoria de cada acesso e os rotaciona sem downtime. Depois integre uma aplicação para que ela busque segredos em tempo de execução em vez de embuti-los. O objetivo é entender o ciclo de vida completo de um segredo: armazenar, controlar acesso, entregar, auditar, rotacionar e revogar.

## Pré-requisitos

- Um gerenciador de segredos que você possa rodar ou acessar (Vault em modo dev, Secrets Manager de nuvem)
- Uma aplicação que precise de ao menos um segredo (uma senha de banco ou chave de API)
- Entendimento de criptografia em repouso vs em trânsito e de políticas de acesso
- Familiaridade com variáveis de ambiente e como apps leem configuração

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Armazenar segredos criptografados em repouso com acesso mediado por política, não por permissões de arquivo
- Conceder acesso de leitura de menor privilégio, escopado por aplicação ou identidade
- Entregar segredos a uma app em tempo de execução sem escrevê-los em disco ou imagem
- Produzir um log de auditoria que responda "quem leu qual segredo, quando"
- Rotacionar um segredo e fazer os consumidores adotarem o novo valor sem uma indisponibilidade brusca

## Requisitos Funcionais

1. Segredos devem ser armazenados criptografados em repouso, nunca em arquivos texto ou na imagem da app.
2. O acesso deve ser governado por políticas que concedam menor privilégio por identidade.
3. Uma aplicação deve recuperar seu segredo em tempo de execução do gerenciador, não de um valor embutido.
4. Todo acesso a segredo deve ser registrado em um log de auditoria com identidade e timestamp.
5. Um segredo deve ser rotacionável, e os consumidores devem obter o novo valor sem redeploy manual quando possível.
6. Uma credencial revogada ou expirada deve parar de funcionar após a revogação.
7. Nenhum valor de segredo pode aparecer em logs da aplicação ou listagens de processos.

## Marcos Sugeridos

1. **Marco 1 — Armazenar e ler:** Coloque um segredo no gerenciador e leia-o de volta com um token escopado.
2. **Marco 2 — Integrar e auditar:** Faça uma app buscar o segredo em tempo de execução; verifique que o acesso aparece no log de auditoria.
3. **Marco 3 — Rotacionar e revogar:** Rotacione o segredo e confirme que os consumidores o adotam; revogue um token e confirme que ele falha.

## Esboço de Dados e Interface

```text
Ciclo de vida de um segredo:
  criar ─> armazenar(criptografado) ─> política(quem pode ler) ─> entregar(runtime)
     ^                                                                |
     └──────────── rotacionar / revogar <── auditoria(quem,o quê,quando)─┘

Modelo de acesso (conceitual):
  identidade (app/role) ── autentica ──> gerenciador
                          └── política: ler apenas secret/app/db-password
  app pede segredo no boot / na renovação de lease, mantém só em memória

Rotação:
  nova versão escrita -> versão antiga depreciada -> consumidores releem -> antiga revogada
```

## Desafios Extras

- Use segredos dinâmicos: faça o gerenciador gerar credenciais de banco de curta duração sob demanda.
- Adicione renovação automática de lease para que apps de longa duração nunca segurem uma credencial obsoleta.
- Integre a injeção de segredos ao Kubernetes (driver CSI ou um sidecar de init).
- Alerte quando um segredo for acessado por uma identidade inesperada ou fora de uma janela de tempo.

## Definição de Pronto

- [ ] Nenhum segredo é armazenado em texto puro no repo, na imagem ou em um `.env` commitado.
- [ ] Uma app lê seu segredo em tempo de execução e ele nunca toca o disco.
- [ ] O log de auditoria mostra cada acesso com identidade e timestamp.
- [ ] Rotacionar um segredo não exige editar código da aplicação.
- [ ] Uma credencial revogada é rejeitada em seu próximo uso.

## Armadilhas Comuns

- "Resolver" segredos movendo-os do código para um `.env` commitado — ainda texto puro no histórico do git.
- Registrar o segredo na inicialização "só para confirmar que carregou", vazando-o para o armazenamento de logs.
- Conceder uma política ampla para tudo, de modo que um único token vazado leia todos os segredos.
- Rotacionar o valor armazenado mas nunca sinalizar os consumidores, que seguem usando o antigo até quebrarem.
- Tratar o log de auditoria como opcional e depois não conseguir responder "esse segredo foi exposto?".

## Recursos

- [Documentação do HashiCorp Vault](https://developer.hashicorp.com/vault/docs) — armazenamento, políticas, segredos dinâmicos e auditoria.
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) — armazenamento e rotação gerenciados.
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) — práticas e antipadrões.
- [NIST SP 800-57: Gerenciamento de Chaves](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final) — fundamentos do manuseio de chaves e segredos.
</content>

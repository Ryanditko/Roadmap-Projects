# Pipeline de CI/CD (GitHub Actions)

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Monte um pipeline completo de integração e entrega contínua para uma aplicação pequena usando GitHub Actions. Todo push deve executar lint, testes e build do código; todo merge na branch principal deve produzir um artefato versionado e promovê-lo por staging e depois produção atrás de um portão de aprovação manual. O objetivo não é entregar um único arquivo de workflow, mas sentir como um pipeline transforma o "funciona na minha máquina" em um caminho de release repetível, controlado e auditável — incluindo a saída de emergência de reverter quando um deploy dá errado.

## Pré-requisitos

- Um repositório no GitHub com uma app que tenha suíte de testes e etapa de build
- Conforto com YAML e leitura de códigos de saída de comandos
- Entendimento básico de separação de ambientes (staging vs produção)
- Um alvo de deploy alcançável a partir de um runner (registro de contêineres, host estático ou máquina SSH)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Estruturar um workflow com múltiplos jobs e dependências entre eles
- Cachear dependências e compartilhar artefatos de build entre jobs
- Proteger deploys de produção com ambientes e revisores obrigatórios
- Injetar segredos com segurança sem vazá-los nos logs
- Disparar uma reversão para um artefato anterior sabidamente bom

## Requisitos Funcionais

1. Todo push em qualquer branch deve executar lint e a suíte completa de testes; uma falha deve falhar a execução.
2. Merges na `main` devem construir um artefato versionado e imutável, marcado com o SHA do commit.
3. O pipeline deve fazer deploy automático em staging após um build bem-sucedido.
4. Deploys de produção devem exigir aprovação manual de um revisor designado.
5. Segredos devem vir dos segredos criptografados do GitHub, nunca embutidos no código.
6. Deve existir um caminho documentado, de uma ação, para reimplantar um artefato anterior (rollback).
7. Cada etapa deve reportar status claro de sucesso/falha de volta ao pull request ou commit.

## Marcos Sugeridos

1. **Marco 1 — CI:** Lint + testes em todo push, com cache de dependências.
2. **Marco 2 — Build e artefato:** Produza um artefato marcado com o SHA e faça upload uma vez, reutilizando-o adiante.
3. **Marco 3 — Deploy e portão:** Deploy automático em staging, produção protegida por aprovação e caminho de rollback.

## Esboço de Dados e Interface

```text
Eventos de gatilho:
  push (qualquer branch)  -> lint, test
  push na main            -> lint, test, build, deploy-staging
  ambiente: produção      -> aprovação manual -> deploy-produção

Grafo de jobs:
  lint ─┐
        ├─> build ─> deploy-staging ─(aprovação)─> deploy-produção
  test ─┘

Nome do artefato:  app-<git-sha>   (imutável, nunca sobrescrito)
Segredos:          REGISTRY_TOKEN, DEPLOY_KEY   (dos segredos do repo/ambiente)
Rollback:          reexecutar o job de deploy fixado em um app-<sha> anterior
```

## Desafios Extras

- Adicione um build em matriz para testar em várias versões de runtime em paralelo.
- Poste o status do deploy e a versão do artefato em um canal do Slack ou Discord.
- Adicione um job de smoke-test que acesse um endpoint de saúde após o deploy em staging e bloqueie a promoção em caso de falha.
- Gere releases automaticamente com tags de versão semântica no merge.

## Definição de Pronto

- [ ] Um teste que falha bloqueia o merge e fica visível no pull request.
- [ ] Artefatos são marcados pelo SHA do commit e nunca alterados após o build.
- [ ] O deploy de produção não pode prosseguir sem uma aprovação humana explícita.
- [ ] Nenhum valor de segredo aparece em qualquer log de workflow.
- [ ] Reverter para o artefato anterior é uma ação documentada e repetível.

## Armadilhas Comuns

- Reconstruir o artefato separadamente para staging e produção, fazendo os dois ambientes rodarem bytes diferentes.
- Ecoar variáveis de ambiente para depuração e vazar um segredo em logs públicos.
- Pular o cache, deixando cada execução lenta o bastante para as pessoas começarem a burlar o CI.
- Tratar um pipeline verde como "implantado e saudável" sem qualquer verificação pós-deploy.
- Não ter plano de rollback, tornando um deploy ruim um hotfix desesperado sob pressão.

## Recursos

- [Documentação do GitHub Actions](https://docs.github.com/pt/actions) — workflows, jobs e gatilhos direto da fonte.
- [Usando ambientes para deploy](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) — portões de aprovação e regras de proteção.
- [Segredos criptografados no Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) — a forma segura de lidar com credenciais.
- [Martin Fowler: Integração Contínua](https://martinfowler.com/articles/continuousIntegration.html) — os princípios por trás da prática.
</content>

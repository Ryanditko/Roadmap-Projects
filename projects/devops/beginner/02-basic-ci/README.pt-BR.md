# Pipeline de CI Básico

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Monte um pipeline de integração contínua que roda sozinho toda vez que você faz push. Ele faz checkout do repositório, instala dependências, roda o linter, executa os testes e reporta sucesso ou falha — transformando "será que quebrei algo?" de uma tarefa manual em um portão automático. Esta é a primeira linha de defesa de qualquer projeto moderno: um check vermelho em um pull request barra um bug antes que outra pessoa o veja. Você vai escolher uma plataforma de CI, expressar o fluxo como estágios declarativos e entender por que um pipeline rápido e ciente de cache é a diferença entre um check em que as pessoas confiam e um que elas contornam.

## Pré-requisitos

- Um repositório com testes que você consegue rodar localmente ([Dockerizar uma Aplicação Simples](../01-dockerize-simple-app/) combina bem)
- Um host Git com CI (GitHub Actions, GitLab CI ou similar)
- Os comandos locais exatos para instalar deps, lintar e testar
- Familiaridade básica com YAML

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Expressar um fluxo de build/teste como estágios declarativos de pipeline
- Disparar um pipeline automaticamente em push e pull request
- Cachear dependências para manter as execuções rápidas
- Falhar o pipeline corretamente quando um passo de lint ou teste falha
- Trazer o status do pipeline de volta ao pull request

## Requisitos Funcionais

1. O pipeline deve rodar automaticamente em todo push e pull request para a branch principal.
2. Ele deve fazer checkout do código, instalar dependências, lintar e testar como passos distintos.
3. Um passo de lint ou teste que falha deve falhar o pipeline inteiro com saída diferente de zero.
4. A instalação de dependências deve usar cache para que dependências inalteradas não sejam rebaixadas a cada execução.
5. O resultado de sucesso/falha deve ser visível no pull request.
6. A configuração do pipeline deve viver no repositório como código versionado.
7. O pipeline deve rodar em um ambiente limpo, sem assumir nada da máquina do desenvolvedor.

## Marcos Sugeridos

1. **Marco 1 — Olá pipeline:** Adicione uma config que faz checkout do código e imprime as versões do toolchain em push.
2. **Marco 2 — Build e teste:** Adicione passos de instalação, lint e teste; faça um teste quebrado deixar a execução vermelha.
3. **Marco 3 — Velocidade e sinal:** Adicione cache de dependências e um badge de status; confirme que o status aparece nos PRs.

## Esboço de Dados e Interface

```text
Layout do repo
  .github/workflows/ci.yml   (ou .gitlab-ci.yml)
  <fonte do projeto + testes>

Estágios do pipeline (estrutura, não o YAML completo)
  trigger: on push + pull_request para main
  job "build-and-test":
    step: checkout
    step: configura runtime (versão fixada)
    step: restaura cache de dependências   (chave = hash do lockfile)
    step: instala dependências
    step: lint       -> saída != 0 falha o job
    step: test       -> saída != 0 falha o job
    step: salva cache de dependências

Superfície de status
  Check no PR: ci / build-and-test  ->  passando | falhando
  Badge no README: ![CI](https://github.com/OWNER/REPO/actions/workflows/ci.yml/badge.svg)
```

## Desafios Extras

- Rode o job de teste sobre uma matriz de versões de runtime ou sistemas operacionais.
- Divida lint e teste em jobs paralelos e observe o ganho de tempo real.
- Faça upload de relatórios de teste ou cobertura como artefatos de build.
- Exija que o check de CI passe antes que um pull request possa ser mesclado (proteção de branch).

## Definição de Pronto

- [ ] Fazer push de um commit dispara o pipeline sem nenhuma ação manual.
- [ ] Um teste deliberadamente quebrado deixa a execução vermelha e bloqueia o PR.
- [ ] Uma segunda execução com dependências inalteradas é visivelmente mais rápida graças ao cache.
- [ ] A config do pipeline está commitada no repositório.
- [ ] Um badge de status ou check de PR reflete a execução mais recente.

## Armadilhas Comuns

- Depender de ferramentas já instaladas no seu laptop que o runner limpo de CI não tem.
- Engolir o código de saída de um passo (encanar por um comando que sempre retorna 0), fazendo falhas parecerem verdes.
- Cachear por uma chave que nunca muda (deps obsoletas) ou uma que sempre muda (nunca acerta).
- Guardar segredos no arquivo do workflow em vez do cofre de segredos criptografado da plataforma.

## Recursos

- [GitHub Actions: Sobre workflows](https://docs.github.com/en/actions/using-workflows/about-workflows) — triggers, jobs e steps.
- [Documentação do GitLab CI/CD](https://docs.gitlab.com/ee/ci/) — estágios e configuração de pipeline.
- [GitHub Actions: Cache de dependências](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows) — chaves de cache bem feitas.
- [Martin Fowler: Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html) — a prática por trás da ferramenta.

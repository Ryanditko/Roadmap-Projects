# Contribuindo

> 🌐 [English](./CONTRIBUTING.md) · **Português**

Obrigado por ajudar a melhorar esta coleção de projetos de aprendizado! Este guia
explica como o repositório é organizado e como contribuir bem.

## A regra que mais importa

Toda descrição de projeto é um **exercício guiado, não uma solução pronta.**
Descreva *o que* construir e *por que* importa; esboce formatos de dados e
contratos — mas nunca cole uma implementação completa. Quem escreve o código é o
aluno. PRs que entregam soluções completas serão convidados a enxugá-las.

## Estrutura do repositório

```text
projects/
  <dominio>/                backend, frontend, data-science,
    <nivel>/                data-engineering, devops, system-design
      NN-<slug>/            beginner | intermediate | advanced
        README.md           descrição do projeto em inglês
        README.pt-BR.md     mesma descrição em português
      README.md             página da trilha (linka todos os projetos)
```

Cada domínio × nível tem 10 projetos numerados.

## Formas de contribuir

| Eu quero… | Como |
|---|---|
| Sugerir um novo projeto | Abra uma issue **💡 New project idea** e depois um PR. |
| Corrigir link, erro ou typo | Abra um PR (ou uma **📝 Content issue**). |
| Melhorar/expandir uma descrição | Abra um PR no `README.md` **e** no `README.pt-BR.md` do projeto. |
| Traduzir uma descrição | Adicione/ajuste o arquivo do idioma faltante para manter sincronia. |
| Melhorar tooling/docs | Abra uma issue **✨ Enhancement** ou um PR. |

## Adicionando um novo projeto

1. Faça o fork e crie um branch.
2. Escolha o `dominio/nivel` correto e o próximo número livre.
3. Crie a pasta `NN-<slug-em-kebab-case>/`.
4. Copie [`.github/PROJECT_TEMPLATE.md`](./.github/PROJECT_TEMPLATE.md) →
   `README.md` e [`.github/PROJECT_TEMPLATE.pt-BR.md`](./.github/PROJECT_TEMPLATE.pt-BR.md)
   → `README.pt-BR.md`, e preencha ambos.
5. Adicione uma linha na tabela da página da trilha (`README.md`) linkando o projeto.
6. Abra um PR.

### Estrutura da descrição

Toda descrição segue o template e inclui: **Visão Geral, Pré-requisitos,
Objetivos de Aprendizado, Requisitos Funcionais, Marcos Sugeridos, Esboço de Dados
e Interface, Desafios Extras, Definição de Pronto, Armadilhas Comuns, Recursos.**

### Critérios de dificuldade

- **Iniciante** (horas–dias): conhecimento básico, poucas dependências, um conceito central.
- **Intermediário** (dias–semanas): APIs/BDs/frameworks, várias funcionalidades juntas.
- **Avançado** (semanas+): arquitetura, escalabilidade, segurança, sistemas integrados.

## Diretrizes de pull request

- Mantenha cada PR focado em um **único tema**.
- Garanta que todo link relativo que você adicionar resolva (a CI verifica isso).
- Mantenha os arquivos bilíngues em sincronia — atualize **os dois** idiomas, ou explique por quê não.
- Escreva mensagens de commit claras (veja abaixo).
- Preencha o checklist do template de PR.

### Formato de mensagem de commit

Use [Conventional Commits](https://www.conventionalcommits.org/):

```text
feat(backend): add rate-limited API project to intermediate
fix(frontend): correct broken link in kanban board brief
docs: expand learning objectives for URL shortener
ci: add markdown lint workflow
```

## Código de conduta

A participação é regida pelo nosso [Código de Conduta](./CODE_OF_CONDUCT.md).

## Licença

Ao contribuir, você concorda que suas contribuições são licenciadas sob a
[Licença MIT](./LICENSE) do repositório.

---

Obrigado por ajudar a tornar este recurso melhor para todos!

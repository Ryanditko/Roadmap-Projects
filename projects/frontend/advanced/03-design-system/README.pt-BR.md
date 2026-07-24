# Design System Completo (biblioteca de componentes)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa a fundação visual compartilhada que muitos times de produto consomem — a camada por trás do Material, do Carbon e do Polaris. Um design system de verdade é mais do que uma pasta de componentes: é uma camada de tokens (cores, espaçamento, tipografia) que alimenta componentes acessíveis e tematizáveis, publicada como um pacote versionado com documentação viva. O desafio de engenharia é a governança em escala. Como evoluir um botão usado por quarenta telas sem quebrar nenhuma delas? Como pegar uma regressão visual de um pixel antes que um consumidor pegue? Este projeto trata o sistema como um produto com seus próprios usuários, seu próprio processo de release e seu próprio contrato — versionamento semântico, testes de regressão visual e docs que nunca se descolam do código.

## Pré-requisitos

- Experiência sólida em criar componentes em um framework (uma [Biblioteca de Componentes Reutilizáveis](../../intermediate/06-markdown-editor/) é um bom aquecimento)
- Entendimento de propriedades customizadas de CSS e da cascata
- Familiaridade com um registro de pacotes e versionamento semântico
- Conforto em configurar um build que emite uma biblioteca distribuível

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar uma camada de design tokens pela qual os temas se propagam, desacoplada dos componentes
- Criar componentes acessíveis e componíveis com APIs mínimas e bem tipadas
- Publicar um pacote versionado e comunicar mudanças que quebram via semver + changelog
- Pegar regressões visuais e de acessibilidade automaticamente antes do release
- Manter documentação gerada a partir do código-fonte e sincronizada com ele

## Requisitos Funcionais

1. Os design tokens (cor, espaçamento, tipografia, raio) devem ser definidos uma vez e consumidos por todos os componentes.
2. Pelo menos dois temas (ex.: claro/escuro) devem alternar puramente trocando valores de tokens, sem mudanças de código nos componentes.
3. Todo componente interativo deve ser operável por teclado e expor roles/ARIA corretos.
4. A biblioteca deve compilar em um pacote versionado e tree-shakeable que um app separado possa instalar e importar.
5. A documentação deve renderizar exemplos vivos e interativos de cada componente e suas props.
6. Um teste de regressão visual deve falhar o build quando a saída renderizada de um componente muda inesperadamente.
7. Mudanças que quebram devem incrementar a versão maior e ser registradas em um changelog legível.

## Marcos Sugeridos

1. **Marco 1 — Tokens e primitivos:** Defina a camada de tokens e alguns componentes base que a consomem.
2. **Marco 2 — Temas e a11y:** Adicione troca de temas e torne os componentes amigáveis a teclado e leitor de tela.
3. **Marco 3 — Docs e playground:** Suba o Storybook (ou equivalente) com controles interativos de props.
4. **Marco 4 — Pipeline de release:** Adicione checagens de regressão visual + a11y e um fluxo de publish versionado.

## Esboço de Dados e Interface

```text
        ┌─────────────────────────────────────────────┐
        │              Design tokens                   │
        │  color.* · space.* · font.* · radius.*       │
        └───────────────┬─────────────────────────────┘
                        │ variáveis CSS / objeto de tema
        ┌───────────────▼─────────────────────────────┐
        │   Primitivos: Box, Text, Icon, Stack          │
        └───────────────┬─────────────────────────────┘
        ┌───────────────▼─────────────────────────────┐
        │   Componentes: Button, Input, Modal, Table    │
        └───────────────┬─────────────────────────────┘
        ┌───────┴────────┐        ┌────────────────────┐
        │  Docs (stories) │        │  pacote npm (semver)│
        └────────────────┘        └────────────────────┘

Fluxo de token:  token -> tema -> componente (nunca hex fixo)
Versionamento:   patch=fix · minor=aditivo · major=API/visual que quebra
Portões:         snapshot de regressão visual + scan a11y axe por PR
```

## Desafios Extras

- Adicione um pipeline de tokens (ex.: Style Dictionary) que emite CSS, JS e formatos nativos de uma única fonte.
- Gere um relatório de acessibilidade por componente e publique-o junto à documentação.
- Suporte uma API de tematização em tempo de execução para que consumidores personalizem a marca sem um rebuild.
- Adicione um mecanismo de "deprecações" que avisa em dev quando uma prop prestes a ser removida é usada.

## Definição de Pronto

- [ ] Trocar o tema muda toda a UI trocando tokens, com zero edições em componentes.
- [ ] Um app consumidor instala o pacote e importa apenas os componentes que usa (verificado pelo tamanho do bundle).
- [ ] Todo componente interativo passa na operação apenas por teclado e em um scan automatizado de a11y.
- [ ] Uma mudança visual intencional falha a suíte de regressão até o snapshot ser revisado e atualizado.
- [ ] O changelog e a versão refletem a natureza de cada mudança (patch/minor/major).

## Armadilhas Comuns

- Fixar cores e espaçamentos nos componentes em vez de referenciar tokens, quebrando a tematização.
- Superengenheirar as APIs de componentes com dezenas de props em vez de favorecer composição.
- Deixar os docs se descolarem do código ao escrevê-los à mão em vez de gerá-los a partir da fonte.
- Publicar mudanças que quebram como versões menores, quebrando silenciosamente apps downstream.
- Tratar acessibilidade como uma etapa posterior em vez de um critério de aceite por componente.

## Recursos

- [Documentação do Storybook](https://storybook.js.org/docs) — o padrão para construir e documentar componentes em isolamento.
- [Formato do Design Tokens Community Group](https://tr.designtokens.org/format/) — o padrão emergente para design tokens portáveis.
- [WAI-ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) — padrões autoritativos para componentes acessíveis.
- [Versionamento Semântico](https://semver.org/lang/pt-BR/) — o contrato para comunicar mudança através de números de versão.

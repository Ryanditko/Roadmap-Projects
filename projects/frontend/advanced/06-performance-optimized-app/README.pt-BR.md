# App de Grande Escala Otimizado para Performance

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Pegue uma aplicação que renderiza grandes conjuntos de dados e interações pesadas e a torne genuinamente rápida — não por adivinhação, mas medindo, definindo orçamentos e defendendo-os. Esta é a disciplina por trás de apps performáticos em escala: um primeiro carregamento que envia apenas o código de que a visão inicial precisa, listas de dezenas de milhares de linhas que rolam a 60fps porque apenas as visíveis existem no DOM, e interações que nunca bloqueiam a thread principal o suficiente para parecer travadas. A armadilha é a otimização prematura ou supersticiosa. Aqui você vai fazer profiling primeiro, encontrar o gargalo real, corrigi-lo e provar o ganho contra um orçamento — depois proteger esse orçamento para que uma mudança futura não possa regredi-lo silenciosamente.

## Pré-requisitos

- Um app não trivial que você possa perfilar e otimizar (qualquer projeto [Intermediário](../../intermediate/) serve)
- Conforto para ler um flame chart e uma cascata de rede no dev tools do navegador
- Entendimento do pipeline de renderização do navegador (layout, paint, composite)
- Familiaridade com empacotamento, code splitting e imports dinâmicos

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Definir e aplicar orçamentos de performance (tamanho de bundle, Core Web Vitals) como portões em tempo de build
- Fazer profiling com dev tools para localizar o gargalo real antes de mudar código
- Aplicar code splitting em nível de rota e de componente para reduzir o payload inicial
- Virtualizar listas longas para que o tamanho do DOM permaneça constante independente do tamanho do conjunto de dados
- Manter interações responsivas evitando tarefas longas na thread principal e re-renders desnecessários

## Requisitos Funcionais

1. O app deve renderizar um conjunto de dados de pelo menos 10.000 itens e rolá-lo de forma fluida.
2. O payload inicial de JavaScript deve permanecer abaixo de um orçamento explícito e aplicado.
3. Listas longas devem ser virtualizadas para que a contagem de nós do DOM permaneça aproximadamente constante conforme os dados crescem.
4. Rotas e componentes não críticos devem carregar sob demanda via code splitting, não no bundle inicial.
5. Os Core Web Vitals (LCP, INP, CLS) devem ser medidos e atender limiares definidos.
6. Uma checagem de orçamento de performance deve rodar na CI e falhar o build quando um limiar for excedido.
7. Toda otimização deve ser respaldada por uma medição antes/depois, não por suposição.

## Marcos Sugeridos

1. **Marco 1 — Linha de base e orçamentos:** Faça profiling do app, registre métricas de base e defina orçamentos explícitos.
2. **Marco 2 — Dieta de payload:** Aplique code splitting, tree-shaking e lazy loading para reduzir o bundle inicial.
3. **Marco 3 — Performance de renderização:** Virtualize listas longas e elimine re-renders desperdiçados.
4. **Marco 4 — Guarda-corpos:** Conecte checagens de orçamento e Web Vitals à CI com portões de passa/falha.

## Esboço de Dados e Interface

```text
   Medir ───▶ Identificar gargalo ───▶ Corrigir ───▶ Verificar vs orçamento ──┐
      ▲                                                                       │
      └──────────────────── proteger na CI ◀──────────────────────────────────┘

Lista virtualizada (DOM constante):
  data[0..N]  (N = 10.000+)
       │  renderiza apenas itens cuja linha intersecta o viewport
       ▼
  ┌───────────── viewport ─────────────┐
  │ item 412  item 413  item 414 ...    │  ~20 nós DOM, não 10.000
  └─────────────────────────────────────┘
  espaçador acima (412 linhas) + espaçador abaixo (linhas restantes)

Orçamentos (aplicados na CI):
  JS inicial      <= 170 KB gzipado
  LCP             <= 2,5 s
  INP             <= 200 ms
  CLS             <= 0,1
  rolagem da lista 60 fps (sem tarefa longa > 50 ms)
```

## Desafios Extras

- Adicione um Web Worker para mover computação pesada (ordenação, filtragem) para fora da thread principal.
- Implemente prefetch baseado em rota para que a próxima visão provável esteja quente antes de o usuário clicar.
- Adicione otimização de imagem: `srcset` responsivo, lazy loading e formatos modernos.
- Registre Web Vitals de usuários reais (dados de campo) e compare com medições de laboratório.

## Definição de Pronto

- [ ] Uma lista de 10.000 itens rola a 60fps com uma contagem de nós DOM quase constante.
- [ ] O bundle inicial está dentro do orçamento, verificado por um analisador de bundle.
- [ ] LCP, INP e CLS atendem seus limiares em um perfil de dispositivo intermediário.
- [ ] A CI falha quando uma mudança empurra qualquer métrica além do seu orçamento.
- [ ] Cada otimização tem uma medição antes/depois documentada.

## Armadilhas Comuns

- Otimizar sem fazer profiling primeiro, corrigindo algo que nunca foi o gargalo.
- Virtualizar uma lista mas manter trabalho caro por linha, de modo que a rolagem ainda trava.
- Fazer code splitting tão agressivo que o app cai em cascata por dezenas de pequenos chunks lazy.
- Medir apenas em uma máquina rápida e rede rápida, escondendo a lentidão do mundo real.
- Corrigir métricas uma vez mas não adicionar guarda na CI, deixando o próximo PR regredi-las silenciosamente.

## Recursos

- [web.dev: Core Web Vitals](https://web.dev/articles/vitals) — as definições e limiares de LCP, INP e CLS.
- [web.dev: Performance budgets 101](https://web.dev/articles/performance-budgets-101) — como definir e aplicar orçamentos.
- [MDN: Performance API](https://developer.mozilla.org/pt-BR/docs/Web/API/Performance_API) — medindo timing no navegador.
- [Chrome DevTools: Performance features reference](https://developer.chrome.com/docs/devtools/performance) — perfilando a thread principal e a renderização.

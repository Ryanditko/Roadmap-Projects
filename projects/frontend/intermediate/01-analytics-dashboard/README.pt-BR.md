# Dashboard com Gráficos (Analytics)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Construa um dashboard de analytics que transforma um fluxo de registros brutos — pedidos, visualizações de página, cadastros — em gráficos, cartões de KPI e uma visão filtrável que um stakeholder realmente consiga ler. A parte interessante não é desenhar um único gráfico; é agregar dados no cliente, manter várias visualizações em sincronia sob um filtro compartilhado e fazer isso sem travar a página quando o conjunto de dados cresce. Você vai conectar uma biblioteca de gráficos a dados buscados, adicionar um filtro de intervalo de datas ao qual todo widget reage e permitir que o usuário navegue de um resumo até o detalhe. É um tour compacto por busca de dados, estado derivado e visualização responsiva e acessível.

## Pré-requisitos

- Conforto com um framework de componentes (React, Vue ou Svelte) e seu modelo de estado
- Buscar dados via HTTP e tratar promessas (`fetch`, `axios` ou a camada de dados do seu framework)
- Transformações de array para agregação: `map`, `filter`, `reduce`, agrupamento
- Entendimento básico de layout responsivo com CSS grid ou flexbox
- Uma biblioteca de gráficos à sua escolha (Recharts, Chart.js ou Apache ECharts)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Agregar registros brutos em séries e totais com funções puras e testáveis
- Alimentar múltiplos gráficos a partir de uma única fonte de verdade para que os filtros fiquem sincronizados
- Renderizar gráficos responsivos que redimensionam com seu contêiner sem distorção
- Representar estados de carregamento, vazio e erro de forma distinta para cada widget
- Tornar gráficos acessíveis com alternativas textuais e controles alcançáveis por teclado
- Raciocinar sobre o custo de re-renderização e memoizar agregações caras

## Requisitos Funcionais

1. O dashboard deve buscar um conjunto de dados e exibir pelo menos três tipos de gráfico (ex.: linha, barra, pizza) além de cartões-resumo de KPI.
2. Um filtro de intervalo de datas deve recalcular todo widget a partir do mesmo conjunto filtrado.
3. Os cartões de KPI devem mostrar um número principal e uma comparação com o período anterior.
4. Cada widget deve mostrar um estado de carregamento durante a busca e um estado de erro distinto na falha, com opção de tentar novamente.
5. Quando um filtro não retornar dados, os widgets devem renderizar um estado vazio explícito em vez de um gráfico quebrado.
6. Os gráficos devem redimensionar de forma responsiva e permanecer legíveis em uma viewport estreita.
7. Todo gráfico deve expor uma alternativa textual acessível (resumo, tabela de dados ou `aria-label`) para usuários não visuais.
8. Clicar em um segmento (uma barra, uma fatia) deve abrir uma visão de detalhe filtrada por aquele segmento.

## Marcos Sugeridos

1. **Marco 1 — Buscar e renderizar:** Carregue o conjunto de dados, agregue-o e renderize um gráfico estático mais os cartões de KPI.
2. **Marco 2 — Filtro compartilhado:** Adicione um controle de intervalo de datas e roteie todos os widgets pelos dados filtrados e memoizados.
3. **Marco 3 — Estados e detalhamento:** Adicione estados de carregamento/vazio/erro, dimensionamento responsivo e clique-para-detalhar.

## Esboço de Dados e Interface

```text
Layout
  ┌───────────────────────────────────────────┐
  │ Cabeçalho: título       [ intervalo ▼ ]    │
  ├───────────────────────────────────────────┤
  │ [KPI: Receita] [KPI: Pedidos] [KPI: Users] │
  ├──────────────────────┬────────────────────┤
  │ Linha: receita / dia │ Pizza: por categ.   │
  ├──────────────────────┴────────────────────┤
  │ Barra: pedidos por região (clique → detalhe)│
  └───────────────────────────────────────────┘

Record (bruto, buscado)
  id: string
  date: string ISO-8601
  category: string
  region: string
  amount: number

Estado derivado
  filter:     { from: Date, to: Date }
  series:     { label: string, points: { x: string, y: number }[] }[]
  kpis:       { label, value, deltaPct }[]

GET /api/records?from=<iso>&to=<iso>
  -> 200 [ { id, date, category, region, amount }, ... ]
```

## Desafios Extras

- Adicione atualizações em tempo real: faça polling ou abra um `EventSource`/WebSocket e mescle novos registros aos gráficos.
- Deixe o usuário ligar e desligar séries por uma legenda interativa.
- Adicione exportação em CSV/PNG da visão atual.
- Persista o filtro selecionado na query string da URL para que uma visão seja compartilhável.

## Definição de Pronto

- [ ] Todos os widgets atualizam a partir de um único conjunto filtrado, sem divergência de estado entre eles.
- [ ] Estados de carregamento, vazio e erro são visivelmente distintos e o erro oferece nova tentativa.
- [ ] Os gráficos redimensionam de forma limpa de um desktop largo até uma viewport de largura de celular.
- [ ] Cada gráfico tem uma alternativa textual funcional utilizável sem visão.
- [ ] Agregações caras são memoizadas e não recalculam em re-renderizações não relacionadas.

## Armadilhas Comuns

- Agregar dentro do caminho de renderização, de modo que cada tecla recalcula todo o conjunto — memoize com base no filtro.
- Deixar cada gráfico buscar e filtrar sua própria cópia dos dados, fazendo os widgets discordarem após uma mudança de filtro.
- Tratar um resultado vazio como erro, ou renderizar um gráfico com zero pontos como uma caixa em branco.
- Publicar gráficos baseados só em cor que falham em contraste e são ilegíveis para daltônicos.
- Larguras de gráfico fixas em pixels que transbordam ou cortam em telas pequenas.

## Recursos

- [MDN: Usando Fetch](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API/Using_Fetch) — os fundamentos de carregar dados.
- [Documentação do Recharts](https://recharts.org/en-US/) — gráficos React componíveis.
- [Documentação do Chart.js](https://www.chartjs.org/docs/latest/) — gráficos em canvas agnósticos de framework.
- [web.dev: Visualizações de dados acessíveis](https://web.dev/articles/accessible-data-viz) — alternativas textuais e contraste.
- [MDN: Array.prototype.reduce()](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) — o cavalo de batalha da agregação no cliente.

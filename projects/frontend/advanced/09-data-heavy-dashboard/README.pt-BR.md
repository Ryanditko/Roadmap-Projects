# Dashboard com Muitos Dados (virtualização)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Construa um dashboard analítico que apresenta dezenas de milhares de pontos de dados em tabelas e gráficos, permanece responsivo enquanto o usuário filtra e faz drill-down, e nunca congela o navegador sob o peso dos próprios dados. Esta é a fronteira onde a visualização de dados encontra a engenharia de performance: uma renderização ingênua de 50.000 linhas ou um gráfico com 100.000 pontos vai travar a thread principal e derrubar frames. Você vai combinar virtualização (renderizar apenas o visível), agregação no servidor ou em worker (resumir antes de desenhar) e renderização em canvas/WebGL para os visuais mais pesados. O objetivo é um dashboard que pareça instantâneo independente do tamanho do conjunto de dados, respaldado por prova medida em vez de um "funciona bem na minha máquina".

## Pré-requisitos

- Conforto para renderizar gráficos com uma biblioteca de visualização ou a API Canvas
- Entendimento de virtualização/windowing para listas longas
- Familiaridade com conceitos de agregação de dados (agrupamento, bucketing, downsampling)
- Consciência do orçamento de frame do navegador (~16ms por frame para 60fps)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Virtualizar tabelas e listas grandes para que o DOM permaneça pequeno independente da contagem de linhas
- Agregar e fazer downsampling dos dados para que os gráficos desenhem um resumo significativo, não cada ponto bruto
- Escolher a superfície de renderização certa (SVG vs. Canvas vs. WebGL) para um dado volume de dados
- Manter as interações de filtragem e drill-down responsivas descarregando o trabalho pesado
- Medir e defender a latência de interação contra um orçamento de frame

## Requisitos Funcionais

1. Uma tabela deve exibir um conjunto de pelo menos 50.000 linhas e rolar de forma fluida via virtualização.
2. Os gráficos devem renderizar séries grandes sem cair abaixo de uma taxa de frames aceitável, usando agregação/downsampling conforme necessário.
3. Filtrar o conjunto de dados deve atualizar tabelas e gráficos de forma responsiva, sem congelar a UI.
4. O drill-down (ex.: clicar em uma barra para ver suas linhas subjacentes) deve carregar e renderizar o detalhe rapidamente.
5. Ordenar e buscar em dados grandes não deve bloquear a thread principal de forma perceptível.
6. O uso de memória deve permanecer limitado conforme o usuário navega, sem crescimento indefinido.
7. Toda interação pesada deve ter uma latência medida que atenda a um orçamento declarado.

## Marcos Sugeridos

1. **Marco 1 — Tabela virtualizada:** Renderize 50k+ linhas com windowing; verifique tamanho de DOM constante.
2. **Marco 2 — Gráficos agregados:** Desenhe séries grandes com downsampling em uma superfície apropriada (Canvas/WebGL).
3. **Marco 3 — Filtro e drill-down:** Adicione filtragem responsiva e click-through para visões de detalhe.
4. **Marco 4 — Descarregar e medir:** Mova ordenação/agregação para fora da thread principal e registre latências.

## Esboço de Dados e Interface

```text
   Conjunto bruto (50k+ linhas)
        │
        ├─▶ Camada de agregação (worker / servidor)
        │      agrupar · bucket · downsample  ──▶ resumo pronto p/ gráfico
        │
        └─▶ Tabela virtualizada
               renderiza apenas linhas visíveis (windowing)
               ┌──────────── viewport ────────────┐
               │ linha 8.201 ... 8.230 (~30 DOM)   │
               └───────────────────────────────────┘

   Superfície de renderização por volume:
     < 1k pontos     SVG (interativo, acessível)
     1k–50k pontos   Canvas 2D
     > 50k pontos    WebGL

Interação:  filtro -> recomputa agregado (worker) -> repinta
Metas não funcionais:
  rolagem da tabela    60 fps, contagem de nós DOM constante
  filtro -> atualização < 200 ms percebido
  ordenar 50k linhas   fora da thread principal, UI segue interativa
  memória              limitada ao longo da navegação
```

## Desafios Extras

- Adicione carregamento de dados incremental/streaming para que o dashboard preencha progressivamente.
- Adicione uma visão de série temporal com "zoom + pan" que reagrega na resolução visível.
- Adicione exportação em CSV/Parquet da visão filtrada atual.
- Adicione um heatmap ou visualização geográfica para uma dimensão categórica grande.

## Definição de Pronto

- [ ] Uma tabela de 50.000 linhas rola a 60fps com uma contagem de nós DOM constante.
- [ ] Um gráfico grande renderiza e permanece interativo usando agregação/downsampling.
- [ ] Aplicar um filtro atualiza todas as visões dentro do orçamento de latência declarado.
- [ ] Ordenar um conjunto de dados grande não congela visivelmente a interface.
- [ ] A memória permanece limitada através de filtragem e drill-down repetidos.

## Armadilhas Comuns

- Renderizar cada ponto de dado para SVG, criando dezenas de milhares de nós DOM que travam o navegador.
- Agregar na thread principal, de modo que uma mudança de filtro congela a UI por segundos.
- Fazer downsampling de forma ingênua (descartar pontos) e esconder picos reais que o usuário precisa ver.
- Virtualizar a tabela mas re-montar as linhas a cada tick de rolagem, anulando a otimização.
- Relatar "parece rápido" sem medir a latência de interação em um conjunto de dados realista.

## Recursos

- [TanStack Virtual](https://tanstack.com/virtual/latest) — uma biblioteca headless para virtualizar listas, tabelas e grids grandes.
- [web.dev: Rendering performance](https://web.dev/articles/rendering-performance) — o orçamento de frame e como atingir 60fps.
- [MDN: Canvas API](https://developer.mozilla.org/pt-BR/docs/Web/API/Canvas_API) — desenhando visuais grandes sem nós DOM por ponto.
- [Observable Plot](https://observablehq.com/plot/) — uma gramática concisa para construir visualizações de dados.

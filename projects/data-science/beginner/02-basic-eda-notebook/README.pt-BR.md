# Notebook Básico de EDA

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Antes de alguém treinar um modelo, alguém precisa de fato olhar para os dados. A Análise Exploratória de Dados (EDA) é esse primeiro olhar honesto — você descreve cada variável, plota sua distribuição, verifica como as features se relacionam e levanta as perguntas que valem a pena investigar em seguida. Neste projeto você constrói um notebook de EDA sobre um conjunto de dados à sua escolha e, crucialmente, termina com achados escritos que um leitor não técnico conseguiria acompanhar. A entrega não é uma parede de gráficos, mas uma narrativa: eis o que os dados contêm, eis o que me surpreendeu, eis o que eu investigaria mais a fundo.

## Pré-requisitos

- Python básico e uma biblioteca de dataframes (pandas)
- Uma biblioteca de plotagem (Matplotlib ou Seaborn)
- Conforto para rodar um notebook Jupyter ou Colab
- Um conjunto de dados tabular com mistura de colunas numéricas e categóricas — o [dataset Adult / Census Income da UCI](https://archive.ics.uci.edu/dataset/2/adult) ou um CSV do Kaggle são boas escolhas

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Resumir dimensões, tipos e estatísticas descritivas de um conjunto de dados
- Escolher o gráfico certo para uma variável (histograma, box plot, gráfico de barras, dispersão)
- Ler uma matriz de correlação e raciocinar sobre relações, não apenas números
- Identificar valores ausentes, outliers e distribuições suspeitas
- Transformar observações em perguntas claras e priorizadas e em um resumo escrito

## Requisitos Funcionais

1. O notebook deve carregar o conjunto de dados e relatar suas dimensões e tipos por coluna.
2. Deve produzir estatísticas descritivas para colunas numéricas e contagens de valores para as categóricas.
3. Deve visualizar a distribuição de ao menos três variáveis com tipos de gráfico apropriados.
4. Deve mostrar relações entre ao menos dois pares de variáveis (ex.: dispersão ou barras agrupadas).
5. Deve incluir uma matriz de correlação para features numéricas com uma interpretação curta.
6. Deve relatar explicitamente valores ausentes e quaisquer outliers encontrados.
7. Deve terminar com um resumo escrito dos achados e perguntas de acompanhamento.

## Marcos Sugeridos

1. **Marco 1 — Descrever:** Carregue os dados, relate dimensões e tipos, gere estatísticas descritivas e contagens de valores.
2. **Marco 2 — Visualizar:** Plote distribuições e relações; construa a matriz de correlação.
3. **Marco 3 — Sintetizar:** Documente valores ausentes, outliers e uma lista priorizada de achados e próximas perguntas.

## Esboço de Dados e Interface

```text
Estrutura do notebook (de cima para baixo)
  1. Setup e load        -> df, dimensões (linhas, colunas), dtypes
  2. Univariada          -> describe() para numéricas, value_counts() para categóricas
                            histogramas + box plots
  3. Bivariada           -> dispersão (num vs num), barras agrupadas (cat vs num)
  4. Correlação          -> matriz de correlação numérica + heatmap
  5. Qualidade de dados  -> contagem de nulos por coluna, notas de outlier
  6. Achados             -> 3-5 insights em bullets + perguntas em aberto

Mapeamento gráfico-para-pergunta
  distribuição de uma variável    -> histograma / box plot
  frequências de categorias       -> gráfico de barras
  relação entre dois numéricos    -> gráfico de dispersão
  numérico dividido por categoria -> agrupado/box por grupo
```

## Desafios Extras

- Adicione um relatório de profiling automatizado (ydata-profiling / pandas-profiling) e compare com sua análise feita à mão.
- Formule uma hipótese e teste-a com um teste estatístico simples (teste t ou qui-quadrado).
- Adicione gráficos interativos com Plotly para que o leitor possa passar o mouse e filtrar.
- Segmente a análise por uma categoria-chave e compare distribuições entre segmentos.

## Definição de Pronto

- [ ] O notebook roda de cima a baixo sem erros em um kernel novo.
- [ ] Todo gráfico tem título, rótulos de eixo e uma conclusão de uma linha.
- [ ] Valores ausentes e outliers são quantificados, não apenas mencionados.
- [ ] A matriz de correlação é interpretada em palavras, não deixada como uma grade crua.
- [ ] O resumo final expõe achados que um leitor não técnico conseguiria entender.

## Armadilhas Comuns

- Plotar dezenas de gráficos sem narrativa, de modo que o leitor não aprende nada.
- Ler correlação como causalidade — um coeficiente forte é uma pista, não uma conclusão.
- Usar um histograma para uma variável categórica ou um gráfico de barras para uma contínua.
- Ignorar as escalas dos eixos, de modo que um outlier achata todas as outras barras até a invisibilidade.

## Recursos

- [pandas: Essential basic functionality](https://pandas.pydata.org/docs/user_guide/basics.html) — `describe`, `info`, `value_counts` e afins.
- [Seaborn: Overview of plotting functions](https://seaborn.pydata.org/tutorial/function_overview.html) — escolhendo o gráfico certo.
- [From Data to Viz](https://www.data-to-viz.com/) — uma árvore de decisão da forma dos dados ao gráfico apropriado.
- [ydata-profiling docs](https://docs.profiling.ydata.ai/) — relatórios de EDA automatizados para o desafio extra.

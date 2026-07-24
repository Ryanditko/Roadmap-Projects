# Previsão Básica de Séries Temporais

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Previsão (forecasting) é predição com uma pegadinha: a ordem dos dados importa, o amanhã depende do hoje e você nunca pode embaralhar para chegar a uma divisão treino/teste. Neste projeto você pega uma série temporal real — temperaturas diárias, vendas mensais, demanda de energia por hora — decompõe em tendência e sazonalidade e constrói uma previsão simples para os próximos períodos. Você vai aprender por que a validação cruzada clássica é uma armadilha aqui, como avaliar uma previsão contra um baseline ingênuo e por que um intervalo de confiança importa mais do que uma única linha prevista.

## Pré-requisitos

- Python básico e pandas (especialmente indexação por datetime)
- Uma biblioteca de plotagem (Matplotlib)
- Entendimento de média e médias móveis
- Um conjunto de série temporal com uma coluna de data — o [dataset Air Quality da UCI](https://archive.ics.uci.edu/dataset/360/air+quality) ou qualquer série datada do Kaggle (vendas de varejo, clima) funciona bem

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Carregar, indexar e plotar uma série temporal corretamente por data
- Decompor uma série em componentes de tendência, sazonalidade e resíduo
- Dividir dados de série temporal cronologicamente, nunca aleatoriamente
- Construir uma previsão simples (média móvel ou suavização exponencial) e projetar adiante
- Avaliar uma previsão com MAE/RMSE/MAPE contra um baseline ingênuo, com intervalos

## Requisitos Funcionais

1. O fluxo deve carregar uma série com um índice datetime apropriado e plotá-la.
2. Deve tratar timestamps ausentes ou lacunas explicitamente (reamostrar ou interpolar).
3. Deve decompor a série em componentes de tendência, sazonalidade e resíduo.
4. Deve dividir os dados cronologicamente em treino e uma cauda de teste separada.
5. Deve produzir uma previsão para o horizonte de teste usando ao menos um método.
6. Deve comparar a previsão com um baseline ingênuo (último valor ou naive sazonal).
7. Deve relatar MAE, RMSE e MAPE, e plotar previsão vs real com uma faixa de confiança.

## Marcos Sugeridos

1. **Marco 1 — Carregar e decompor:** Indexe por data, preencha lacunas, plote e decomponha tendência/sazonalidade.
2. **Marco 2 — Prever:** Divida cronologicamente e preveja o horizonte de teste com o método escolhido.
3. **Marco 3 — Avaliar:** Compare com um baseline ingênuo, relate métricas de erro e adicione intervalos de confiança.

## Esboço de Dados e Interface

```text
Formato da série
  index:  datetime (diário/mensal/horário, espaçado uniformemente)
  value:  alvo numérico
  gaps:   reamostrar para frequência fixa; interpolar ou preencher pontos ausentes

Divisão cronológica (NUNCA aleatória)
  |------------------ treino -----------------|---- teste ----|
  ajusta no treino, prevê len(teste) passos à frente

Decomposição
  observado = tendência + sazonal + resíduo   (aditiva)   ou  tendência * sazonal * resíduo

Avaliação vs baseline
  método            MAE    RMSE   MAPE
  naive (último)    8.2    11.0   6.1%
  naive_sazonal     5.4     7.1   3.9%
  seu_modelo        4.1     5.8   3.0%   <- precisa vencer o baseline ingênuo para ser útil
```

## Desafios Extras

- Adicione suavização exponencial de Holt-Winters para capturar tendência e sazonalidade.
- Use validação de origem móvel (walk-forward) em vez de uma única divisão.
- Compare com um modelo ARIMA e discuta os trade-offs.
- Amplie o horizonte de previsão e observe como a faixa de confiança cresce com a distância.

## Definição de Pronto

- [ ] A série é corretamente indexada por data e as lacunas são tratadas explicitamente.
- [ ] Tendência e sazonalidade são separadas e mostradas.
- [ ] A divisão treino/teste é estritamente cronológica.
- [ ] A previsão é comparada com um baseline ingênuo e o vence (ou você explica por que não).
- [ ] MAE, RMSE e MAPE são relatados e uma faixa de confiança é plotada.

## Armadilhas Comuns

- Embaralhar os dados para uma divisão treino/teste aleatória, vazando o futuro para o passado.
- Prever sem um baseline ingênuo, sem conseguir dizer se o modelo agrega algum valor.
- Ignorar a sazonalidade e ficar perplexo com resíduos periódicos.
- Relatar uma única linha de previsão sem incerteza, implicando falsa precisão.

## Recursos

- [statsmodels: Time Series Analysis](https://www.statsmodels.org/stable/tsa.html) — decomposição, suavização e ARIMA.
- [pandas: Time series / date functionality](https://pandas.pydata.org/docs/user_guide/timeseries.html) — indexação, reamostragem e janelas móveis.
- [Forecasting: Principles and Practice (Hyndman)](https://otexts.com/fpp3/) — o livro-texto gratuito e definitivo sobre previsão.
- [scikit-learn: TimeSeriesSplit](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.TimeSeriesSplit.html) — validação cruzada cronológica para o desafio extra.

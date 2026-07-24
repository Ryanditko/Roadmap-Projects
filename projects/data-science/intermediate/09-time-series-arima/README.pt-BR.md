# Previsão de Séries Temporais (ARIMA)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Séries temporais quebram as suposições em que a maioria dos modelos confia: as observações são ordenadas, correlacionadas com o próprio passado, e você nunca pode deixar o futuro vazar para trás. Este projeto constrói um fluxo de previsão ARIMA do jeito certo — testar estacionariedade, diferenciar até a série ficar estável, ler gráficos ACF/PACF para propor ordens do modelo, ajustar e validar em um hold-out cronológico, e produzir previsões *com* intervalos de predição para que a incerteza seja honesta. A disciplina que separa uma previsão real de uma fantasia é o split: o treino sempre precede a janela de validação no tempo, e as métricas são calculadas em pontos genuinamente futuros.

## Pré-requisitos

- Conforto com uma biblioteca de dataframe e plotagem de dados indexados no tempo
- Entender média, variância, autocorrelação e tendência/sazonalidade
- Familiaridade com uma biblioteca de estatística/previsão (statsmodels, pmdarima)
- Uma série temporal univariada com histórico suficiente (observações diárias/mensais)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Testar estacionariedade com o teste Augmented Dickey-Fuller e diferenciar para alcançá-la
- Ler gráficos ACF e PACF para propor ordens candidatas (p, d, q)
- Ajustar ARIMA/SARIMA e selecionar ordens por AIC/BIC junto a diagnósticos de resíduos
- Validar com um split cronológico treino/validação e backtesting de origem móvel
- Produzir previsões pontuais com intervalos de predição e avaliar com MAE, RMSE e MAPE

## Requisitos Funcionais

1. O fluxo deve testar estacionariedade e aplicar diferenciação até a série ficar estacionária.
2. Deve usar evidência de ACF/PACF para justificar ordens candidatas, não só busca automática.
3. Deve dividir a série cronologicamente — treino estritamente antes da validação no tempo.
4. Deve ajustar ARIMA e ARIMA sazonal e compará-los via critérios de informação e métricas de erro.
5. Deve rodar diagnósticos de resíduos (autocorrelação, normalidade) para checar a adequação do modelo.
6. Deve produzir previsões com intervalos de predição, não estimativas pontuais sozinhas.
7. Deve avaliar as previsões com pelo menos duas métricas de erro na janela retida.

## Marcos Sugeridos

1. **Marco 1 — Estacionariedade:** Teste com ADF, diferencie e inspecione ACF/PACF.
2. **Marco 2 — Ajustar e diagnosticar:** Ajuste ARIMA/SARIMA, selecione ordens, cheque resíduos.
3. **Marco 3 — Prever e backtest:** Produza previsões com intervalo e avaliação de origem móvel.

## Esboço de Dados e Interface

```text
Série
  ts     : timestamp ordenado (índice)
  value  : float

Passos
  1. plotar + decompor (tendência / sazonal / resíduo)
  2. teste ADF -> estacionária? se não, diferenciar (d)
  3. ACF/PACF -> candidatos p, q  (sazonal: P, D, Q, s)
  4. dividir cronologicamente: treino = [t0 .. tk] , valid = (tk .. tn]
  5. ajustar ARIMA(p,d,q) / SARIMA -> escolher por AIC + checagem de resíduo
  6. prever horizonte h com intervalo de predição de 95%
  7. métricas no valid: MAE, RMSE, MAPE
  8. backtest de origem móvel para estabilidade
```

## Desafios Extras

- Adicione regressores exógenos (ARIMAX/SARIMAX) como feriados ou preço.
- Compare ARIMA com um baseline simples (naive/naive sazonal) e um modelo estilo Prophet.
- Trate timestamps faltantes e quebras estruturais explicitamente.
- Automatize a seleção de ordens com pmdarima e reconcilie com sua leitura manual de ACF/PACF.

## Definição de Pronto

- [ ] A estacionariedade é testada e a ordem de diferenciação é justificada.
- [ ] As ordens candidatas são apoiadas por ACF/PACF, não só por auto-arima.
- [ ] O split é estritamente cronológico — sem dados do futuro no treino.
- [ ] Os diagnósticos de resíduos confirmam que o modelo capturou a estrutura.
- [ ] As previsões incluem intervalos de predição e são avaliadas com pelo menos duas métricas.

## Armadilhas Comuns

- Dividir a série aleatoriamente, vazando observações futuras para o treino.
- Prever sobre uma série não estacionária e confiar nas bandas de confiança.
- Ler MAPE numa série com valores perto de zero, onde ele explode sem sentido.
- Ignorar a sazonalidade e depois culpar o ARIMA por perder um ciclo anual óbvio.

## Recursos

- [Hyndman & Athanasopoulos: Forecasting Principles and Practice](https://otexts.com/fpp3/) — o livro-texto gratuito canônico.
- [statsmodels: ARIMA e SARIMAX](https://www.statsmodels.org/stable/tsa.html) — implementação e diagnósticos.
- [Duke: Identificando modelos ARIMA via ACF/PACF](https://people.duke.edu/~rnau/411arim.htm) — como ler os gráficos.
- [Wikipedia: teste Augmented Dickey–Fuller](https://en.wikipedia.org/wiki/Augmented_Dickey%E2%80%93Fuller_test) — a checagem de estacionariedade.

# Time Series Forecasting (ARIMA)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Time series break the assumptions most models rely on: the observations are ordered, correlated with their own past, and you must never let the future leak backward. This project builds an ARIMA forecasting workflow the right way — test for stationarity, difference until the series is stable, read ACF/PACF plots to propose model orders, fit and validate on a chronological hold-out, and produce forecasts *with* prediction intervals so the uncertainty is honest. The discipline that separates a real forecast from a fantasy is the split: training always precedes the validation window in time, and metrics are computed on genuinely future points.

## Prerequisites

- Comfort with a dataframe library and plotting time-indexed data
- Understanding of mean, variance, autocorrelation, and trend/seasonality
- Familiarity with a stats/forecasting library (statsmodels, pmdarima)
- A univariate time series with enough history (daily/monthly observations)

## Learning Objectives

By the end, you should be able to:

- Test stationarity with the Augmented Dickey-Fuller test and difference to achieve it
- Read ACF and PACF plots to propose candidate (p, d, q) orders
- Fit ARIMA/SARIMA and select orders by AIC/BIC alongside residual diagnostics
- Validate with a chronological train/validation split and rolling-origin backtesting
- Produce point forecasts with prediction intervals and evaluate with MAE, RMSE, and MAPE

## Functional Requirements

1. The workflow must test stationarity and apply differencing until the series is stationary.
2. It must use ACF/PACF evidence to justify candidate model orders, not only auto-search.
3. It must split the series chronologically — training strictly before validation in time.
4. It must fit ARIMA and seasonal ARIMA and compare them via information criteria and error metrics.
5. It must run residual diagnostics (autocorrelation, normality) to check model adequacy.
6. It must output forecasts with prediction intervals, not point estimates alone.
7. It must evaluate forecasts with at least two error metrics on the held-out window.

## Suggested Milestones

1. **Milestone 1 — Stationarity:** Test with ADF, difference, and inspect ACF/PACF.
2. **Milestone 2 — Fit & diagnose:** Fit ARIMA/SARIMA, select orders, check residuals.
3. **Milestone 3 — Forecast & backtest:** Produce interval forecasts and rolling-origin evaluation.

## Data & Interface Sketch

```text
Series
  ts     : ordered timestamp (index)
  value  : float

Steps
  1. plot + decompose (trend / seasonal / residual)
  2. ADF test -> stationary? if not, difference (d)
  3. ACF/PACF -> candidate p, q  (seasonal: P, D, Q, s)
  4. split chronologically:  train = [t0 .. tk] , valid = (tk .. tn]
  5. fit ARIMA(p,d,q) / SARIMA -> pick by AIC + residual check
  6. forecast horizon h with 95% prediction interval
  7. metrics on valid: MAE, RMSE, MAPE
  8. rolling-origin backtest for stability
```

## Stretch Goals

- Add exogenous regressors (ARIMAX/SARIMAX) such as holidays or price.
- Compare ARIMA against a simple baseline (naive/seasonal-naive) and a Prophet-style model.
- Handle missing timestamps and structural breaks explicitly.
- Automate order selection with pmdarima and reconcile it against your manual ACF/PACF read.

## Definition of Done

- [ ] Stationarity is tested and the differencing order is justified.
- [ ] Candidate orders are supported by ACF/PACF, not just auto-arima.
- [ ] The split is strictly chronological — no future data in training.
- [ ] Residual diagnostics confirm the model captured the structure.
- [ ] Forecasts include prediction intervals and are scored with at least two metrics.

## Common Pitfalls

- Random-splitting a time series, leaking future observations into training.
- Forecasting on a non-stationary series and trusting the confidence bands.
- Reading MAPE on a series with values near zero, where it explodes meaninglessly.
- Ignoring seasonality, then blaming ARIMA for missing an obvious yearly cycle.

## Resources

- [Hyndman & Athanasopoulos: Forecasting Principles and Practice](https://otexts.com/fpp3/) — the canonical free textbook.
- [statsmodels: ARIMA and SARIMAX](https://www.statsmodels.org/stable/tsa.html) — implementation and diagnostics.
- [Duke: Identifying ARIMA models via ACF/PACF](https://people.duke.edu/~rnau/411arim.htm) — how to read the plots.
- [Wikipedia: Augmented Dickey–Fuller test](https://en.wikipedia.org/wiki/Augmented_Dickey%E2%80%93Fuller_test) — the stationarity check.

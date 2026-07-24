# Time Series Basic Forecast

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Forecasting is prediction with a twist: the order of the data matters, tomorrow depends on today, and you can never shuffle your way to a train/test split. In this project you take a real time series — daily temperatures, monthly sales, hourly energy demand — decompose it into trend and seasonality, and build a simple forecast for the next few periods. You will learn why classic cross-validation is a trap here, how to evaluate a forecast against a naive baseline, and why a confidence interval matters more than a single predicted line.

## Prerequisites

- Basic Python and pandas (especially datetime indexing)
- A plotting library (Matplotlib)
- Understanding of mean and moving averages
- A time series dataset with a date column — the [UCI Air Quality dataset](https://archive.ics.uci.edu/dataset/360/air+quality) or any dated Kaggle series (retail sales, weather) works well

## Learning Objectives

By the end, you should be able to:

- Load, index, and plot a time series correctly by date
- Decompose a series into trend, seasonality, and residual components
- Split time series data chronologically, never randomly
- Build a simple forecast (moving average or exponential smoothing) and project forward
- Evaluate a forecast with MAE/RMSE/MAPE against a naive baseline, with intervals

## Functional Requirements

1. The workflow must load a series with a proper datetime index and plot it.
2. It must handle missing timestamps or gaps explicitly (resample or interpolate).
3. It must decompose the series into trend, seasonal, and residual components.
4. It must split the data chronologically into train and a held-out test tail.
5. It must produce a forecast for the test horizon using at least one method.
6. It must compare the forecast to a naive baseline (last value or seasonal naive).
7. It must report MAE, RMSE, and MAPE, and plot forecast vs actual with a confidence band.

## Suggested Milestones

1. **Milestone 1 — Load & decompose:** Index by date, fill gaps, plot, and decompose trend/seasonality.
2. **Milestone 2 — Forecast:** Split chronologically and forecast the test horizon with your chosen method.
3. **Milestone 3 — Evaluate:** Compare to a naive baseline, report error metrics, and add confidence intervals.

## Data & Interface Sketch

```text
Series shape
  index:  datetime (daily/monthly/hourly, evenly spaced)
  value:  numeric target
  gaps:   resample to fixed frequency; interpolate or forward-fill missing points

Chronological split (NEVER random)
  |------------------ train ------------------|---- test ----|
  fit on train, forecast len(test) steps ahead

Decomposition
  observed = trend + seasonal + residual   (additive)   or  trend * seasonal * residual

Evaluation vs baseline
  method            MAE    RMSE   MAPE
  naive (last)      8.2    11.0   6.1%
  seasonal_naive    5.4     7.1   3.9%
  your_model        4.1     5.8   3.0%   <- must beat the naive baseline to be useful
```

## Stretch Goals

- Add Holt-Winters exponential smoothing to capture both trend and seasonality.
- Use rolling-origin (walk-forward) validation instead of a single split.
- Compare against an ARIMA model and discuss the trade-offs.
- Widen the forecast horizon and observe how the confidence band grows with distance.

## Definition of Done

- [ ] The series is correctly indexed by date and gaps are handled explicitly.
- [ ] Trend and seasonality are separated and shown.
- [ ] The train/test split is strictly chronological.
- [ ] The forecast is compared to a naive baseline and beats it (or you explain why not).
- [ ] MAE, RMSE, and MAPE are reported and a confidence band is plotted.

## Common Pitfalls

- Shuffling the data for a random train/test split, leaking the future into the past.
- Forecasting without a naive baseline, so you cannot tell if the model adds any value.
- Ignoring seasonality and being baffled by periodic residuals.
- Reporting a single forecast line with no uncertainty, implying false precision.

## Resources

- [statsmodels: Time Series Analysis](https://www.statsmodels.org/stable/tsa.html) — decomposition, smoothing, and ARIMA.
- [pandas: Time series / date functionality](https://pandas.pydata.org/docs/user_guide/timeseries.html) — indexing, resampling, and rolling windows.
- [Forecasting: Principles and Practice (Hyndman)](https://otexts.com/fpp3/) — the free, authoritative forecasting textbook.
- [scikit-learn: TimeSeriesSplit](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.TimeSeriesSplit.html) — chronological cross-validation for the stretch goal.

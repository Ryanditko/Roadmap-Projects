# Linear Regression Model

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Linear regression is the "hello world" of predictive modeling, and it is worth doing properly because every habit you form here — splitting data, measuring error honestly, checking assumptions — carries into every model you ever build. In this project you predict a continuous target (house price, fuel efficiency, tip amount) from a handful of features, then interpret what the model learned and how much to trust it. The point is not a high score; it is understanding why the score is what it is and what the coefficients actually mean.

## Prerequisites

- Basic Python and pandas
- scikit-learn installed
- Comfort with the idea of features (inputs) and a target (output)
- A regression dataset with a numeric target — the [scikit-learn California Housing dataset](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.fetch_california_housing.html) or the [UCI Auto MPG dataset](https://archive.ics.uci.edu/dataset/9/auto+mpg) work well

## Learning Objectives

By the end, you should be able to:

- Split data into train and test sets and explain why the split matters
- Train a linear regression model and generate predictions
- Evaluate with R², RMSE, and MAE, and explain what each measures
- Read a residual plot to check whether a linear model is appropriate
- Interpret coefficients in the units of the problem, mindful of feature scaling

## Functional Requirements

1. The workflow must load the dataset and select a numeric target plus at least three features.
2. It must split data into training and test sets before any model is fit.
3. It must train a linear regression model on the training set only.
4. It must report R², RMSE, and MAE computed on the held-out test set.
5. It must produce a residual plot (predicted vs residuals) and comment on the pattern.
6. It must report and interpret the learned coefficients.
7. It must use k-fold cross-validation to check that the score is stable, not a lucky split.

## Suggested Milestones

1. **Milestone 1 — Prepare & split:** Load data, choose target and features, scale if needed, split train/test.
2. **Milestone 2 — Train & evaluate:** Fit the model, predict on the test set, report R²/RMSE/MAE.
3. **Milestone 3 — Diagnose:** Plot residuals, interpret coefficients, run cross-validation for stability.

## Data & Interface Sketch

```text
Model I/O
  X (features):  matrix [n_samples, n_features]  (numeric)
  y (target):    vector [n_samples]              (continuous)
  prediction:    y_hat = intercept + sum(coef_i * x_i)

Evaluation report
  R2:    fraction of variance explained (1.0 = perfect, 0 = mean baseline)
  RMSE:  error in target units, penalizes large misses
  MAE:   average absolute error in target units
  CV:    mean +/- std of R2 across k folds

Residual check
  plot(predicted, actual - predicted)
    random cloud around 0  -> linear model is reasonable
    curve or funnel shape   -> non-linearity or heteroscedasticity
```

## Stretch Goals

- Add polynomial features and compare against the plain linear model without overfitting.
- Apply Ridge or Lasso regularization and observe the effect on coefficients.
- Engineer a new feature from existing columns and measure whether it helps.
- Add prediction intervals so each prediction carries an uncertainty range.

## Definition of Done

- [ ] The model is trained on training data only and scored on unseen test data.
- [ ] R², RMSE, and MAE are all reported and explained in one sentence each.
- [ ] A residual plot exists and its shape is interpreted.
- [ ] Coefficients are reported in the problem's units, with scaling accounted for.
- [ ] Cross-validation confirms the test score is not a fluke of one split.

## Common Pitfalls

- Evaluating on the training set and celebrating a score the model will never repeat.
- Interpreting raw coefficients when features are on wildly different scales.
- Fitting the scaler on the full dataset (leaking test info) instead of on train only.
- Chasing a higher R² while ignoring residuals that clearly show a non-linear relationship.

## Resources

- [scikit-learn: Linear Models](https://scikit-learn.org/stable/modules/linear_model.html) — the reference for `LinearRegression`, Ridge, and Lasso.
- [scikit-learn: Cross-validation](https://scikit-learn.org/stable/modules/cross_validation.html) — how and why to use k-fold.
- [Wikipedia: Coefficient of determination (R²)](https://en.wikipedia.org/wiki/Coefficient_of_determination) — what R² does and does not tell you.
- [STAT 501: Regression Methods (Penn State)](https://online.stat.psu.edu/stat501/) — a free, thorough course on regression assumptions.

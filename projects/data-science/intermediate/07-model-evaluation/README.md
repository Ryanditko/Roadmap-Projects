# Model Evaluation Framework

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Two models, one dataset — which is better? The honest answer is harder than reading off a single accuracy number. This project builds a framework that evaluates and compares models the way a careful practitioner does: with proper cross-validation, a suite of task-appropriate metrics, confidence intervals around each score, and a statistical test to decide whether one model truly beats another or just got a luckier split. You will learn that the *evaluation protocol* is itself a design decision — the wrong split strategy or the wrong metric can crown the wrong model with total confidence.

## Prerequisites

- Comfort training classifiers/regressors and reading a confusion matrix
- Understanding of overfitting and the train/validation/test split rationale
- Familiarity with scikit-learn's estimator interface (fit/predict)
- A labelled dataset (classification or regression) to evaluate models on

## Learning Objectives

By the end, you should be able to:

- Implement k-fold, stratified, and time-series cross-validation and know when each applies
- Assemble a metric suite matched to the task (ROC-AUC, PR-AUC, F1 for classification; RMSE, MAE, R² for regression)
- Attach confidence intervals to metrics via cross-validation folds or bootstrapping
- Run a statistical test (paired, e.g. corrected resampled t-test) to compare two models
- Diagnose over/underfitting with learning and validation curves

## Functional Requirements

1. The framework must support multiple cross-validation strategies selectable per run.
2. It must compute a configurable suite of metrics appropriate to the task type.
3. It must report a mean and confidence interval (or standard deviation across folds) for each metric.
4. It must compare at least two models on identical folds and report which wins.
5. It must apply a statistical significance test to the model comparison.
6. It must produce learning and/or validation curves for overfitting diagnosis.
7. It must keep the final test set untouched until the single last evaluation.

## Suggested Milestones

1. **Milestone 1 — CV & metrics:** Implement CV strategies and the metric suite.
2. **Milestone 2 — Intervals & curves:** Add confidence intervals and learning curves.
3. **Milestone 3 — Compare:** Run models on shared folds and test significance of the gap.

## Data & Interface Sketch

```text
Eval config
  task    : "classification" | "regression"
  cv      : "kfold" | "stratified" | "timeseries"
  folds   : int
  metrics : [ ... task-appropriate ... ]
  models  : [ estimatorA, estimatorB ]

Steps
  1. hold out final TEST set, untouched
  2. for each model:
       cross_validate over SAME folds
       collect per-fold scores
  3. summarize: mean +/- CI per metric
  4. compare: paired test on fold scores -> p
  5. learning_curve(model) -> train vs valid gap
  6. one final eval on TEST for the chosen model

Output: metric table + winner + significance
```

## Stretch Goals

- Add nested cross-validation so hyperparameter tuning does not leak into the score.
- Support calibration curves and Brier score for probabilistic classifiers.
- Add a cost-sensitive metric driven by a user-supplied cost matrix.
- Generate a one-page markdown/HTML report per comparison run.

## Definition of Done

- [ ] At least three CV strategies are implemented and chosen appropriately per task.
- [ ] Every metric is reported with a mean and an interval, never a bare point estimate.
- [ ] Two models are compared on identical folds with a significance test on the difference.
- [ ] Learning/validation curves reveal over- or underfitting.
- [ ] The test set is evaluated exactly once, at the end.

## Common Pitfalls

- Using plain k-fold on imbalanced data instead of stratified folds.
- Using random CV on time series, letting the model peek at the future.
- Declaring a winner from a 0.2% metric gap that is well inside the fold variance.
- Tuning hyperparameters on the test set, so the final number is optimistic.

## Resources

- [scikit-learn: Cross-validation](https://scikit-learn.org/stable/modules/cross_validation.html) — strategies and pitfalls.
- [scikit-learn: Metrics and scoring](https://scikit-learn.org/stable/modules/model_evaluation.html) — the full metric catalogue.
- [Nadeau & Bengio: Inference for the Generalization Error](https://link.springer.com/article/10.1023/A:1024068626366) — the corrected resampled t-test.
- [scikit-learn: Learning curves](https://scikit-learn.org/stable/modules/learning_curve.html) — diagnosing bias vs variance.

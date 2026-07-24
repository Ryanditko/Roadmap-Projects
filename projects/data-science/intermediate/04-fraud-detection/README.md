# Fraud Detection Model

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Fraud is rare — maybe one transaction in a thousand — and that rarity breaks every intuition you have about accuracy. A model that predicts "not fraud" for everything scores 99.9% accurate and catches zero fraud. This project is about building a classifier that actually finds the needle: engineering features that expose fraud patterns, handling the class imbalance honestly, and evaluating with metrics that survive skew (precision, recall, PR-AUC). You will also confront the business reality that a false positive (blocking a real customer) and a false negative (missing fraud) cost very different amounts, so the decision threshold is a design choice, not a default.

## Prerequisites

- Comfort training a classifier (logistic regression, tree ensembles) in scikit-learn or similar
- Understanding of the confusion matrix and precision/recall
- Familiarity with train/validation/test splitting
- An imbalanced transaction dataset (e.g. the Kaggle credit-card fraud set)

## Learning Objectives

By the end, you should be able to:

- Engineer transaction features (velocity, amount deviation, time-of-day) that expose fraud
- Handle imbalance with resampling (SMOTE) and/or class weights, applied only to training data
- Choose evaluation metrics that are meaningful under heavy skew (PR-AUC, recall at fixed precision)
- Tune a decision threshold against an explicit cost trade-off, not the default 0.5
- Produce per-prediction explanations to support an analyst reviewing an alert

## Functional Requirements

1. The pipeline must split data into train/validation/test *before* any resampling or fitting.
2. Resampling or class weighting must be applied to the training fold only.
3. It must engineer at least three fraud-relevant features from the raw fields.
4. It must report precision, recall, F1, and PR-AUC — not accuracy alone.
5. It must expose a tunable decision threshold and show the precision/recall trade-off curve.
6. It must output a fraud score per transaction, not just a hard label.
7. It must include a way to explain why a given transaction was flagged.

## Suggested Milestones

1. **Milestone 1 — Split & features:** Split data, then engineer fraud-pattern features.
2. **Milestone 2 — Train & balance:** Train models with imbalance handling on the train fold.
3. **Milestone 3 — Evaluate & threshold:** Report skew-aware metrics and tune the threshold.

## Data & Interface Sketch

```text
Transaction record
  txn_id     : string
  amount     : float
  ts         : epoch seconds
  merchant   : category
  is_fraud   : 0 | 1   (very rare)

Engineered features
  amount_zscore_per_user
  txns_last_1h  (velocity)
  hour_of_day
  amount_vs_merchant_median

Pipeline steps
  1. split -> train / valid / test  (stratified on is_fraud)
  2. engineer features on each split independently
  3. resample TRAIN only (SMOTE | class_weight)
  4. train (logreg | gradient-boosted trees)
  5. evaluate valid: PR-AUC, recall@precision, confusion matrix
  6. pick threshold via cost curve -> apply to test
```

## Stretch Goals

- Add an unsupervised anomaly detector (Isolation Forest) and blend it with the classifier.
- Simulate a cost matrix and report expected monetary loss at each threshold.
- Add temporal validation (train on past, test on future) to catch concept drift.
- Build a simple alert queue that ranks flagged transactions by score.

## Definition of Done

- [ ] The split happens before any fitting or resampling — zero leakage.
- [ ] Resampling touches only the training fold; validation/test stay at natural prevalence.
- [ ] Reported metrics include PR-AUC and recall at a stated precision, not accuracy.
- [ ] The decision threshold is chosen deliberately with a documented trade-off.
- [ ] Each flagged transaction comes with a human-readable reason.

## Common Pitfalls

- Applying SMOTE before the split, leaking synthetic neighbours into validation.
- Reporting 99% accuracy on a 0.1% fraud rate and calling it a success.
- Leaving the threshold at 0.5 when the cost of a miss dwarfs a false alarm (or vice versa).
- Engineering features using future information (e.g. a label-derived aggregate).

## Resources

- [scikit-learn: Imbalanced classification metrics](https://scikit-learn.org/stable/modules/model_evaluation.html#precision-recall-f-measure-metrics) — precision, recall, PR curves.
- [imbalanced-learn docs](https://imbalanced-learn.org/stable/) — SMOTE and resampling done right.
- [Google: Classification on imbalanced data](https://developers.google.com/machine-learning/data-prep/construct/sampling-splitting/imbalanced-data) — practical guidance.
- [Kaggle: Credit Card Fraud Detection dataset](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud) — a canonical imbalanced set.

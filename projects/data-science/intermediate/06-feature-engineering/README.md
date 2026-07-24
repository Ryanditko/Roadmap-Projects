# Feature Engineering Pipeline

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Models are only as good as the features you feed them, and raw columns are rarely the right features. This project builds a reusable pipeline that transforms raw data into model-ready features: encoding categoricals, scaling numerics, creating interactions and temporal features, and then *selecting* the subset that actually helps. The discipline that separates this from a pile of ad-hoc transformations is leak-free fitting — every transformer learns its parameters from the training fold only — and honest selection, where you prove a smaller feature set holds up on held-out data rather than just fitting the training noise better.

## Prerequisites

- Comfort with a dataframe library and scikit-learn (or an equivalent) transformers
- Understanding of the difference between fitting and transforming
- Familiarity with train/validation/test splitting
- A tabular dataset with mixed numeric and categorical columns and a target

## Learning Objectives

By the end, you should be able to:

- Build a composable pipeline of transformers that fits on train and transforms any split
- Encode categoricals (one-hot, target/mean with smoothing) without leaking the target
- Create interaction, polynomial, and time-derived features and judge which earn their keep
- Apply filter, wrapper, and embedded feature-selection methods and compare them
- Rank feature importance and validate that pruned features do not hurt held-out performance

## Functional Requirements

1. The pipeline must be a single fitted object that transforms train, validation, and test identically.
2. All transformer parameters (scaler stats, encoding maps) must be learned from training data only.
3. It must generate at least three engineered feature types (interaction, temporal, categorical encoding).
4. It must implement at least two feature-selection strategies and report the features each keeps.
5. It must produce a ranked feature-importance table.
6. It must compare model performance on the full vs selected feature set on held-out data.
7. It must handle unseen categorical values at transform time without crashing.

## Suggested Milestones

1. **Milestone 1 — Transformers:** Build encoding, scaling, and generation steps into one pipeline.
2. **Milestone 2 — Selection:** Add and compare feature-selection strategies.
3. **Milestone 3 — Validate:** Rank importance and confirm the selected set holds on test.

## Data & Interface Sketch

```text
Raw row
  id          : string
  age         : int
  signup_date : date
  plan        : category
  target      : numeric | class

Pipeline (fit on TRAIN only)
  numeric   -> impute -> scale
  category  -> encode (one-hot | smoothed target)
  temporal  -> days_since, day_of_week, is_weekend
  generate  -> interactions, polynomial terms
  select    -> filter (mutual info) | embedded (L1 / tree importance)

Outputs
  feature_matrix per split
  importance table: feature -> score
  perf(full) vs perf(selected) on held-out
```

## Stretch Goals

- Add a lightweight feature store: persist fitted transformers and version the feature set.
- Detect and drop highly-correlated features automatically before modelling.
- Add automated feature generation (e.g. deep feature synthesis) and prune the explosion.
- Track feature distributions so a downstream drift check can reuse them.

## Definition of Done

- [ ] One fitted pipeline transforms all splits identically and reproducibly.
- [ ] No transformer sees validation or test data during fitting — no target leakage.
- [ ] At least two selection strategies are compared with the features each retains listed.
- [ ] A ranked importance table is produced.
- [ ] Selected-set performance is compared to full-set on held-out data.

## Common Pitfalls

- Fitting the scaler or target encoder on the whole dataset, leaking test statistics.
- Target-encoding a high-cardinality column without smoothing or out-of-fold computation.
- Reading feature importance from the training fit and assuming it generalizes.
- Crashing on an unseen category at inference because the encoder never planned for it.

## Resources

- [scikit-learn: Pipelines and composite estimators](https://scikit-learn.org/stable/modules/compose.html) — building leak-free pipelines.
- [scikit-learn: Feature selection](https://scikit-learn.org/stable/modules/feature_selection.html) — filter, wrapper, embedded methods.
- [category_encoders docs](https://contrib.scikit-learn.org/category_encoders/) — target encoding done safely.
- [Kaggle: Feature Engineering course](https://www.kaggle.com/learn/feature-engineering) — practical patterns and pitfalls.

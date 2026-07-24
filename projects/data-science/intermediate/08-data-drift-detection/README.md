# Data Drift Detection

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

A model trained on last year's data quietly rots when this year's data stops looking like it. This project builds a monitor that watches incoming data against a reference (training) distribution and raises a flag when they diverge — before the model's predictions silently degrade. You will implement statistical distance tests per feature, distinguish feature drift from prediction drift, set thresholds that separate genuine shift from ordinary noise, and produce an interpretable report that says not just *that* drift happened but *which* features moved and by how much. The methodology hinge is a clean, frozen reference window compared against sliding production windows.

## Prerequisites

- Comfort with a dataframe library and basic statistics (distributions, quantiles)
- Understanding of the training vs serving data distinction
- Familiarity with hypothesis testing (p-values, test statistics)
- A dataset you can split into a reference period and later "production" batches

## Learning Objectives

By the end, you should be able to:

- Freeze a reference distribution from training data and compare batches against it
- Detect numeric drift with the Kolmogorov-Smirnov test and categorical drift with chi-square or PSI
- Compute Population Stability Index and interpret its conventional thresholds
- Separate feature (covariate) drift from prediction/target drift and label drift
- Set noise-aware thresholds so ordinary variation does not trigger constant false alarms

## Functional Requirements

1. The monitor must accept a reference dataset and successive production batches.
2. It must compute a per-feature drift statistic appropriate to the feature type.
3. It must report Population Stability Index per feature with a severity band (stable/moderate/severe).
4. It must distinguish input-feature drift from prediction-distribution drift.
5. It must apply configurable thresholds and emit an alert only when they are crossed.
6. It must rank which features drifted most in a given batch.
7. It must track drift over time so trends are visible, not just point-in-time snapshots.

## Suggested Milestones

1. **Milestone 1 — Reference & tests:** Freeze the reference and implement per-feature drift tests.
2. **Milestone 2 — PSI & alerts:** Add PSI, severity bands, and threshold-based alerting.
3. **Milestone 3 — Track & rank:** Log drift over batches and rank the top movers.

## Data & Interface Sketch

```text
Reference window (from training data)
  per numeric feature: histogram / quantiles
  per categorical    : category frequencies

Batch check
  for each feature:
    numeric     -> KS test statistic, p_value
    categorical -> chi-square | PSI
  psi = sum( (p_i - q_i) * ln(p_i / q_i) )
  band: psi<0.1 stable | 0.1-0.25 moderate | >0.25 severe

Report
  batch_id, ts
  feature -> {psi, test_p, band}
  prediction_drift: {psi on model output}
  alert: any(band == severe)
  top_movers: features sorted by psi desc
```

## Stretch Goals

- Add multivariate drift detection (e.g. a domain classifier that tries to tell reference from batch).
- Correlate detected drift with an observed drop in model performance where labels arrive late.
- Add automatic reference-window refresh with a documented policy and guardrails.
- Build a small time-series dashboard of PSI per feature.

## Definition of Done

- [ ] A frozen reference is compared against each production batch, never re-fit silently.
- [ ] Numeric and categorical features each use an appropriate drift test.
- [ ] PSI is reported per feature with severity bands.
- [ ] Feature drift and prediction drift are reported separately.
- [ ] Alerts fire only past configured thresholds and top movers are ranked.

## Common Pitfalls

- Comparing against a reference that keeps updating, so real drift is masked.
- Flagging every tiny p-value on huge batches, where even trivial shifts are "significant".
- Watching only input features and missing that the prediction mix has shifted.
- Using one global threshold for features with very different natural variance.

## Resources

- [Evidently AI: Data drift guide](https://docs.evidentlyai.com/) — practical drift detection and reports.
- [Wikipedia: Kolmogorov–Smirnov test](https://en.wikipedia.org/wiki/Kolmogorov%E2%80%93Smirnov_test) — the numeric drift workhorse.
- [Population Stability Index explained](https://www.listendata.com/2015/05/population-stability-index.html) — PSI formula and thresholds.
- [Google: ML data and concept drift](https://developers.google.com/machine-learning/managing-ml-projects/monitoring) — monitoring in production.

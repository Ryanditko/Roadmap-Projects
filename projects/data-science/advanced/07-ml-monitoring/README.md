# ML Monitoring System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a monitoring system that watches models in production and tells you when they are quietly going wrong. A deployed model rarely fails loudly; it degrades as the world drifts away from its training data, and by the time business metrics dip the damage is done. This project centers on detecting that drift early: monitoring input feature distributions, prediction distributions, and — when labels eventually arrive — real performance. You will implement statistical drift tests, tune alert thresholds against false positives, and close the loop with an automated retraining trigger. The deliverable is the safety net, not the model.

## Prerequisites

- Familiarity with models in production and how they degrade
- Understanding of statistical distances and tests (PSI, KS test, chi-squared)
- Experience with a metrics/dashboard stack (Prometheus + Grafana or similar)
- Comfort with time-series data and delayed ground-truth labels

## Learning Objectives

By the end, you should be able to:

- Distinguish data drift, concept drift, and label delay, and monitor each appropriately
- Implement statistical drift detection on features and predictions
- Compute real performance once delayed labels arrive
- Tune alert thresholds to balance early detection against false positives
- Trigger automated retraining from a monitoring signal

## Functional Requirements

1. The system must log model inputs, predictions, and (when available) ground-truth labels.
2. It must compute feature drift per feature against a reference/training distribution.
3. It must monitor the prediction distribution for shifts independent of labels.
4. When labels arrive, it must compute performance metrics over the corresponding window.
5. It must raise alerts when drift or performance crosses configurable thresholds.
6. It must present dashboards showing drift and performance trends over time.
7. It must expose a retraining trigger that fires on a sustained alert condition.

## Non-Functional Requirements

- **Timeliness:** drift on a monitored feature must be detectable within a bounded window.
- **False-positive control:** alert thresholds must be tunable and the false-positive rate reported.
- **Scalability:** monitoring must handle many features and multiple models without linear blowup in cost.
- **Reproducibility:** the reference distribution and thresholds must be versioned.

## Suggested Milestones

1. **Milestone 1 — Logging:** Capture inputs, predictions, and labels with timestamps and model version.
2. **Milestone 2 — Drift detection:** Implement feature and prediction drift tests against a reference.
3. **Milestone 3 — Performance & alerts:** Compute delayed-label performance and add threshold alerts.
4. **Milestone 4 — Dashboards & retraining:** Build trend dashboards and wire a retraining trigger.

## Data & Interface Sketch

```text
 serving  --> prediction log { ts, model_version, features{}, pred, [label] }
                     |
                     v
 +--------------------------+   window vs reference distribution
 | Drift detectors          |   feature: PSI / KS ;  pred: population shift
 +------------+-------------+
              v
 +--------------------------+   arrives late; joined by request_id
 | Performance calc (labels)|   accuracy, AUC, calibration over window
 +------------+-------------+
              v
 +--------------------------+   thresholds (versioned)
 | Alerting                 |   sustained breach -> notify + trigger
 +------------+-------------+
              v
     Dashboards (drift/perf trends)   +   POST /retrain (on sustained alert)

 Metrics per feature: PSI, distance-to-reference, missing-rate
 Reference: frozen training-period distribution, versioned
```

## Stretch Goals

- Add automatic segmentation to find which slice is drifting, not just that drift exists.
- Support multivariate drift detection, not only per-feature.
- Add a "silent failure" detector using prediction confidence when labels are absent.
- Track drift across multiple model versions on one dashboard.

## Definition of Done

- [ ] Inputs, predictions, and labels are logged and joinable by request ID.
- [ ] Feature and prediction drift are computed against a versioned reference distribution.
- [ ] Delayed-label performance is computed over the correct time window.
- [ ] Alerts fire on sustained threshold breaches, with a reported false-positive rate.
- [ ] A sustained alert can trigger a retraining job automatically.

## Common Pitfalls

- Monitoring only accuracy and being blind until labels arrive weeks later.
- Alerting on every tiny distribution wobble, training the team to ignore alerts.
- Comparing against a stale reference and treating normal seasonality as drift.
- Failing to join predictions to labels correctly, so performance numbers are wrong.

## Resources

- [Google: Data Validation for Machine Learning / TFX](https://www.tensorflow.org/tfx/guide/tfdv) — distribution and schema skew detection.
- [Evidently AI Documentation](https://docs.evidentlyai.com/) — drift metrics and monitoring reports.
- [A Survey on Concept Drift Adaptation (Gama et al., 2014)](https://dl.acm.org/doi/10.1145/2523813) — types of drift and detection methods.
- [Population Stability Index (PSI) explained](https://www.mdpi.com/2227-9091/7/2/53) — a standard drift metric for tabular features.

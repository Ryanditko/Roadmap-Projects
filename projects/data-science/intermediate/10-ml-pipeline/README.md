# ML Pipeline (Train + Validate + Deploy)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

A notebook that trains a good model is a start; a pipeline you can re-run tomorrow, on new data, with a versioned result you can serve, is the actual job. This project ties together everything an intermediate practitioner needs: a reproducible flow from raw data through feature engineering, training, validation, and a served prediction endpoint, with the model and its metrics versioned in a registry. The methodological spine is a clean train/validation/test split enforced *inside* the pipeline, so every re-run evaluates honestly and the number you promote to "production" is the one measured on data the model never saw.

## Prerequisites

- Comfort training a model and building a feature transformer (see [Feature Engineering Pipeline](../06-feature-engineering/))
- Understanding of train/validation/test methodology and evaluation metrics
- Familiarity with functions/modules and a serving option (a small HTTP framework)
- A labelled dataset suitable for a supervised task

## Learning Objectives

By the end, you should be able to:

- Compose data loading, feature engineering, training, and evaluation into one runnable pipeline
- Enforce a leak-free train/validation/test split as a pipeline stage, not an afterthought
- Persist a trained model with its metrics and metadata in a simple registry
- Serve predictions from the persisted model behind an endpoint
- Re-run the whole pipeline reproducibly and compare a new model version to the current one

## Functional Requirements

1. The pipeline must run end to end from raw data to an evaluated, saved model with one command.
2. It must split data into train/validation/test and fit all transformers on training data only.
3. It must evaluate on the held-out test set and record the metrics with the model artifact.
4. It must version each model (id, timestamp, metrics, data hash) in a registry.
5. It must load a chosen model version and serve predictions behind an endpoint.
6. It must validate incoming prediction inputs and reject malformed requests.
7. It must let a new model be compared against the current one before promotion.

## Suggested Milestones

1. **Milestone 1 — Train pipeline:** Load, split, engineer features, train, evaluate on test.
2. **Milestone 2 — Registry:** Persist model + metrics + metadata and version it.
3. **Milestone 3 — Serve & compare:** Load a version, serve predictions, compare candidates.

## Data & Interface Sketch

```text
Registry entry
  model_id   : string
  created_at : ISO-8601
  metrics    : { accuracy, f1, auc, ... on TEST }
  data_hash  : string   (which data produced it)
  path       : artifact location

Pipeline stages
  1. load raw -> validate schema
  2. split -> train / valid / test  (fixed seed, recorded)
  3. fit feature transformers on TRAIN
  4. train model; tune on VALID
  5. final eval on TEST -> metrics
  6. register(model, metrics, metadata)

Serving
  POST /predict  body: { features: {...} }
                 -> 200 { prediction, model_id } | 400 invalid
  GET  /models   -> list versions + metrics
```

## Stretch Goals

- Add scheduled retraining and only promote a candidate if it beats the incumbent on test.
- Emit basic monitoring (prediction latency, input distribution) for a drift check to consume.
- Add rollback: repoint serving to a previous model version by id.
- Containerize the serving component and parameterize the model version it loads.

## Definition of Done

- [ ] One command runs load → split → features → train → test-eval → register.
- [ ] Transformers are fit on training data only; the test metric is on unseen data.
- [ ] Every model version carries its metrics, timestamp, and data hash in the registry.
- [ ] The endpoint serves a chosen version and validates its inputs.
- [ ] A new candidate can be compared to the current model before promotion.

## Common Pitfalls

- Fitting the scaler/encoder before the split, so the "test" metric is inflated.
- Serving a model without recording which data or code produced it — unreproducible.
- Skipping input validation, so the endpoint crashes or silently mispredicts on bad payloads.
- Promoting a new model on a validation number while never touching the true test set.

## Resources

- [scikit-learn: Pipeline](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html) — composing reproducible flows.
- [MLflow: Tracking and Model Registry](https://mlflow.org/docs/latest/index.html) — versioning models and metrics.
- [Google: MLOps continuous delivery](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) — pipeline maturity levels.
- [FastAPI docs](https://fastapi.tiangolo.com/) — a common way to serve model predictions.

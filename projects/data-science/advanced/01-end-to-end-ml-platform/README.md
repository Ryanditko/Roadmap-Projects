# End-to-End ML Platform

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Design and build a small internal ML platform that carries a model from raw data to a monitored production endpoint without any manual glue. The platform ties together four capabilities that are usually separate: versioned data, tracked experiments, a model registry with lifecycle stages, and an automated path from a registered model to a serving deployment. The point is not any single model — you might train something trivial — but the plumbing that makes the whole loop reproducible, auditable, and re-runnable months later by someone who did not build it. This is the MLOps backbone every production data team eventually needs.

## Prerequisites

- Comfort training and evaluating models with a mainstream framework (scikit-learn, PyTorch, or TensorFlow)
- Experience with containers and a CI runner (Docker plus GitHub Actions or similar)
- Familiarity with REST services and object storage (S3/GCS or a MinIO stand-in)
- Having built at least a couple of end-to-end pipelines before (e.g. an intermediate data-pipeline project) helps a lot

## Learning Objectives

By the end, you should be able to:

- Version datasets and tie each model artifact to the exact data and code that produced it
- Track experiments (params, metrics, artifacts) and compare runs objectively
- Operate a model registry with staged promotion (Staging → Production → Archived)
- Automate deployment so a promotion triggers a serving rollout
- Instrument the loop with monitoring and a retraining trigger

## Functional Requirements

1. The platform must version datasets so any model can be traced to the exact snapshot it trained on.
2. Every training run must log parameters, metrics, and artifacts to a queryable experiment tracker.
3. The model registry must store models with metadata and support stage transitions with an audit trail.
4. Promoting a model to Production must automatically trigger a serving deployment without manual file copying.
5. The serving endpoint must expose the current production model version and health via an API.
6. The system must record prediction inputs/outputs for later monitoring and drift analysis.
7. A retraining job must be triggerable both on a schedule and by a monitoring signal.

## Non-Functional Requirements

- **Reproducibility:** re-running a registered model's training must reproduce metrics within a documented tolerance.
- **Availability:** serving should tolerate a single-node failure; target 99.5% for the endpoint.
- **Latency/throughput:** serving p95 under a stated budget (e.g. 200 ms) at a defined request rate.
- **Auditability:** every promotion and deployment is attributable to a user, model version, and timestamp.

## Suggested Milestones

1. **Milestone 1 — Tracking & data versioning:** Stand up experiment tracking (MLflow) and dataset versioning (DVC or lakeFS); log a training run end to end.
2. **Milestone 2 — Registry & promotion:** Register models, implement staged transitions, and record the audit trail.
3. **Milestone 3 — Automated serving:** Wire a promotion event to a containerized deployment behind a stable API.
4. **Milestone 4 — Monitoring & retraining:** Log predictions, compute basic drift, and close the loop with a retraining trigger.

## Data & Interface Sketch

```text
                +-------------+     +------------------+
  raw data ---> | Data Version | -> | Training Job      |
                | (DVC/lakeFS) |    | logs -> Tracker   |
                +-------------+     +---------+--------+
                                              |
                                        register model
                                              v
                                     +------------------+
                                     | Model Registry    |
                                     | Staging|Prod|Arch |
                                     +---------+--------+
                                  promote(Prod) | event
                                              v
                                     +------------------+     +-----------+
   client --> POST /predict ------> | Serving (Prod ver)| --> | pred log  |
              GET  /model/info      +------------------+     +-----+-----+
                                                                   |
                                              drift/perf signal ---+--> retrain

Registered model
  name, version, stage, run_id, data_version, metrics{}, created_by, created_at
```

## Stretch Goals

- Add a feature store so training and serving share the same feature definitions.
- Support shadow/canary deployment: route a slice of traffic to a candidate version.
- Add lineage graphs linking data → run → model → deployment visually.
- Enforce approval gates (a second reviewer) before Production promotion.

## Definition of Done

- [ ] Any production model traces cleanly to its data version, code commit, and run.
- [ ] A promotion event deploys the model automatically with no manual artifact handling.
- [ ] The serving endpoint reports its live model version and passes health checks.
- [ ] Predictions are logged and a drift metric is computed on a schedule.
- [ ] A retraining run can be triggered by both schedule and monitoring signal.

## Common Pitfalls

- Treating experiment tracking as optional and losing the ability to reproduce a "best" model.
- Storing models as loose files instead of registry entries, so lifecycle state lives in someone's head.
- Coupling training and serving code so tightly that a serving change forces a retrain.
- Skipping prediction logging, then having nothing to compute drift against when quality drops.

## Resources

- [MLflow Documentation](https://mlflow.org/docs/latest/index.html) — tracking, registry, and model packaging.
- [Google Cloud: MLOps — Continuous delivery and automation pipelines in ML](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) — the canonical MLOps maturity reference.
- [DVC Documentation](https://dvc.org/doc) — data and pipeline versioning.
- [Hidden Technical Debt in Machine Learning Systems (Sculley et al., 2015)](https://papers.nips.cc/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html) — why the glue matters more than the model.

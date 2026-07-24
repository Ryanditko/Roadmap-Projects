# AutoML Pipeline

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a system that, given a tabular dataset and a target column, searches over preprocessing choices, candidate algorithms, and hyperparameters to return a good model with almost no manual tuning. AutoML is deceptively deep: the search space is enormous, compute is finite, and it is dangerously easy to overfit the validation set through repeated selection. The interesting work is the search strategy (Bayesian optimization beats grid/random for a reason), honest evaluation under a compute budget, and reporting that a human can actually trust. You are building the engine, not calling an off-the-shelf one.

## Prerequisites

- Strong grasp of cross-validation, data leakage, and the bias–variance tradeoff
- Experience with scikit-learn pipelines and several model families
- Familiarity with an optimization library (Optuna or Hyperopt) conceptually
- Comfort reasoning about compute budgets and parallel jobs

## Learning Objectives

By the end, you should be able to:

- Define a structured search space over preprocessing, algorithms, and hyperparameters
- Implement and compare search strategies (random, Bayesian, successive halving)
- Use early stopping and pruning to spend compute where it pays off
- Guard against overfitting the validation set with nested CV or a held-out gate
- Produce a leaderboard and analysis a stakeholder can interpret

## Functional Requirements

1. The system must accept a dataset and target and infer column types (numeric, categorical, datetime).
2. It must construct preprocessing pipelines per column type as part of the search.
3. It must search over at least three algorithm families with their hyperparameters.
4. The search must support a Bayesian strategy and honor a wall-clock or trial budget.
5. Underperforming trials must be pruned/early-stopped rather than run to completion.
6. Final model selection must use an evaluation split not touched during the search.
7. The system must output a ranked leaderboard with metrics and the winning pipeline's config.

## Non-Functional Requirements

- **Reproducibility:** a fixed seed and budget must reproduce the same leaderboard.
- **Throughput:** trials must run in parallel and scale with available workers.
- **Robustness:** a single failing trial must not abort the whole search.
- **Budget adherence:** the search must stop within the configured time/trial limit.

## Suggested Milestones

1. **Milestone 1 — Type inference & preprocessing:** Infer column types and build per-type preprocessing pipelines.
2. **Milestone 2 — Search engine:** Wire an optimizer over one algorithm family with proper CV.
3. **Milestone 3 — Multi-algorithm & pruning:** Add more families, Bayesian search, and early stopping.
4. **Milestone 4 — Honest gate & leaderboard:** Add a held-out selection gate and a reportable leaderboard.

## Data & Interface Sketch

```text
 dataset + target
        |
        v
 +----------------+    infers  {col -> numeric|categorical|datetime}
 | Type inference |
 +-------+--------+
         v
 +----------------------------+
 | Search controller (Optuna) |  budget: N trials or T minutes
 |   sample -> build pipeline |
 |   -> CV score -> prune?    |
 +-------------+--------------+
     parallel workers | trials
                       v
 +----------------------------+
 | Trial: preprocess + model  |  families: {trees, linear, boosting}
 +-------------+--------------+
               v
 +----------------------------+
 | Held-out selection gate    |  <- untouched split
 +-------------+--------------+
               v
   Leaderboard[ {rank, algo, params, cv_score, holdout_score} ]

Anti-leakage: fit preprocessing INSIDE each CV fold, never on full data
```

## Stretch Goals

- Add meta-learning warm starts from prior runs on similar datasets.
- Support multi-objective search (accuracy vs inference latency) with a Pareto front.
- Add automatic feature generation (interactions, target encoding) as search dimensions.
- Persist and resume an interrupted search from its trial history.

## Definition of Done

- [ ] Preprocessing is fit inside each CV fold, with no leakage from the full dataset.
- [ ] At least three algorithm families are searched with a Bayesian strategy.
- [ ] Pruning/early stopping demonstrably saves compute versus exhaustive search.
- [ ] Final selection uses a split untouched during search, and its score is reported separately.
- [ ] A fixed seed and budget reproduce the same leaderboard.

## Common Pitfalls

- Fitting scalers or encoders on the full dataset before CV, leaking test information.
- Selecting the "best" model on the same validation folds used for tuning, then overstating its accuracy.
- Letting one crashing trial kill the whole run instead of logging and skipping it.
- Ignoring the compute budget and reporting results that took ten times longer than allowed.

## Resources

- [Optuna Documentation](https://optuna.readthedocs.io/en/stable/) — Bayesian search, pruners, and study persistence.
- [scikit-learn: Common pitfalls and recommended practices](https://scikit-learn.org/stable/common_pitfalls.html) — leakage and evaluation done right.
- [Auto-sklearn (Feurer et al., 2015)](https://papers.nips.cc/paper/2015/hash/11d0e6287202fced83f79975ec59a3a6-Abstract.html) — meta-learning and ensemble construction in AutoML.
- [Hyperband (Li et al., 2018)](https://jmlr.org/papers/v18/16-558.html) — successive halving for budget-aware search.

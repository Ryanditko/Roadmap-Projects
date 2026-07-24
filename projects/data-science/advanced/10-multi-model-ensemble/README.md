# Multi-Model Ensemble System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a system that combines several models into an ensemble that beats any of them alone — and, just as importantly, knows when it does not. Ensembling is not "average some models"; its power comes from diversity, and combining correlated models buys nothing while multiplying inference cost. This project has you assemble a diverse model pool, implement real combination strategies (stacking with a meta-learner, weighted blending, voting), measure diversity honestly, and weigh the accuracy gain against the compute and latency you pay for it. The interesting tension is that the best ensemble is rarely "all of them."

## Prerequisites

- Solid grasp of bias–variance and why diverse errors cancel
- Experience training multiple model families and doing proper cross-validation
- Understanding of stacking, blending, and the risk of meta-learner leakage
- Comfort measuring inference cost, not just accuracy

## Learning Objectives

By the end, you should be able to:

- Construct a deliberately diverse pool of base models and measure their diversity
- Implement stacking with out-of-fold predictions to avoid leakage into the meta-learner
- Compare stacking, weighted blending, and voting on the same task
- Weigh ensemble accuracy gains against added latency and compute
- Prune the ensemble to the subset that carries most of the benefit

## Functional Requirements

1. The system must train and manage a pool of at least three diverse base models.
2. It must compute out-of-fold predictions so the meta-learner never sees in-fold leakage.
3. It must implement at least two combination strategies (e.g. stacking and weighted blending).
4. It must measure ensemble diversity (e.g. pairwise disagreement or error correlation).
5. It must report ensemble performance against the best single model and a naive average.
6. It must expose the ensemble behind a prediction interface with per-model contribution visible.
7. It must support pruning the pool to a smaller subset with a stated accuracy/cost tradeoff.

## Non-Functional Requirements

- **Accuracy vs cost:** the ensemble's accuracy gain must be reported alongside its added latency/compute.
- **Latency:** total ensemble inference must stay within a stated budget (base models may run in parallel).
- **Reproducibility:** fold assignments, seeds, and weights must reproduce reported results.
- **Robustness:** failure of one base model must degrade the ensemble gracefully, not break it.

## Suggested Milestones

1. **Milestone 1 — Diverse pool:** Train several distinct model families and measure their pairwise diversity.
2. **Milestone 2 — Stacking:** Generate out-of-fold predictions and train a meta-learner without leakage.
3. **Milestone 3 — Compare strategies:** Add weighted blending and voting; compare against best-single and naive-average.
4. **Milestone 4 — Prune & serve:** Prune to a cost-effective subset and serve with per-model contributions.

## Data & Interface Sketch

```text
 training data
      |
      +--> Model A (trees)      \
      +--> Model B (boosting)    >  base pool (diverse!)
      +--> Model C (neural net) /
      +--> Model D (linear)    /
             |
             v   out-of-fold predictions (no leakage)
   +-----------------------------+
   | OOF prediction matrix        |   rows=samples, cols=base models
   +--------------+--------------+
                  v
   +------ combine (choose) ------+
   | stacking:  meta-learner(OOF) |
   | blending:  weighted avg      |
   | voting:    majority/soft     |
   +--------------+--------------+
                  v
   diversity: pairwise disagreement, error-correlation matrix
   report: ensemble vs best-single vs naive-avg  (acc AND latency)

 GET /predict -> { prediction, per_model: {A:.., B:..}, weights }
```

## Stretch Goals

- Add dynamic per-instance weighting (choose experts based on the input region).
- Add online updating so weights adapt as new labeled data arrives.
- Add explainability that attributes the final prediction back to contributing models.
- Search for the optimal subset with a greedy or evolutionary ensemble-selection method.

## Definition of Done

- [ ] The base pool has measured diversity, not just multiple copies of the same model.
- [ ] Stacking uses out-of-fold predictions with no leakage into the meta-learner.
- [ ] At least two combination strategies are compared against best-single and naive-average baselines.
- [ ] Accuracy gains are reported alongside the added latency/compute cost.
- [ ] A pruned subset is offered with an explicit accuracy-vs-cost tradeoff.

## Common Pitfalls

- Building a "diverse" ensemble of highly correlated models that adds cost but no accuracy.
- Training the meta-learner on in-fold predictions and leaking, inflating apparent gains.
- Reporting only the accuracy win and hiding that inference now costs 5x as much.
- Assuming more models is always better instead of pruning to the effective subset.

## Resources

- [scikit-learn: Ensemble methods](https://scikit-learn.org/stable/modules/ensemble.html) — stacking, voting, and boosting APIs and theory.
- [Stacked Generalization (Wolpert, 1992)](https://www.sciencedirect.com/science/article/abs/pii/S0893608005800231) — the original stacking paper.
- [Ensemble Selection from Libraries of Models (Caruana et al., 2004)](https://dl.acm.org/doi/10.1145/1015330.1015432) — greedy ensemble pruning.
- [Popular Ensemble Methods: An Empirical Study (Opitz & Maclin, 1999)](https://www.jair.org/index.php/jair/article/view/10239) — why diversity drives ensemble gains.

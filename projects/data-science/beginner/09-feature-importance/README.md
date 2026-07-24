# Feature Importance Analysis

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

A model that predicts well but cannot explain itself is a hard sell to anyone who has to act on it. Feature importance is how you answer "which inputs actually drive the prediction?" — and it is trickier than it looks, because different methods can disagree and correlated features can steal each other's credit. In this project you train a model on a tabular dataset, then rank its features using at least two importance methods, compare the rankings, and turn the result into plain-language recommendations. The goal is interpretation you can defend, not a single leaderboard you take on faith.

## Prerequisites

- Basic Python, pandas, and scikit-learn
- Having trained at least one model before (see [Linear Regression Model](../03-linear-regression/) or [Classification Model](../04-classification-iris/))
- Understanding of features and a target
- A tabular dataset with several features — the [UCI Wine Quality dataset](https://archive.ics.uci.edu/dataset/186/wine+quality) or [scikit-learn's diabetes dataset](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_diabetes.html) work well

## Learning Objectives

By the end, you should be able to:

- Compute feature importance with more than one method
- Explain how model-based, permutation, and SHAP importances differ
- Recognize how correlated features distort importance rankings
- Visualize and communicate importance clearly
- Translate a ranking into actionable, honestly-hedged recommendations

## Functional Requirements

1. The workflow must train a predictive model on a tabular dataset.
2. It must compute feature importance with at least two distinct methods.
3. It must present each ranking as a sorted, labeled bar chart.
4. It must compare the rankings and discuss where and why they disagree.
5. It must identify at least one pair of correlated features and note the effect on importance.
6. It must produce a permutation-importance ranking computed on held-out data.
7. It must end with plain-language recommendations about which features matter.

## Suggested Milestones

1. **Milestone 1 — Train:** Fit a model good enough to interpret, and confirm its baseline performance.
2. **Milestone 2 — Rank:** Compute model-based and permutation importance; visualize both.
3. **Milestone 3 — Reconcile:** Compare rankings, inspect correlations, and write the recommendations.

## Data & Interface Sketch

```text
Importance methods (all return: feature -> score)
  model-based    tree feature_importances_ or linear |coef|   (fast, can be biased)
  permutation    shuffle one column, measure performance drop   (model-agnostic)
  SHAP           per-prediction contribution, averaged          (detailed, slower)

Ranking table (compare side by side)
  feature        model_imp   perm_imp   rank_agrees?
  alcohol          0.28        0.31        yes
  density          0.19        0.05        NO  <- likely correlated w/ alcohol
  citric_acid      0.04        0.03        yes

Correlation check
  corr(density, alcohol) = -0.69  -> importance may be split/stolen between them
```

## Stretch Goals

- Add SHAP values and compare their ranking to permutation importance.
- Add partial dependence plots for the top two features to show direction of effect.
- Drop the lowest-ranked features and check whether performance survives.
- Repeat the analysis with a second model type and see if the top features are stable.

## Definition of Done

- [ ] A trained model with a stated baseline score exists to interpret.
- [ ] At least two importance methods are computed and charted.
- [ ] Permutation importance is computed on held-out, not training, data.
- [ ] Disagreements between rankings are explained, including a correlation effect.
- [ ] Recommendations are written in plain language with appropriate caveats.

## Common Pitfalls

- Trusting tree-based `feature_importances_` alone — it is biased toward high-cardinality features.
- Computing permutation importance on training data, which rewards memorization.
- Reading importance as causation ("this feature causes the outcome").
- Splitting importance across two correlated features and concluding both are weak.

## Resources

- [scikit-learn: Permutation feature importance](https://scikit-learn.org/stable/modules/permutation_importance.html) — the model-agnostic method and its caveats.
- [scikit-learn: Feature importances caveats](https://scikit-learn.org/stable/auto_examples/inspection/plot_permutation_importance.html) — why impurity-based importance misleads.
- [SHAP documentation](https://shap.readthedocs.io/) — additive per-prediction explanations.
- [Interpretable Machine Learning (Molnar)](https://christophm.github.io/interpretable-ml-book/) — a free book on importance and interpretation.

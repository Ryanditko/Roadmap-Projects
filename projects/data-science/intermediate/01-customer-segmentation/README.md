# Customer Segmentation (Clustering)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Take a table of customer behaviour — recency, frequency, monetary value, tenure, product mix — and discover the natural groups hiding inside it. This is an unsupervised project, so there is no label to predict and no accuracy to chase; the win is a set of segments a marketing team can actually name and act on ("dormant high-spenders", "new bargain hunters"). The real work is upstream of the algorithm: scaling features so distance means something, choosing how many clusters to keep, and proving the grouping is stable rather than an artifact of one random seed.

## Prerequisites

- Comfort with a dataframe library (pandas, Polars, or R) and basic plotting
- Familiarity with descriptive statistics and standardization (z-score, min-max)
- A first exposure to K-means or nearest-neighbour thinking
- A tabular customer dataset (e.g. an e-commerce or RFM sample)

## Learning Objectives

By the end, you should be able to:

- Engineer RFM-style features and justify each scaling choice you make
- Run K-means and pick *k* using the elbow method **and** silhouette score, not just one
- Compare a centroid method (K-means) against a density method (DBSCAN) and explain when each fits
- Reduce dimensionality with PCA for visualization without leaking it into the clustering
- Profile each cluster into a business-readable persona with supporting statistics

## Functional Requirements

1. The pipeline must load raw customer records and derive a documented feature table.
2. Features must be scaled, and the chosen scaler must be fit on training data only, then reused.
3. The tool must run at least two clustering algorithms and report cluster sizes for each.
4. Optimal cluster count must be selected using at least two independent diagnostics.
5. Each resulting cluster must be summarized with per-feature means and a written persona.
6. Cluster stability must be checked by re-running with different random seeds or subsamples.
7. Results must be visualized in 2D (PCA or t-SNE) with clusters colour-coded.

## Suggested Milestones

1. **Milestone 1 — Feature table:** Build and scale RFM features from raw transactions.
2. **Milestone 2 — Cluster & tune:** Run K-means, sweep *k*, choose it with elbow + silhouette.
3. **Milestone 3 — Compare & profile:** Add DBSCAN, check stability, and write cluster personas.

## Data & Interface Sketch

```text
Feature table (one row per customer)
  customer_id   : string
  recency_days  : integer   (days since last purchase)
  frequency     : integer   (# orders in window)
  monetary      : float     (total spend)
  tenure_days   : integer
  avg_basket    : float

Pipeline steps
  1. aggregate transactions -> per-customer features
  2. scale features (StandardScaler, fit on train split)
  3. cluster (KMeans k=2..10 | DBSCAN eps sweep)
  4. score (inertia elbow, silhouette) -> pick k
  5. project to 2D (PCA) for plotting only
  6. profile: groupby(cluster).mean() -> persona table
```

## Stretch Goals

- Add Gaussian Mixture Models and compare soft vs hard cluster assignment.
- Score new customers into existing segments without refitting the whole model.
- Weight monetary features by recency to capture recent behaviour shifts.
- Build a small dashboard letting a stakeholder filter customers by segment.

## Definition of Done

- [ ] The feature table is reproducible from raw data with a single documented step.
- [ ] Scaling is fit on a training split only — no leakage from the full dataset.
- [ ] The chosen *k* is defended with both elbow and silhouette evidence.
- [ ] At least two algorithms are compared with a clear recommendation.
- [ ] Every cluster has a named persona backed by per-feature statistics.

## Common Pitfalls

- Clustering unscaled features, letting `monetary` dominate every distance.
- Reading the elbow plot as gospel — it is often ambiguous; pair it with silhouette.
- Treating DBSCAN noise points (label -1) as a real cluster in the profiles.
- Fitting PCA or the scaler on all data, then wondering why segments feel unstable.

## Resources

- [scikit-learn: Clustering](https://scikit-learn.org/stable/modules/clustering.html) — algorithms and trade-offs side by side.
- [scikit-learn: Silhouette analysis](https://scikit-learn.org/stable/auto_examples/cluster/plot_kmeans_silhouette_analysis.html) — choosing *k* visually.
- [Wikipedia: RFM (market research)](https://en.wikipedia.org/wiki/RFM_(market_research)) — the classic customer feature framework.
- [scikit-learn: PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) — dimensionality reduction for visualization.

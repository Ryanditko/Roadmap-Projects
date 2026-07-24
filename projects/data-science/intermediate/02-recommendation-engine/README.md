# Recommendation Engine (Collaborative Filtering)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Given a sparse table of who rated (or bought, or clicked) what, predict what a user will want next. This project builds a collaborative-filtering recommender and — just as importantly — an honest way to measure it. The trap in recommenders is that a naive accuracy number looks great while the recommendations are useless (always suggest the most popular item and you will be "right" a lot). You will build neighbourhood and matrix-factorization models, split ratings so the test set simulates the future, and evaluate with ranking metrics that reward putting the right items near the top.

## Prerequisites

- Comfort with NumPy/pandas and basic linear algebra (dot products, matrices)
- Understanding of sparse data and why a full user-item matrix is mostly empty
- Familiarity with train/test splitting concepts
- A ratings dataset (MovieLens is the canonical choice)

## Learning Objectives

By the end, you should be able to:

- Build a user-item matrix and reason about its sparsity
- Implement item-based and user-based collaborative filtering with a similarity metric
- Apply matrix factorization (SVD-style latent factors) and interpret the factors
- Split interactions *temporally or leave-one-out* so evaluation mimics real prediction
- Evaluate with ranking metrics (Precision@K, Recall@K, NDCG) rather than raw RMSE alone

## Functional Requirements

1. The system must build a user-item interaction matrix from raw event/rating data.
2. It must produce a top-N recommendation list for any given user.
3. It must implement at least two approaches (e.g. item-based CF and matrix factorization).
4. Interactions must be split into train/validation/test so no test interaction leaks into training.
5. The system must report Precision@K, Recall@K, and one rank-aware metric (NDCG or MAP).
6. It must handle the cold-start case: a user or item with no history returns a sensible fallback.
7. It must compare its models against a popularity baseline and report the lift.

## Suggested Milestones

1. **Milestone 1 — Matrix & baseline:** Build the interaction matrix and a most-popular baseline.
2. **Milestone 2 — CF & factorization:** Add item-based CF and an SVD-style model.
3. **Milestone 3 — Evaluate & rank:** Split data, compute ranking metrics, compare to baseline.

## Data & Interface Sketch

```text
Interaction record
  user_id : string
  item_id : string
  rating  : float | implicit 1.0
  ts      : epoch seconds

Matrix R  (users x items), mostly empty (sparse)

Pipeline steps
  1. split: per-user leave-last-N-out -> train / valid / test
  2. build R from train only
  3. model:
       item-CF  -> similarity(item_i, item_j) via cosine
       MF       -> R ~= U * V^T  (latent factors)
  4. recommend: score unseen items, take top-N
  5. evaluate on test: Precision@K, Recall@K, NDCG@K
  6. compare vs popularity baseline -> lift
```

## Stretch Goals

- Add a hybrid model mixing content features (genre, tags) with collaborative signal.
- Introduce diversity/novelty into the ranking so it does not only surface blockbusters.
- Support implicit feedback with confidence weighting instead of explicit ratings.
- Serve recommendations behind a small API that returns top-N for a user id.

## Definition of Done

- [ ] No test interaction is ever visible to a model during training.
- [ ] Top-N lists are produced for users, including cold-start fallbacks.
- [ ] At least two models are evaluated with the same ranking metrics.
- [ ] A popularity baseline is included and beaten (or the gap is explained).
- [ ] Metric definitions (K value, split strategy) are documented.

## Common Pitfalls

- Random-splitting ratings so a user's future leaks into their training rows.
- Reporting RMSE only — a low error can still rank items badly for top-N.
- Forgetting the popularity baseline, so you cannot tell if the model adds value.
- Treating unseen items as disliked (0) when the feedback is implicit, not negative.

## Resources

- [MovieLens datasets](https://grouplens.org/datasets/movielens/) — the standard recommender benchmark.
- [Google: Recommendation Systems course](https://developers.google.com/machine-learning/recommendation) — CF and matrix factorization explained.
- [Surprise library docs](https://surprise.readthedocs.io/en/stable/) — reference for CF and evaluation splits.
- [Wikipedia: Discounted cumulative gain](https://en.wikipedia.org/wiki/Discounted_cumulative_gain) — how NDCG rewards ranking order.

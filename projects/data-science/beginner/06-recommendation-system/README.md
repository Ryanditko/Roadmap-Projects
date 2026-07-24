# Simple Recommendation System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

"Because you watched X, you might like Y" is powered by a surprisingly approachable idea: find things that are similar and recommend those. In this project you build a basic recommender from a ratings dataset using similarity metrics — either item-to-item ("people who liked this also liked...") or user-to-user ("people like you enjoyed..."). You will build the interaction matrix, compute similarities, generate top-N recommendations, and confront the two problems every recommender faces: what to do about brand-new users or items (cold start) and how to know whether the recommendations are any good.

## Prerequisites

- Basic Python and pandas
- NumPy, and ideally scikit-learn for similarity functions
- Understanding of a matrix as rows-by-columns of numbers
- A user-item ratings dataset — the [MovieLens 100K dataset](https://grouplens.org/datasets/movielens/100k/) is the standard choice

## Learning Objectives

By the end, you should be able to:

- Build a user-item interaction (ratings) matrix from raw records
- Compute similarity between items or users with cosine similarity
- Generate top-N recommendations from similarity scores
- Reason about the cold-start problem and offer a popularity fallback
- Evaluate recommendations with a held-out set using precision@k or recall@k

## Functional Requirements

1. The system must build a user-item matrix from a ratings file.
2. It must compute a similarity score between items (or users) using a stated metric.
3. Given a user or item, it must return the top-N most relevant recommendations.
4. It must exclude items the user has already rated from their recommendations.
5. It must provide a popularity-based fallback for users or items with no history.
6. It must evaluate quality on a held-out test set with precision@k or recall@k.
7. It must attach a short reason to each recommendation ("similar to X you rated highly").

## Suggested Milestones

1. **Milestone 1 — Matrix:** Load ratings and build the user-item matrix, noting how sparse it is.
2. **Milestone 2 — Recommend:** Compute similarities and produce top-N recommendations, excluding seen items.
3. **Milestone 3 — Evaluate & fall back:** Hold out ratings, measure precision@k, and add a cold-start fallback.

## Data & Interface Sketch

```text
Interaction matrix (users x items, mostly empty)
              item_1  item_2  item_3  ...  item_m
  user_1        5       -       3           -
  user_2        -       4       -           2
  ...          (sparse: most cells unrated)

Recommendation request/response
  recommend(user_id, n=5)
    -> [ { item_id, score, reason: "similar to <item> you rated 5" }, ... ]
       excludes items already rated by user_id
    -> if user_id unknown: return top-n most popular items

Similarity
  cosine(a, b) = dot(a, b) / (||a|| * ||b||)   in {0..1} for non-negative ratings
Evaluation
  precision@k = (relevant items in top-k) / k   on held-out ratings
```

## Stretch Goals

- Combine collaborative similarity with a popularity prior into a simple hybrid.
- Add diversity so the top-N is not five near-identical items.
- Compare item-based vs user-based recommendations on the same test set.
- Add matrix factorization (SVD) and compare its precision@k to the similarity approach.

## Definition of Done

- [ ] The user-item matrix is built and its sparsity is reported.
- [ ] Recommendations exclude items the user has already rated.
- [ ] A cold-start user receives sensible popularity-based recommendations.
- [ ] Precision@k or recall@k is measured on a held-out set, not the training data.
- [ ] Each recommendation carries a human-readable reason.

## Common Pitfalls

- Recommending items the user already rated because you forgot to mask them out.
- Ignoring sparsity — most users rate almost nothing, so naive similarity is noisy.
- Evaluating on the same ratings used to build the matrix, inflating the score.
- Letting a few blockbuster items dominate every recommendation list.

## Resources

- [MovieLens datasets (GroupLens)](https://grouplens.org/datasets/movielens/) — the classic ratings data.
- [scikit-learn: Pairwise metrics (cosine similarity)](https://scikit-learn.org/stable/modules/metrics.html#cosine-similarity) — computing similarity efficiently.
- [Google: Recommendation Systems crash course](https://developers.google.com/machine-learning/recommendation) — collaborative filtering and cold start explained.
- [Wikipedia: Collaborative filtering](https://en.wikipedia.org/wiki/Collaborative_filtering) — the concepts behind item- and user-based methods.

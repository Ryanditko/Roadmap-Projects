# Recommendation System at Scale

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Science · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Design a recommendation system that stays fast and relevant when the catalog is huge and traffic is heavy. The naive "score every item for every user" approach dies at scale, so this project centers on the two-stage industry pattern: a cheap candidate-retrieval step that narrows millions of items to a few hundred, followed by a heavier ranking model over that shortlist. Around it you will handle cold start, keep results diverse, run online updates, and measure quality with an A/B-style offline evaluation. The goal is a system whose latency and quality both hold up as the catalog grows.

## Prerequisites

- Understanding of collaborative filtering and matrix factorization
- Experience with embeddings and approximate nearest-neighbor search
- Familiarity with a streaming or batch data tool (Kafka, Spark, or similar)
- Comfort with offline ranking metrics (recall@k, NDCG, MAP)

## Learning Objectives

By the end, you should be able to:

- Build a two-stage retrieval-then-ranking recommendation architecture
- Use embeddings plus ANN indexing for sub-linear candidate retrieval
- Handle cold start for new users and items
- Balance relevance against diversity and freshness in the ranker
- Evaluate recommendations offline with ranking metrics and reason about online A/B design

## Functional Requirements

1. The system must generate candidates for a user from a large catalog in sub-linear time (ANN, not full scan).
2. A ranking model must re-score candidates using richer features than the retrieval stage.
3. The system must handle cold start for new users and new items with an explicit strategy.
4. Recommendations must include a diversity/de-duplication step so results are not near-identical.
5. The system must support online updates so new interactions influence future recommendations.
6. Offline evaluation must report ranking metrics (recall@k, NDCG) on a held-out period.
7. The system must expose a recommendation API returning ranked items with scores.

## Non-Functional Requirements

- **Latency:** end-to-end recommendation p95 within a stated budget at target QPS.
- **Scalability:** retrieval latency must stay sub-linear as the catalog grows by an order of magnitude.
- **Freshness:** new interactions must influence recommendations within a bounded delay.
- **Consistency:** the offline evaluation split must respect time (no future leakage into the past).

## Suggested Milestones

1. **Milestone 1 — Embeddings & retrieval:** Train item/user embeddings and build an ANN index for candidate retrieval.
2. **Milestone 2 — Ranking:** Add a ranking model over candidates with interaction and context features.
3. **Milestone 3 — Cold start & diversity:** Add cold-start handling and a diversity re-ranking step.
4. **Milestone 4 — Online & evaluation:** Add online updates and a time-respecting offline evaluation harness.

## Data & Interface Sketch

```text
 interactions (user, item, ts, signal)
        |
        v
 +------------------+   train user/item embeddings
 | Embedding model  |
 +--------+---------+
          v
 +------------------+   millions -> ~hundreds
 | ANN retrieval    |   (FAISS/ScaNN)  <-- cold start fallback: popularity/content
 +--------+---------+
          v
 +------------------+   rich features: recency, context, cross features
 | Ranking model    |
 +--------+---------+
          v
 +------------------+   MMR / category caps
 | Diversity re-rank|
 +--------+---------+
          v
   GET /recommend?user_id=U&k=20 -> [ {item_id, score, reason} ... ]

 Eval: split by TIME; recall@k, NDCG@k on the future window
 Online: stream new interactions -> update embeddings/features
```

## Stretch Goals

- Add a bandit (epsilon-greedy or Thompson sampling) for exploration vs exploitation.
- Support session-based recommendations using sequence models.
- Add a feature store shared between offline training and online serving.
- Precompute recommendations for the hottest users and serve them from a cache.

## Definition of Done

- [ ] Candidate retrieval uses ANN and stays sub-linear as the catalog grows.
- [ ] A ranking stage re-scores candidates with features beyond those used in retrieval.
- [ ] Cold-start users and items get sensible recommendations via an explicit fallback.
- [ ] A diversity step prevents near-duplicate result lists.
- [ ] Offline evaluation respects time ordering and reports recall@k and NDCG.

## Common Pitfalls

- Scoring the entire catalog per request and blowing the latency budget as it grows.
- Evaluating with a random split, leaking future interactions into the training window.
- Optimizing pure relevance until every list looks identical and users disengage.
- Ignoring cold start, so new items are never surfaced and never gather signal.

## Resources

- [Google: Recommendation Systems course](https://developers.google.com/machine-learning/recommendation) — retrieval, ranking, and candidate generation.
- [FAISS Documentation](https://faiss.ai/) — approximate nearest-neighbor search at scale.
- [Deep Neural Networks for YouTube Recommendations (Covington et al., 2016)](https://research.google/pubs/pub45530/) — the canonical two-stage architecture.
- [Matrix Factorization Techniques for Recommender Systems (Koren et al., 2009)](https://ieeexplore.ieee.org/document/5197422) — foundational collaborative filtering.

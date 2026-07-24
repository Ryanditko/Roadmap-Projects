# Design a Recommendation System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design the backend of a recommendation system like the ones behind Netflix rows or an e-commerce "you may also like" strip: given a user, produce a short, ranked list of items they are likely to engage with, fast enough to render on page load. The architectural pattern is a two-stage funnel — cheap candidate generation over millions of items, then expensive ranking over a few hundred. The hard problems are the cold-start user, serving latency, and keeping recommendations fresh. This is a design exercise: you produce a design document, capacity numbers, and diagrams — not working code.

## Prerequisites

- Understanding of collaborative filtering vs. content-based recommendation at a conceptual level
- Familiarity with offline (batch) vs. online (serving) systems
- Awareness of embeddings and approximate nearest-neighbor (ANN) search
- Comfort estimating serving QPS and precomputed-data storage

## Learning Objectives

By the end, you should be able to:

- Design a candidate-generation → ranking two-stage architecture
- Estimate serving QPS and storage for precomputed recommendations and embeddings
- Handle the cold-start problem for new users and new items
- Design a serving cache so recommendations render within the latency budget
- Justify trade-offs between precomputed (batch) and real-time recommendations

## Requirements & Constraints

- Assume 50M users, 10M items, ~30k recommendation requests/s at peak.
- A recommendation response must return in under ~150 ms p99.
- New users (no history) and new items (no interactions) must still get reasonable results.
- Recommendations should reflect recent behavior within minutes, not days.
- Estimate serving QPS, embedding storage, and precomputed-recs storage.

## Suggested Approach

1. Split offline (train models, build embeddings, batch-precompute) from online serving.
2. Design candidate generation: ANN over item embeddings, plus popularity and recency sources.
3. Design ranking: a lightweight model scoring a few hundred candidates per request.
4. Handle cold start with popularity and content-based fallbacks.
5. Add a serving cache keyed by user with short TTL for freshness.

## Architecture Sketch

```text
Offline: interactions -> [Training] -> item/user embeddings + ranking model
                          -> batch precompute top-N per user -> Recs store

Online:  request(userId) -> [Serving svc]
             1. candidates: ANN(user emb) UNION popular UNION recent  (few hundred)
             2. rank: model.score(user, candidate) -> top-K
             3. filter: already-seen, business rules
          -> cache(userId, TTL) -> return

GET /recommendations?userId&context -> 200 { items[], modelVersion }
POST /events { userId, itemId, action, ts } -> 204   // feeds fresh signals

UserEmb { userId -> vector[d] }             // partition by userId
ItemEmb { itemId -> vector[d] }             // ANN index, sharded
Recs    { userId -> [itemId, score]... }    // precomputed, cache with TTL
```

## Deep-Dive Topics

- **Candidate generation:** ANN search over embeddings; blending multiple candidate sources.
- **Cold start:** content-based features and popularity for users/items without history.
- **Trade-off 1 — batch-precomputed vs. real-time recs:** precomputing top-N per user makes serving a cache lookup but goes stale between runs and wastes compute on inactive users; real-time reranking is fresh but latency-critical. Justify precompute + real-time reranking of the head.
- **Trade-off 2 — model complexity vs. latency:** a richer ranking model improves quality but risks the 150 ms budget; keep candidate generation cheap and spend the budget on ranking a small set.

## Deliverables

- [ ] A design document (~3–5 pages) expanding the two-stage architecture above.
- [ ] Capacity estimates: serving QPS, embedding storage, precomputed-recs storage.
- [ ] A partitioning plan for embeddings and the recs store.
- [ ] A caching strategy for served recommendations with TTL and freshness policy.
- [ ] At least two trade-offs, each with the option chosen and why.

## Common Pitfalls

- Ignoring cold start, so new users get an empty or nonsensical list.
- Ranking over the full catalog per request instead of a bounded candidate set.
- Precomputing recs for every user daily, wasting compute on the inactive majority.
- Never filtering already-seen items, so the same recommendations repeat forever.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — batch vs. serving system design.
- [Google ML: Recommendation Systems course](https://developers.google.com/machine-learning/recommendation) — candidate generation and ranking.
- [YouTube recommendations paper (RecSys 2016)](https://research.google/pubs/deep-neural-networks-for-youtube-recommendations/) — the two-stage funnel in production.
- [Faiss: efficient similarity search](https://github.com/facebookresearch/faiss) — approximate nearest-neighbor for candidates.

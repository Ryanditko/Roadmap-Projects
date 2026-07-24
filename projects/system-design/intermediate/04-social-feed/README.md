# Design a Social Media Feed

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design the backend that produces the home feed for a network like Twitter/X or Instagram: users follow others, publish posts, and open the app to a ranked stream of recent activity. The defining tension is fan-out — when someone with millions of followers posts, do you push that post into millions of feeds immediately, or assemble each feed on read? The answer shapes the whole system. This is a design exercise: you produce a design document, capacity numbers, and diagrams — not working code.

## Prerequisites

- Understanding of the follow graph as a data model
- Familiarity with caching and precomputation strategies
- Awareness of message queues for asynchronous fan-out
- Comfort estimating read/write ratios and storage for timelines

## Learning Objectives

By the end, you should be able to:

- Compare fan-out-on-write vs. fan-out-on-read and pick per user class
- Estimate feed read QPS, post write QPS, and timeline cache storage
- Design a hybrid feed that handles both normal users and celebrities
- Partition the follow graph and timeline stores sensibly
- Justify trade-offs between write amplification and read latency

## Requirements & Constraints

- Assume 200M daily users, ~5k posts/s, each user opening the feed ~10×/day (~20k feed reads/s).
- Feed load must return in under ~200 ms; near-real-time freshness (seconds) is enough.
- Some accounts have tens of millions of followers (the "celebrity problem").
- Feeds show recent posts from followees, ranked (recency + engagement signals).
- Estimate write amplification for fan-out and storage for cached timelines.

## Suggested Approach

1. Compute the fan-out cost: avg followers × post rate = timeline writes/s.
2. Choose fan-out-on-write for most users; fall back to read-time merge for celebrities.
3. Design the timeline cache (per-user list of post IDs) and its trimming policy.
4. Add a ranking step that reorders the candidate set before returning.
5. Partition the graph and timelines by userId; plan for hot celebrity nodes.

## Architecture Sketch

```text
Post -> [Post svc] -> Post store (shard by postId)
                         |-> fan-out worker -> Queue -> write postId into follower timelines (cache)
                                                          (skip if author is a celebrity)

Open app -> [Feed svc] -> read own timeline cache
                            + merge recent posts from followed celebrities (read-time)
                            -> Ranking -> return page

POST /posts            { userId, text, media[] } -> 201 { postId }
GET  /feed?userId&cursor                          -> 200 { posts[], nextCursor }
POST /follow           { followerId, followeeId } -> 204

Follow   { followerId, followeeId, ts }        // partition by followerId
Timeline { userId -> [postId, ...] }           // per-user cache, trimmed to N
Post     { postId, authorId, text, ts, stats } // shard by postId
```

## Deep-Dive Topics

- **Fan-out strategy:** push (write) vs. pull (read) vs. hybrid by follower count threshold.
- **Ranking:** recency baseline plus engagement features; keep it cheap at read time.
- **Trade-off 1 — write vs. read amplification:** fan-out-on-write makes reads O(1) but a celebrity post triggers millions of writes; fan-out-on-read is cheap to write but slow to assemble. Justify a hybrid threshold.
- **Trade-off 2 — timeline storage:** caching full posts is fast but huge; cache only post IDs and hydrate from the post store on read.

## Deliverables

- [ ] A design document (~3–5 pages) expanding the architecture above.
- [ ] Capacity estimates: feed read QPS, post write QPS, fan-out writes/s, timeline cache storage.
- [ ] A partitioning plan for the follow graph and timelines, addressing celebrity hot nodes.
- [ ] A caching strategy for timelines with a trimming/eviction policy.
- [ ] At least two trade-offs, each with the option chosen and why.

## Common Pitfalls

- Pure fan-out-on-write with no celebrity exception, so one viral post stalls the pipeline.
- Caching full post bodies per follower, multiplying storage by follower count.
- Never trimming timelines, so inactive users accumulate unbounded history.
- Ranking synchronously over thousands of candidates, blowing the latency budget.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — fan-out and timeline design.
- [Twitter Engineering: the infrastructure behind Timelines](https://blog.twitter.com/engineering/en_us/topics/infrastructure) — real-world feed architecture.
- [Redis Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/) — a natural fit for ranked timelines.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — the Twitter fan-out case study.

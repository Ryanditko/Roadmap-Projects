# Design a Distributed Message Queue

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Design a distributed message queue like Apache Kafka or RabbitMQ: producers publish messages to topics, consumers read them at their own pace, and the system durably retains messages so a slow or restarted consumer loses nothing. The interesting problems are partitioning topics for parallelism, tracking each consumer's position (offset), replicating for durability, and deciding what delivery guarantee you can honestly promise. This is a design exercise: you produce a design document, capacity numbers, and diagrams — not working code.

## Prerequisites

- Understanding of the pub/sub and log-based messaging models
- Familiarity with replication and leader/follower concepts
- Awareness of at-most-once, at-least-once, and exactly-once semantics
- Comfort estimating throughput and retention storage

## Learning Objectives

By the end, you should be able to:

- Design topics, partitions, and consumer groups for parallel consumption
- Estimate write throughput, retention storage, and replication overhead
- Design offset management so consumers resume exactly where they left off
- Reason honestly about ordering and delivery guarantees
- Justify trade-offs between throughput and delivery-guarantee strength

## Requirements & Constraints

- Assume 1M messages/s ingest, avg 1 KB each, 7-day retention, replication factor 3.
- Ordering is guaranteed within a partition, not across partitions.
- Consumers may lag, restart, or scale out without losing or skipping messages.
- A broker failure must not lose acknowledged messages.
- Estimate write throughput in MB/s, total retention storage, and replicated storage.

## Suggested Approach

1. Compute ingest bandwidth (msgs/s × size) and retention storage (× days × replication).
2. Design the partitioned append-only log and how producers pick a partition.
3. Design consumer groups: each partition consumed by one member; offsets committed per group.
4. Design replication: leader per partition, followers replicate, in-sync replica set.
5. Choose a delivery guarantee and explain what the producer/consumer must do to achieve it.

## Architecture Sketch

```text
Producers -> [Broker cluster] -> Topic "orders"
                                    Partition 0 (leader B1, replicas B2,B3) append-only log
                                    Partition 1 (leader B2, replicas B1,B3)
                                    ...
Consumer group "billing":  P0 -> consumer A   P1 -> consumer B   (offsets committed per group)

PUB  topic=orders key=userId value=<bytes>   -> ack after replicated to ISR
SUB  group=billing topic=orders              -> stream from committed offset
COMMIT group=billing partition=0 offset=12345

Partition { topicId, partId, log[offset -> message], leaderBroker, replicaSet }
Offset    { groupId, topicId, partId, committedOffset }   // partition by (group, topic, part)
```

## Deep-Dive Topics

- **Partitioning & ordering:** key-based partition assignment; ordering only within a partition.
- **Offset management:** committed vs. current offset; effect on redelivery after crash.
- **Trade-off 1 — throughput vs. delivery guarantee:** at-least-once with async acks is fast but redelivers on failure; exactly-once needs idempotent producers plus transactional commits, costing latency. Justify at-least-once + idempotent consumers as the pragmatic default.
- **Trade-off 2 — replication acks:** waiting for all replicas maximizes durability but stalls on a slow follower; acking the in-sync set balances durability and latency.

## Deliverables

- [ ] A design document (~3–5 pages) expanding the architecture above.
- [ ] Capacity estimates: ingest MB/s, retention storage, replicated storage, partition count.
- [ ] A partitioning plan tying keys to partitions and consumers.
- [ ] A caching/buffering strategy (page cache, batching) and its effect on throughput.
- [ ] At least two trade-offs, each with the option chosen and why.

## Common Pitfalls

- Promising global ordering across partitions — it is impossible without one partition.
- Committing offsets before processing, turning at-least-once into silent message loss.
- Ignoring replication overhead in the storage estimate (factor of 3 is easy to forget).
- Too few partitions, capping consumer parallelism no matter how many consumers you add.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — async messaging patterns.
- [Kafka: The Log (Jay Kreps)](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying-abstraction) — the log abstraction behind modern queues.
- [Kafka documentation: design](https://kafka.apache.org/documentation/#design) — partitioning, replication, delivery semantics.
- [Martin Kleppmann, *Designing Data-Intensive Applications*](https://dataintensive.net/) — streams and delivery guarantees.

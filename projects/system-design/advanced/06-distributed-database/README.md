# Design a Distributed Database

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design a distributed database that provides ACID transactions across many nodes and datacenters. Unlike designing an application on top of a database, here the database *is* the system: you decide how data is partitioned, how replicas agree on writes via consensus, how transactions commit atomically across shards, and what happens when the network splits a cluster in two. This is where CAP stops being a slogan and becomes a concrete set of engineering choices. This is a design exercise: your deliverable is a written design document with diagrams and trade-off analysis, not running code.

## Prerequisites

- Firm grasp of ACID, isolation levels, and the CAP/PACELC theorems
- Understanding of consensus algorithms (Raft, Paxos) at least conceptually
- Familiarity with replication (leader/follower, quorum) and partitioning
- Exposure to distributed transaction protocols (two-phase commit)

## Learning Objectives

By the end, you should be able to:

- Choose a partitioning scheme (range vs hash) and justify it against the workload
- Design replication with a consensus group per shard and reason about quorum sizes
- Specify a cross-shard transaction protocol and its isolation guarantee
- Analyze behavior under network partition and place your system on the CAP spectrum
- Plan resharding, failure recovery, and multi-datacenter replication

## Requirements & Constraints

1. Provide serializable or snapshot-isolation transactions across shards.
2. Survive the loss of a minority of replicas in any shard without data loss.
3. Partition data to scale writes horizontally; support online resharding.
4. Bound commit latency; document the latency cost of cross-shard/cross-region commits.
5. State the CAP posture explicitly: on partition, favor consistency (reject writes) — CP.
6. Support multi-datacenter replication with a defined consistency/latency trade-off.
7. Provide backup, point-in-time recovery, and schema evolution without downtime.

## Suggested Approach

Begin by fixing the consistency target — say, serializable snapshot isolation — because it constrains everything else. Partition the keyspace (range partitioning enables efficient scans; hash spreads load evenly and avoids hotspots). Replicate each partition with a consensus group (Raft) so a majority quorum commits writes and tolerates a minority failure. For transactions spanning shards, layer two-phase commit over the per-shard consensus (the Spanner model), and use a time or timestamp-ordering mechanism for global ordering. Analyze the partition case: as CP, the minority side rejects writes to preserve consistency. Design resharding as a background split/merge that moves ranges without stopping the world.

## Architecture Sketch

```text
Client ──> Coordinator (per txn) ──> route by key ──> Shard groups
                                                        each shard = Raft group:
                                                          leader + followers (quorum commit)

Cross-shard txn:
  Coordinator: BEGIN -> acquire locks on participant shards
             -> PREPARE (each shard: durably vote via its Raft group)
             -> all YES? COMMIT : ABORT  (2PC over consensus)
  timestamp ordering gives global serializability

Partitioning: keyspace -> [range or hash] -> shards -> resharded by split/merge

Key APIs (SQL-like or KV):
BEGIN / COMMIT / ROLLBACK
PUT(key, value) / GET(key) @ snapshot_ts
scan(range) @ snapshot_ts

Replication (per shard):
Raft{ term, log[], commitIndex }  # majority quorum, minority may lag/fail
```

## Deep-Dive Topics

- **CAP/PACELC in practice:** why this design is CP; the availability cost on partition.
- **Consensus:** Raft leader election, log replication, quorum math, membership changes.
- **Isolation levels:** serializable vs snapshot isolation; how MVCC provides reads.
- **Cross-shard commit:** two-phase commit over consensus; the Spanner TrueTime idea.
- **Resharding & recovery:** online range splits, replica catch-up, backup/PITR.

## Deliverables

- [ ] A design document (~4–8 pages) with the partition/replication/transaction layers, refined.
- [ ] An explicit CAP/PACELC statement with the partition-behavior analysis.
- [ ] The chosen partitioning and replication strategy with quorum-size justification.
- [ ] The cross-shard transaction protocol and its isolation guarantee, spelled out.
- [ ] A failure/DR analysis: leader loss, minority partition, whole-datacenter loss, resharding mid-write.

## Common Pitfalls

- Claiming "CA" under CAP — a partition will happen; you must choose C or A, not both.
- Using plain two-phase commit without consensus, so a coordinator crash blocks forever.
- Range partitioning on a monotonically increasing key, creating a permanent write hotspot.
- Ignoring clock skew in timestamp ordering, breaking global serializability.
- Resharding that stops the world instead of moving ranges online.

## Resources

- [Spanner: Google's Globally-Distributed Database](https://research.google/pubs/pub39966/) — TrueTime and externally-consistent distributed transactions.
- [In Search of an Understandable Consensus Algorithm (Raft)](https://raft.github.io/raft.pdf) — the Raft paper.
- [Dynamo: Amazon's Highly Available Key-value Store](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — the AP counterpoint to Spanner's CP.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — replication, partitioning, transactions in one place.
- [Jepsen: consistency analyses](https://jepsen.io/analyses) — how real databases behave under partition.

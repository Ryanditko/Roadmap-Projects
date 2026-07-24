# Distributed ETL System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Design a distributed ETL job that reads a large dataset, transforms it across many worker nodes, and writes a partitioned result — while staying correct and cheap when a worker dies mid-run. The interesting problems here are not the transformations themselves but everything around them: how data is partitioned, why one hot key can stall a whole stage (data skew), how a shuffle moves gigabytes across the network, and how checkpointing lets you recover without redoing everything. You will treat the cluster as an unreliable machine — nodes vanish, disks fill, one partition is 100× the others — and build a job whose runtime and cost stay predictable anyway. The deliverable is a design and a working distributed job on a framework like Spark, plus a documented plan for skew, retries, and recovery.

## Prerequisites

- Working knowledge of a distributed framework (Apache Spark or Hadoop MapReduce)
- Understanding of partitioning, shuffles, and the map/reduce mental model
- Comfort reasoning about network and disk I/O as the dominant cost
- Familiarity with a columnar format (Parquet/ORC) and object storage

## Learning Objectives

By the end, you should be able to:

- Partition input and control parallelism to match cluster resources
- Diagnose and mitigate data skew (salting, broadcast joins, repartitioning)
- Explain what a shuffle does and how to minimize its cost
- Use checkpointing and idempotent writes so a failed run resumes safely
- Instrument a distributed job and read its stage/task metrics to find bottlenecks

## Functional Requirements

1. The job must process a dataset far larger than any single node's memory, in bounded parallelism.
2. Transformations must be deterministic and idempotent so a retried task produces identical output.
3. The job must detect and mitigate skew on at least one join or group-by key.
4. On worker failure, only the lost tasks must re-execute — not the entire job.
5. Output must be written partitioned (e.g. by date) to object storage, with atomic commit so partial writes are never read as complete.
6. The job must emit metrics for records in/out, shuffle bytes, and per-stage duration.

## Suggested Milestones

1. **Milestone 1 — Baseline job:** Read → transform → write partitioned output on a small cluster; confirm correctness on a known sample.
2. **Milestone 2 — Scale & skew:** Run against a large, skewed dataset; measure the straggler, then apply salting or a broadcast join and re-measure.
3. **Milestone 3 — Resilience:** Add checkpointing and atomic/idempotent writes; kill a worker mid-run and verify clean recovery.

## Data & Interface Sketch

```text
[source: object store]  raw/events/*.parquet   (billions of rows)
        │  read, partitions = f(input size, cores)
        ▼
[map stage]  parse, filter, derive columns        (no shuffle)
        │
        ▼
[shuffle]  repartition by joinKey  ── skew? salt hot keys: key -> key#rand(0..N)
        │
        ▼
[reduce stage]  join / aggregate                   (checkpoint here)
        │
        ▼
[sink]  write partitioned by dt=YYYY-MM-DD, atomic commit (_SUCCESS marker)
        out/agg/dt=2026-07-24/part-*.parquet

Non-functional targets: runtime < T for dataset size S, cost < $C,
recovery re-runs only failed tasks.
```

## Stretch Goals

- Add adaptive query execution (or manual equivalent) that re-partitions based on observed shuffle sizes.
- Support incremental runs that process only new partitions instead of the full dataset.
- Add a spot/preemptible instance pool and prove the job still completes when nodes are reclaimed.

## Definition of Done

- [ ] The job completes on a dataset larger than any node's RAM without OOM.
- [ ] A documented skew mitigation measurably shrinks the slowest task.
- [ ] Killing a worker mid-run reruns only lost tasks and yields identical output.
- [ ] Output is partitioned and only visible after atomic commit; a crashed write leaves no readable partial data.
- [ ] A benchmark records runtime, shuffle bytes, and cost at your target dataset size.

## Common Pitfalls

- Letting the framework pick a default partition count that is far too low or too high for your data.
- Fixing skew by adding memory instead of rebalancing keys — it postpones the failure, it doesn't remove it.
- Non-idempotent writes, so a retried task appends duplicates.
- Reading output before the atomic commit marker exists and treating a partial write as complete.

## Resources

- [Spark: Tuning & Performance](https://spark.apache.org/docs/latest/tuning.html) — memory, serialization, and partitioning guidance.
- [Spark SQL Performance Tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html) — broadcast joins and adaptive execution for skew.
- [MapReduce (paper)](https://research.google/pubs/pub62/) — the original model for fault-tolerant distributed processing.
- [Apache Parquet documentation](https://parquet.apache.org/docs/) — the columnar format and its partitioning story.

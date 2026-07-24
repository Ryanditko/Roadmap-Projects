# High-Throughput Streaming System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Data Engineering · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build and tune a streaming system to move millions of events per second, then measure it honestly. This is a performance-engineering project: the goal is not a new feature but a documented, reproducible benchmark and a set of tuning decisions that provably move the numbers. You will push against the fundamental tension between throughput and latency — bigger batches and compression raise throughput but add latency; smaller batches cut latency but waste per-message overhead — and you will find where your system falls over. Along the way you will handle backpressure (what happens when consumers can't keep up), pick a serialization format, tune partitioning and parallelism, and separate genuine bottlenecks from noise. The deliverable is a system plus a benchmark report showing throughput, p50/p99 latency, and the effect of each tuning knob.

## Prerequisites

- Experience with a streaming platform (Kafka, Pulsar, Redpanda) and its producer/consumer tuning knobs
- Comfort profiling and reading latency percentiles, not just averages
- Understanding of batching, compression, and serialization tradeoffs
- Familiarity with backpressure and flow control concepts

## Learning Objectives

By the end, you should be able to:

- Design a repeatable throughput/latency benchmark with a controlled load generator
- Tune batching, compression, and partition count and quantify each effect
- Handle backpressure so an overwhelmed consumer degrades gracefully instead of collapsing
- Choose a serialization format (Avro/Protobuf vs JSON) based on measured cost
- Identify the real bottleneck (CPU, network, GC, disk) instead of guessing

## Functional Requirements

1. A load generator must produce a controllable, sustained event rate for benchmarking.
2. The system must report throughput (events/s and bytes/s) and end-to-end latency percentiles (p50/p99).
3. Backpressure must be handled: when consumers lag, the system must throttle or buffer within bounds, never lose data silently or OOM.
4. At least three tuning knobs (e.g. batch size, compression, partition count) must be varied and their effects measured.
5. The benchmark must be reproducible: same config, same result within tolerance.
6. The system must sustain a target rate for a sustained window without unbounded lag growth.

## Suggested Milestones

1. **Milestone 1 — Baseline & harness:** Build the load generator and metrics collection; record an untuned baseline (throughput, p99).
2. **Milestone 2 — Tune:** Sweep batching, compression, serialization, and partition count; chart each knob's throughput/latency effect.
3. **Milestone 3 — Backpressure & limits:** Add flow control, then push past capacity to find and document the breaking point.

## Data & Interface Sketch

```text
[load gen] --rate R--> [producer]
   knobs: batch.size, linger.ms, compression{none|lz4|zstd}, serialization{json|proto}
              │
              ▼
        [log: P partitions]   throughput scales ~ with P (up to a point)
              │
              ▼
        [consumers, C instances]   consumer lag = latest offset - committed offset
              │
              ▼
        [sink]

backpressure: if lag > threshold -> throttle producer / grow buffer to bound B
              never: drop silently, or buffer unbounded -> OOM

report (per config):
  throughput_eps | throughput_MBps | p50_ms | p99_ms | max_lag | cpu% | gc_ms
tradeoff seen:  larger batch => higher throughput, higher p99 latency
```

## Stretch Goals

- Add auto-scaling of consumers driven by lag and show it holds latency under a spike.
- Compare zero-copy / no-compression vs zstd and quantify the CPU/network tradeoff.
- Profile and eliminate a GC-pause-driven p99 spike (tune heap or switch to off-heap buffers).

## Definition of Done

- [ ] The benchmark is reproducible and reports throughput plus p50/p99 latency.
- [ ] At least three tuning knobs are swept with charted, explained effects.
- [ ] Backpressure keeps the system bounded under overload — no silent loss, no OOM.
- [ ] The documented breaking point identifies the actual bottleneck (CPU/network/GC/disk).
- [ ] The throughput/latency tradeoff is demonstrated with data, not asserted.

## Common Pitfalls

- Reporting average latency, hiding the p99 tail where the real pain lives.
- Benchmarking with a load generator too weak to saturate the system — you measure the generator, not the pipeline.
- Adding partitions past the point of diminishing returns and paying coordination overhead for nothing.
- "Fixing" backpressure with an unbounded in-memory queue that just moves the crash later.

## Resources

- [Kafka: Producer & consumer configs](https://kafka.apache.org/documentation/#producerconfigs) — batching, linger, and compression knobs.
- [Confluent: Optimizing Kafka throughput vs latency](https://docs.confluent.io/cloud/current/client-apps/optimizing/throughput.html) — the core tradeoff, with concrete settings.
- [Flink: Back Pressure](https://nightlies.apache.org/flink/flink-docs-stable/docs/ops/monitoring/back_pressure/) — how a processor surfaces and handles overload.
- [Brendan Gregg: Latency percentiles & USE method](https://www.brendangregg.com/usemethod.html) — a rigorous approach to finding bottlenecks.

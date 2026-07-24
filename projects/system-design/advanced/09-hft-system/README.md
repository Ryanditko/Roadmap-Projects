# Design a High-Frequency Trading System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design the core of a high-frequency trading (HFT) system: the path that ingests a market-data feed, runs a strategy, passes pre-trade risk checks, and submits an order to an exchange — all in single-digit microseconds. At this end of the latency spectrum the usual system-design instincts invert. Garbage collection pauses, kernel network stacks, and even cache misses become first-order design concerns, and correctness under a strict risk budget matters as much as speed. This is a design exercise — the deliverable is a rigorous design document, not a working trading engine.

## Prerequisites

- Solid grasp of concurrency, lock-free data structures, and memory layout
- Understanding of networking internals (kernel bypass, NIC, TCP vs. UDP multicast)
- Familiarity with order book mechanics and exchange order types
- Comfort with tail-latency reasoning (p99/p99.9) and capacity estimation
- An intermediate distributed-systems background (a stepping stone: [Design a Distributed Cache](../../intermediate/) or similar)

## Learning Objectives

By the end, you should be able to:

- Perform rigorous capacity estimation: market-data message rate, order rate, and per-stage latency budget
- Design a wire-to-wire hot path that minimizes latency (kernel bypass, busy-polling, cache-friendly layout)
- Reason about the consistency vs. latency trade-off in pre-trade risk and position tracking
- Design failure handling and disaster recovery that never leaves orders in an unknown state
- Justify where you spend microseconds and where you accept them

## Requirements & Constraints

**Functional**

- Ingest a normalized market-data feed and maintain an in-memory order book.
- Run a strategy that emits order intents on book updates.
- Enforce pre-trade risk checks (position limits, max order size, price collars, kill switch).
- Submit, amend, and cancel orders on the exchange gateway; reconcile fills.
- Persist an immutable, ordered audit log of every decision for compliance.

**Non-functional**

- Wire-to-wire p99 latency budget of ~5–10 µs for the decision path; state the split per stage.
- Sustain peak market-data rates (design for a burst of millions of messages/sec).
- No unbounded GC pauses on the hot path; deterministic latency is the goal, not just low mean.
- Risk checks must be correct even under recovery — never exceed a position limit.
- Recover from process/host failure with a known, reconciled order state.

## Suggested Approach

1. **Estimate the budget.** Pick concrete numbers (e.g. 2M msgs/sec peak market data, 50k orders/sec, 8 µs wire-to-wire target). Allocate the microsecond budget across parse → book update → strategy → risk → send.
2. **Design the hot path.** Kernel-bypass NIC (DPDK/Solarflare), busy-poll instead of interrupts, pre-allocated ring buffers, single-writer sequenced pipeline. Keep the decision thread pinned to a core.
3. **Model the order book.** Cache-friendly price-level structure with O(1) top-of-book access.
4. **Design risk as a fast in-line stage.** Position state in a lock-free structure; decide how strong its consistency needs to be versus the latency cost.
5. **Design failover & DR.** Sequenced event log, warm standby, and a reconciliation protocol with the exchange on restart.

## Architecture Sketch

```text
Exchange multicast ─UDP─► Feed handler ─► Order book (in-mem, cache-aligned)
   (market data)          (kernel bypass)        │
                                                  ▼
                                          Strategy engine
                                                  │ order intent
                                                  ▼
                                          Pre-trade risk  ──reject──► drop + log
                                          (position, size, collar,
                                           kill switch)
                                                  │ approve
                                                  ▼
                                          Order gateway ─TCP/binary─► Exchange
                                                  │
   Sequencer ──► append-only event log ──► warm standby (replay on failover)

Order (event)
  seq:       uint64        (monotonic, single-writer)
  clientId:  uint64
  symbol:    uint32        (interned)
  side:      BUY | SELL
  price:     int64         (fixed-point ticks)
  qty:       uint32
  tsNanos:   uint64
  state:     NEW | ACK | PARTIAL | FILLED | CANCELED | REJECTED

Internal API (in-process, no network on hot path)
  onMarketData(msg)     -> book.apply(msg)
  strategy.evaluate()   -> Optional<OrderIntent>
  risk.check(intent)    -> APPROVE | REJECT(reason)
  gateway.submit(order) -> async ACK via sequencer
```

## Deep-Dive Topics

- **Latency budgeting:** Measuring and attributing wire-to-wire latency; why p99.9 and jitter matter more than the mean.
- **Kernel bypass & busy-polling:** Trade-off of burning a core for determinism vs. interrupt-driven I/O.
- **Consistency vs. latency in risk:** Strong in-line position checks vs. optimistic-with-reconciliation, and the failure modes of each.
- **Deterministic memory:** Object pools, arena allocation, and avoiding GC on the hot path.
- **Failure & DR:** Sequencer-based replication, exchange reconciliation, and kill-switch semantics on restart.

## Deliverables

- A design document (~4–6 pages) covering the hot path, risk, and recovery.
- A rigorous capacity estimation: message rates, order rates, and a per-stage microsecond latency budget with assumptions.
- The architecture diagram, event/data model, and internal API contract.
- An explicit consistency vs. latency trade-off analysis for the risk stage, with the rejected option and why.
- A failure/DR section: what happens on host loss, and how order state is reconciled on recovery.

## Common Pitfalls

- Optimizing mean latency while ignoring tail latency and jitter — HFT lives and dies on p99.9.
- Putting risk checks off the hot path "for speed," creating a window where a position limit can be breached.
- Assuming a language runtime's GC is negligible; an unplanned pause is a missed market or a runaway order.
- Designing recovery that replays orders without reconciling against the exchange, causing duplicate or lost orders.
- Neglecting the kill switch — every HFT design needs a fast, tested way to halt all trading.

## Resources

- [LMAX Disruptor technical paper](https://lmax-exchange.github.io/disruptor/disruptor.html) — the canonical low-latency single-writer ring-buffer design.
- [DPDK: Programmer's Guide](https://doc.dpdk.org/guides/prog_guide/) — kernel-bypass packet processing fundamentals.
- ["Latency Numbers Every Programmer Should Know"](https://gist.github.com/jboner/2841832) — grounding for the microsecond budget.
- [Nasdaq: How exchange matching engines work](https://www.nasdaq.com/articles/how-do-exchanges-match-orders) — order types and matching mechanics.
- [System Design Primer: Latency vs. throughput](https://github.com/donnemartin/system-design-primer#latency-vs-throughput) — trade-off framing.

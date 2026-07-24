# Design a Real-time Collaboration System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design a real-time collaborative editor in the style of Google Docs, where dozens of people edit the same document simultaneously and every keystroke converges to a single consistent state on all screens — with cursors, presence, and offline edits that reconcile on reconnect. The heart of the problem is concurrency control on shared mutable data: two people typing at the same position must merge deterministically without a lock and without losing either edit. This forces a choice between two families of algorithms, Operational Transformation and CRDTs. This is a design exercise: your deliverable is a written design document with diagrams and trade-off analysis, not running code.

## Prerequisites

- Understanding of eventual consistency and the concept of convergence
- Familiarity with real-time transport (WebSockets) and pub/sub
- Conceptual grasp of Operational Transformation (OT) and CRDTs
- Awareness of the challenges of offline-first synchronization

## Learning Objectives

By the end, you should be able to:

- Compare OT and CRDT approaches and justify a choice for a text editor
- Design a convergence guarantee so all replicas reach the same state regardless of order
- Model presence, cursors, and selections as ephemeral (non-persisted) state
- Handle offline editing and reconciliation without losing or duplicating edits
- Reason about the cost of storing edit history and how to compact it

## Requirements & Constraints

1. All clients editing a document must converge to identical content.
2. Concurrent edits at the same position must merge deterministically, no lock, no data loss.
3. Reflect remote edits with sub-200 ms perceived latency for co-located users.
4. Support offline editing that reconciles correctly on reconnect.
5. Maintain per-user presence and cursor positions as transient state.
6. Preserve a version history and allow restoring prior versions.
7. Scale to documents with many concurrent editors and large edit histories.

## Suggested Approach

Pick the concurrency model first, because everything else follows. **OT** transforms each operation against concurrent ones and traditionally relies on a central server to order operations — simpler storage, but transformation functions are notoriously tricky. **CRDTs** attach identity to each character/element so merges are commutative and need no central authority — richer metadata per character, more storage, but robust offline. Route edits through a per-document server (or shard) that broadcasts to subscribers via pub/sub. Keep presence/cursors in a separate ephemeral channel that is never persisted. For offline, buffer local ops and replay/merge on reconnect. Manage history with periodic snapshots plus an operation log, compacting old operations behind a snapshot.

## Architecture Sketch

```text
Client A ─op─┐
Client B ─op─┼─> Doc session server (per doc / sharded) ─> Ordering + OT/CRDT merge
Client C ─op─┘        │                                      │
                      ├─broadcast(op)─> subscribers          └─> Op log + periodic snapshots (store)
                      └─presence channel (ephemeral: cursors, selections) NOT persisted

Offline: client buffers ops -> reconnect -> send buffered -> server merges -> converge

Key APIs (over WebSocket):
JOIN   { docId, sinceVersion }     -> { snapshot, version }
OP     { docId, op, baseVersion }  -> ACK { version }  (+ broadcast to others)
PRESENCE { docId, cursorPos, selection }   # ephemeral, best-effort

Data model (sketch):
Doc{ id, snapshot, version, opLog[] }       # opLog compacted behind snapshots
Op{ type: INSERT|DELETE, pos/id, char, siteId, seq }  # CRDT: stable id per char
```

## Deep-Dive Topics

- **OT vs CRDT:** convergence guarantees, metadata cost, offline robustness, complexity.
- **Convergence proof:** why concurrent ops merge to the same state regardless of arrival order.
- **Presence as ephemeral state:** separate channel, never persisted, best-effort delivery.
- **Offline reconciliation:** op buffering, causal ordering, tombstones for deletes.
- **History compaction:** snapshots plus op log; garbage-collecting old operations.

## Deliverables

- [ ] A design document (~4–8 pages) with the merge model and session architecture, refined.
- [ ] A reasoned OT-vs-CRDT choice with the convergence guarantee stated.
- [ ] The offline-editing reconciliation flow specified end to end.
- [ ] A failure/DR analysis: session-server crash, split-brain edits, op-log corruption.
- [ ] A history/storage plan showing snapshot cadence and op-log compaction.

## Common Pitfalls

- Using last-write-wins on the whole document, silently discarding concurrent edits.
- Persisting presence/cursor data, bloating storage with transient state.
- Assuming operations arrive in order; the merge must be correct for any arrival order.
- Forgetting tombstones for deletes in a CRDT, so a delete and a concurrent insert diverge.
- Keeping an unbounded op log with no snapshots, making document load slower over time.

## Resources

- [Conflict-free Replicated Data Types (CRDT paper)](https://inria.hal.science/inria-00609399/document) — the formal foundation of CRDTs.
- [Operational Transformation (Wikipedia)](https://en.wikipedia.org/wiki/Operational_transformation) — the OT model and its history in collaborative editing.
- [Yjs](https://docs.yjs.dev/) — a widely-used CRDT framework worth studying for real-world design.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — real-time and pub/sub patterns.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — convergence and conflict resolution.

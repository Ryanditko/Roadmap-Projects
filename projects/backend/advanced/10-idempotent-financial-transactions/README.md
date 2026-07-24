# Idempotent Financial Transaction System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

In finance, "the request timed out" is the scariest sentence there is: did the transfer happen or not? The client will retry, and if your system processes that retry as a second transfer, you have just moved someone's money twice. This project has you build a transaction system where **exactly-once** semantics hold even though the network guarantees only at-least-once. The mechanism is the idempotency key: the client stamps each logical operation with a unique key, and the server promises that replaying the same key returns the original result without repeating the side effect. Around that you will build a transaction state machine, double-entry bookkeeping that never lets money appear or vanish, reversals for the cases that must be undone, and an audit trail an accountant would trust.

## Prerequisites

- Solid experience building transactional REST APIs backed by a relational database
- Firm grasp of database transactions, isolation levels, and unique constraints
- Understanding of concurrency hazards (race conditions, lost updates, double-submit)
- Familiarity with retry behavior in clients and message queues
- A backend stack of your choice plus a datastore with real transactions (Postgres, MySQL, etc.)

## Learning Objectives

By the end, you should be able to:

- Design an idempotency-key protocol that makes retries safe end-to-end
- Model a transaction as a state machine with legal, auditable transitions
- Implement double-entry bookkeeping so debits and credits always balance
- Handle concurrent duplicate requests deterministically under real isolation levels
- Implement reversals and reconciliation without breaking the ledger's invariants
- Build an immutable audit trail that can reconstruct any balance at any point in time

## Functional Requirements

1. Every mutating operation must accept a client-supplied idempotency key; replaying the same key must return the original result and never repeat the side effect.
2. A transaction must move through an explicit state machine (e.g. `pending → processing → settled | failed | reversed`) with only legal transitions allowed.
3. The ledger must use double-entry bookkeeping: every transaction posts balanced debit and credit entries, and total balances must always reconcile to zero.
4. Concurrent requests with the same idempotency key must be serialized so exactly one performs the effect (enforced by a unique constraint or lock, not just app logic).
5. The system must support reversals that post compensating entries rather than deleting or mutating original records.
6. An immutable, append-only audit log must record every state transition and posting with actor, timestamp, and reason.
7. A reconciliation routine must verify that account balances equal the sum of their ledger entries and flag any drift.
8. **Consistency:** money must never be created or destroyed; every operation is atomic and the ledger invariant holds after any crash or retry.
9. **Reliability:** an operation interrupted mid-flight (crash between charge and settle) must be recoverable to a correct terminal state, not left ambiguous.

## Suggested Milestones

1. **Milestone 1 — Ledger & double-entry:** Model accounts and balanced entries; enforce that every transaction balances atomically.
2. **Milestone 2 — Idempotency keys:** Add the key protocol; store key → result and make replays return the stored outcome.
3. **Milestone 3 — State machine & concurrency:** Implement legal transitions and prove concurrent same-key requests produce exactly one effect.
4. **Milestone 4 — Reversals & audit:** Add compensating reversals and an append-only audit trail.
5. **Milestone 5 — Reconciliation & recovery:** Reconcile balances against entries and recover interrupted transactions to a terminal state.

## Data & Interface Sketch

```text
Account            Ledger Entry (double-entry)
  id, balance        id, txId, accountId, direction(debit|credit), amount

Transaction
  id, idempotencyKey (UNIQUE), state, amount, from, to, createdAt

Idempotency record
  key (UNIQUE), requestHash, responseSnapshot, status(in_progress|done)

State machine
  pending -> processing -> settled
                        \-> failed
  settled  -> reversed   (posts compensating entries; original untouched)

POST /transfers
  Header: Idempotency-Key: <uuid>
  body { from, to, amount }
  -> 201 { txId, state }        (first time)
  -> 200 { txId, state }        (replay: same result, no new effect)
  -> 409                        (same key, different request body)

Invariant checked on every commit:  Σ debits == Σ credits
```

## Stretch Goals

- Add a settlement window that batches pending transactions and settles them together.
- Support multi-currency accounts with explicit FX entries that still balance.
- Add basic fraud/velocity checks that can hold a transaction in `pending` for review.
- Expose a point-in-time balance query by replaying the ledger up to a timestamp.

## Definition of Done

- [ ] Replaying any request with the same idempotency key returns the original result and causes no second effect.
- [ ] Concurrent same-key requests result in exactly one posted transaction, enforced at the datastore.
- [ ] Every transaction posts balanced entries; the ledger reconciles to zero at all times.
- [ ] Only legal state transitions are permitted; illegal ones are rejected.
- [ ] Reversals post compensating entries and never mutate or delete originals.
- [ ] A crash mid-transaction recovers to a correct terminal state with no ambiguous or duplicated money.

## Common Pitfalls

- Enforcing idempotency only in application code — without a unique constraint, two concurrent requests both pass the check and double-post.
- Treating a timeout as a failure and letting the client retry into a duplicate, instead of making the retry safe.
- Mutating or deleting ledger entries to "fix" a mistake instead of posting a reversal, destroying the audit trail.
- Storing balances as the source of truth instead of deriving them from entries, so drift becomes unrecoverable.
- Returning a cached idempotent response for a *different* request body under the same key — detect the mismatch and reject.
- Ignoring isolation levels, so a lost update lets two transfers read the same balance and overdraw.

## Resources

- [Stripe: Designing robust and predictable APIs with idempotency](https://stripe.com/blog/idempotency) — the reference on idempotency keys in a payments API.
- [Martin Fowler: Accounting Patterns (Ledger)](https://martinfowler.com/eaaDev/AccountingNarrative.html) — double-entry modeling for software.
- [PostgreSQL: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html) — what each isolation level actually prevents.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — chapters on transactions, isolation, and consistency.
- [Pat Helland: Life beyond Distributed Transactions](https://queue.acm.org/detail.cfm?id=3025012) — idempotence and at-least-once at scale.

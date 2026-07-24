# Distributed Order Processing System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

An order in a real e-commerce system rarely touches a single database. Placing one debits inventory, charges a card, and schedules a shipment — three separate services, three separate data stores, no shared transaction to bind them. This project asks you to make that flow *correct* anyway: when the payment succeeds but shipping has no capacity, the reserved stock must be released and the charge refunded. You will build the flow as a **saga** — a sequence of local transactions where each step has a compensating action that undoes it. Along the way you meet the hard truths of distributed systems: there is no global rollback, consistency is eventual, and every network call can fail, time out, or silently succeed twice.

## Prerequisites

- Solid REST/HTTP service experience ([API Gateway](../../intermediate/09-api-gateway/) and [Job Queue with RabbitMQ](../../intermediate/04-job-queue-rabbitmq/) are strong warm-ups)
- Comfort running several services locally (Docker Compose or equivalent)
- Familiarity with a message broker or durable queue
- A working mental model of database transactions (ACID) and why they don't cross service boundaries
- Any backend stack you like (Node, Go, Java, Python, C#)

## Learning Objectives

By the end, you should be able to:

- Model a multi-service business flow as a saga with explicit compensations
- Choose between orchestration (a central coordinator) and choreography (services reacting to events) and justify the trade-off
- Make each step idempotent so retries and duplicate messages are safe
- Reason about eventual consistency and expose intermediate states honestly to clients
- Trace a single order across service boundaries and recover from partial failures

## Functional Requirements

1. The system must accept an order and coordinate inventory reservation, payment, and shipping across at least three services.
2. Each step must be a local transaction with a defined compensating action (release stock, refund payment, cancel shipment).
3. A failure at any step must trigger compensations for all previously completed steps, in reverse order.
4. Every operation must be idempotent: replaying the same command or event must not double-charge or double-reserve.
5. The order must expose an observable state machine (e.g. `pending → confirmed → failed → compensating → cancelled`).
6. Failed or unprocessable messages must land in a dead-letter queue rather than being lost or retried forever.
7. **Reliability:** the flow must survive a service crash mid-saga and resume or compensate on restart — no orphaned reservations.
8. **Consistency:** the design must be eventually consistent; document which windows of inconsistency are acceptable and why.
9. **Observability:** a single correlation ID must let you trace one order end-to-end across all services.

## Suggested Milestones

1. **Milestone 1 — The happy path:** Three services, synchronous or event-driven, completing a successful order. No failures yet.
2. **Milestone 2 — Compensation:** Force a failure at each step and implement the compensating transactions so the system self-heals.
3. **Milestone 3 — Idempotency & durability:** Add idempotency keys, a durable saga log, and dead-letter handling; kill a service mid-flow and confirm it recovers.
4. **Milestone 4 — Observability:** Add correlation IDs and distributed tracing so you can watch an order move through the system.

## Data & Interface Sketch

```text
                 ┌─────────────┐
   POST /orders  │ Order/Saga  │  persists saga state + step log
 ───────────────▶│ Orchestrator│
                 └──────┬──────┘
        reserve │ charge │ ship        (each step: try + compensate)
        ┌───────▼──┐ ┌───▼────┐ ┌──────▼───────┐
        │Inventory │ │Payment │ │  Shipping    │
        └──────────┘ └────────┘ └──────────────┘

Saga step log entry
  orderId, stepName, status(pending|done|compensated), idempotencyKey

Order state: pending -> confirmed
                     \-> failed -> compensating -> cancelled
```

## Stretch Goals

- Implement the same flow **twice** — once orchestrated, once choreographed — and compare debuggability.
- Add a timeout per step so a stuck service triggers compensation instead of hanging forever.
- Introduce event sourcing for the order aggregate and rebuild state from the event log.
- Add a small admin view listing sagas currently stuck in `compensating`.

## Definition of Done

- [ ] A successful order transitions cleanly through all steps to `confirmed`.
- [ ] A forced failure at any step compensates all prior steps and ends in `cancelled`.
- [ ] Replaying any command or event produces no duplicate side effects.
- [ ] Killing a service mid-saga leaves no permanently reserved stock or unrefunded charge after recovery.
- [ ] A single correlation ID traces one order across every service log.

## Common Pitfalls

- Treating a saga like a database transaction and expecting an automatic rollback — there isn't one; you write every compensation by hand.
- Forgetting that "the network timed out" and "the operation failed" are different: the charge may have gone through. Idempotency keys are what save you.
- Compensating in the wrong order, or compensating a step that never actually completed.
- Hiding the intermediate `pending` state from clients and pretending the order is instant.

## Resources

- [Microservices.io: Saga pattern](https://microservices.io/patterns/data/saga.html) — orchestration vs. choreography, with worked examples.
- [Microsoft: Saga distributed transactions pattern](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga) — architecture-level walkthrough.
- [Hector Garcia-Molina & Kenneth Salem: "Sagas" (1987)](https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf) — the original paper.
- [Stripe: Designing robust and predictable APIs with idempotency](https://stripe.com/blog/idempotency) — how idempotency keys work in practice.
- [Martin Kleppmann: Designing Data-Intensive Applications](https://dataintensive.net/) — chapters on distributed transactions and consistency.

# Event-Driven Microservices Architecture

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

In a request-driven system, services call each other directly and every dependency is a potential point of coupling and failure. In an **event-driven** system, services announce facts — "OrderPlaced", "PaymentReceived", "UserRegistered" — to a shared log, and interested services react on their own schedule. The publisher doesn't know or care who consumes. This decoupling buys you independent deployment, natural buffering under load, and the ability to add new consumers without touching producers. It also hands you a fresh set of problems: events arrive out of order, schemas evolve, consumers crash mid-batch, and "what actually happened?" becomes a question you answer by replaying a log. This project has you build a small but honest event-driven system and confront each of those realities.

## Prerequisites

- Experience building standalone HTTP services ([Job Queue with RabbitMQ](../../intermediate/04-job-queue-rabbitmq/) and [Notification Service](../../intermediate/08-notification-service/) are good preparation)
- A message broker or event log you can run locally (Kafka, Redpanda, RabbitMQ, or NATS)
- Comfort with serialization formats (JSON, and ideally Avro or Protobuf)
- Basic understanding of at-least-once vs. exactly-once delivery semantics
- Any backend stack; two different languages across services is a bonus for proving decoupling

## Learning Objectives

By the end, you should be able to:

- Design events as immutable, self-describing facts and split reads from writes (CQRS)
- Version an event schema and evolve it without breaking existing consumers
- Build consumers that are idempotent and tolerant of duplicates and reordering
- Replay events to rebuild state or onboard a new service
- Contain failure with dead-letter queues and per-service observability

## Functional Requirements

1. At least three independent services must communicate exclusively through published events — no direct synchronous calls for the core flow.
2. Events must be immutable, carry a schema version, and be stored in an append-only log or broker.
3. A schema change must be introduced without breaking already-deployed consumers (additive/backward-compatible evolution).
4. Consumers must be idempotent: reprocessing an event must not corrupt state or duplicate side effects.
5. The system must support replay — a new or reset consumer can rebuild its state from the beginning of the log.
6. Poison messages must be routed to a dead-letter queue with enough context to diagnose them.
7. **Consistency:** the system is eventually consistent; document the read-your-writes windows and where staleness is visible.
8. **Availability:** a consumer being down must not block producers; events buffer and are processed on recovery.
9. **Observability:** each service must expose its consumer lag and processing metrics, and events must carry a correlation ID.

## Suggested Milestones

1. **Milestone 1 — Publish & subscribe:** One producer, one broker, two consumers reacting to a shared event stream.
2. **Milestone 2 — Schema & versioning:** Add a schema registry or version field; evolve an event and prove old consumers still work.
3. **Milestone 3 — Resilience:** Add idempotency, dead-letter handling, and consumer-group offsets; recover a crashed consumer.
4. **Milestone 4 — Replay & CQRS:** Rebuild a read model from the log and separate the write path from the query path.

## Data & Interface Sketch

```text
  ┌──────────┐   OrderPlaced    ┌───────────────────────────┐
  │ Producer │ ───────────────▶ │   Event Log / Broker       │
  └──────────┘                  │   (append-only, versioned) │
                                └───────┬─────────┬──────────┘
                              consumes  │         │  consumes
                                ┌───────▼──┐  ┌───▼────────┐
                                │ Billing  │  │ Read Model │──▶ query API
                                └──────────┘  └────────────┘
                                     │ poison
                                ┌────▼─────┐
                                │  DLQ     │
                                └──────────┘

Event envelope
  eventId, type, schemaVersion, occurredAt, correlationId, payload{...}
```

## Stretch Goals

- Introduce a schema registry (e.g. Confluent/Apicurio) and enforce compatibility on publish.
- Add the outbox pattern so a service's DB write and its event publish can't diverge.
- Implement event sourcing for one aggregate and derive multiple projections from the same stream.
- Add a saga/process manager coordinating a multi-service workflow purely through events.

## Definition of Done

- [ ] Core flow runs with zero direct service-to-service HTTP calls.
- [ ] An evolved event schema is consumed by both old and new consumer versions without error.
- [ ] Replaying the log rebuilds a consumer's state identically to its live-processed state.
- [ ] A downed consumer catches up from its last offset after restart with no lost events.
- [ ] Poison messages appear in the DLQ with correlation IDs, not silently dropped.

## Common Pitfalls

- Sneaking a synchronous call between services "just this once" — it reintroduces the coupling events were meant to remove.
- Making events commands in disguise ("DoThisNow") instead of facts ("ThisHappened").
- Breaking consumers with a non-additive schema change (renaming or removing a field) instead of evolving compatibly.
- Assuming exactly-once delivery; most brokers give at-least-once, so idempotency is mandatory, not optional.

## Resources

- [Martin Fowler: What do you mean by "Event-Driven"?](https://martinfowler.com/articles/201701-event-driven.html) — untangles the four distinct meanings of the term.
- [Confluent: Event sourcing, CQRS, and stream processing](https://www.confluent.io/learn/event-sourcing/) — patterns and their trade-offs.
- [Confluent: Schema evolution and compatibility](https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html) — how to change schemas safely.
- [Microservices.io: Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html) — reliably publishing events alongside DB writes.
- [Apache Kafka documentation](https://kafka.apache.org/documentation/) — logs, partitions, consumer groups, and offsets.

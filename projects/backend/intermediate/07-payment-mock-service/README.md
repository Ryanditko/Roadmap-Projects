# Payment Processing Mock Service

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a service that behaves like a payment gateway — Stripe or Adyen — without ever touching a real card network. Clients submit a charge, your service validates it, moves it through a transaction lifecycle (authorized → captured → settled, or declined), and later fires a webhook telling the merchant what happened. The money is fake, but the hard problems are real: making a retried request charge the customer exactly once, modeling a state machine that can never skip a step, and delivering webhooks reliably to a receiver that might be down. This is where you learn why idempotency keys exist and why payment code is written so defensively.

## Prerequisites

- Comfort building and versioning REST endpoints ([E-commerce API with JWT](../01-ecommerce-api-jwt/) pairs well as a consumer of this service)
- A relational database and understanding of transactions and unique constraints
- Familiarity with HTTP status semantics and request/response headers
- Basic grasp of finite state machines and why illegal transitions matter

## Learning Objectives

By the end, you should be able to:

- Implement idempotency keys so a duplicate request never double-charges
- Model a payment as an explicit state machine with guarded transitions
- Design refund logic that cannot exceed the captured amount
- Deliver webhooks reliably with signatures, retries, and at-least-once semantics
- Explain, at a conceptual level, why real systems never store raw card data (PCI-DSS scope)

## Functional Requirements

1. The system must accept a charge request carrying an `Idempotency-Key`; replaying the same key must return the original result, not create a new charge.
2. A charge must be validated (amount > 0, supported currency, well-formed card token) and rejected with a 4xx before any state is created.
3. Each payment must progress through a defined lifecycle; the system must reject any transition that is not legal from the current state.
4. The system must simulate declines deterministically (e.g. a magic test card token always fails) so clients can test failure paths.
5. A refund must never exceed the remaining refundable balance, and partial refunds must accumulate correctly.
6. On every state change the system must enqueue a webhook to the merchant's registered URL, signed so the receiver can verify authenticity.
7. Failed webhook deliveries must be retried with exponential backoff up to a bounded number of attempts.
8. Every state change must be recorded in an append-only audit log that is never mutated.

## Suggested Milestones

1. **Milestone 1 — Charge & idempotency:** Accept a charge, persist it, and make the `Idempotency-Key` return the stored result on replay.
2. **Milestone 2 — Lifecycle & refunds:** Implement the state machine, capture/settle transitions, deterministic declines, and bounded refunds.
3. **Milestone 3 — Webhooks & audit:** Sign and deliver webhooks with retry/backoff, and write an immutable audit trail for every transition.

## Data & Interface Sketch

```text
Payment
  id:               string
  idempotencyKey:   string (unique)
  amount:           integer (minor units, e.g. cents)
  currency:         string (ISO 4217, e.g. "USD")
  status:           enum { requires_capture, captured, settled, declined, refunded, partially_refunded }
  refundedAmount:   integer
  createdAt:        ISO-8601 string

Legal transitions:
  requires_capture -> captured -> settled
  requires_capture -> declined
  captured|settled -> partially_refunded -> refunded

POST /payments        headers: Idempotency-Key: <uuid>
                      body: { amount, currency, cardToken }
                      -> 201 { id, status } | 200 (replayed) | 402 declined
POST /payments/{id}/refund   body: { amount }
                      -> 200 { status, refundedAmount } | 422 over-refund
GET  /payments/{id}   -> 200 { ...payment } | 404

Webhook -> merchant URL
  headers: X-Signature: hmac-sha256(secret, body)
  body:    { event: "payment.captured", paymentId, status }
```

## Stretch Goals

- Add a 3D Secure simulation: some tokens require an extra confirmation step before capture.
- Implement a reconciliation report that sums captures minus refunds per day.
- Support multiple currencies with a fixed conversion table and store the rate used.
- Add a webhook replay endpoint so merchants can re-request past events by ID.

## Definition of Done

- [ ] Replaying a request with the same idempotency key returns the identical response and creates no second charge.
- [ ] Every illegal state transition is rejected; only defined paths succeed.
- [ ] Refunds are capped at the refundable balance and partial refunds sum exactly.
- [ ] Webhooks are signed, retried with backoff, and give up after a bounded number of attempts.
- [ ] The audit log is append-only and reflects every transition in order.

## Common Pitfalls

- Treating the idempotency key as advisory — check it inside the same transaction that inserts the charge, or a race creates two.
- Storing card numbers "just for the mock." Store an opaque token; keeping real PANs pulls you into PCI scope for no reason.
- Making declines random instead of deterministic, which makes client tests flaky.
- Firing webhooks synchronously inside the charge request, so a slow merchant blocks payments — enqueue and deliver out of band.
- Using floats for money. Integer minor units avoid rounding drift.

## Resources

- [Stripe: Idempotent requests](https://docs.stripe.com/api/idempotent_requests) — the canonical model for safe retries.
- [Stripe: Webhooks](https://docs.stripe.com/webhooks) — signing, retries, and event design.
- [PCI Security Standards Council](https://www.pcisecuritystandards.org/) — what card-data handling actually requires.
- [Martin Fowler: State Machine](https://martinfowler.com/dslCatalog/stateMachine.html) — modeling guarded transitions.
- [RFC 4122: UUIDs](https://datatracker.ietf.org/doc/html/rfc4122) — generating collision-free idempotency keys.

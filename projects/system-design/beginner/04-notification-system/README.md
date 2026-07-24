# Design a Notification System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design a system that delivers notifications across multiple channels — email, SMS, and push. The core insight is that the app producing a notification should not wait for a slow, unreliable third party (an email or SMS provider) to respond. A queue decouples the two: producers enqueue, workers dequeue and deliver, retrying on failure. You'll reason about reliability, avoiding duplicate sends, and respecting user preferences. Deliver a design document that explains the queue-based flow, the data model, and how delivery survives provider failures.

## Prerequisites

- Understanding of what a message queue does and why it decouples systems
- Awareness that external providers (email/SMS) are slow and sometimes fail
- Familiarity with the idea of a retry and a worker process
- Comfort reasoning about idempotency at a high level

## Learning Objectives

By the end, you should be able to:

- Design an asynchronous, queue-based delivery pipeline
- Reason about retries, backoff, and idempotency to avoid duplicate sends
- Model multiple delivery channels behind one interface
- Incorporate user preferences and opt-outs
- State a trade-off between delivery speed and reliability

## Requirements & Constraints

1. Accept a notification request and deliver it via one or more channels.
2. Support at least email, SMS, and push notifications.
3. Retry failed deliveries without sending duplicates to the user.
4. Respect user preferences (channel choice, opt-out, quiet hours).
5. The request-accepting API must stay fast even when a provider is slow.
6. Estimate scale: 5M notifications/day across channels — roughly 58/s average with peaks.

## Suggested Approach

1. Split the system into an ingestion API, a queue, and channel workers.
2. Do the math: 5M/day ≈ 58/s average; assume peaks of 5–10× during campaigns.
3. Design the enqueue path so accepting a request never blocks on a provider.
4. Design channel workers that pull from the queue and call the right provider.
5. Add an idempotency key and delivery-status tracking so retries don't double-send.

## Architecture Sketch

```text
Producer ── POST /notify ──> [ Ingestion API ] ──> [ Queue ]
                                                      │
                    ┌─────────────────────────────────┼──────────────────┐
              [ Email Worker ]                 [ SMS Worker ]       [ Push Worker ]
                    │                                │                    │
              email provider                   SMS provider         push provider
                    └──────────> [ Delivery Log ] <──────────────────────┘

Core API
  POST /notify  { userId, template, channels, data, idempotencyKey }
                -> 202 { notificationId }
  GET  /notify/{id}                       -> { status per channel }

Data model
  notifications: id (PK) | user_id | template | payload | idempotency_key | created_at
  deliveries:    notification_id | channel | status | attempts | last_attempt_at
  preferences:   user_id | channel | enabled | quiet_hours
```

## Deep-Dive Topics

- **Idempotency:** how an idempotency key plus a status check prevents duplicate sends on retry.
- **Retry strategy:** exponential backoff, max attempts, and routing exhausted messages to a dead-letter queue.
- **Prioritization:** separate queues for transactional (OTP) vs. bulk (marketing) notifications.

## Deliverables

- An architecture diagram showing ingestion, queue, and per-channel workers.
- The core API contract for submitting and checking a notification.
- A data model for notifications, deliveries, and preferences.
- One trade-off written up: e.g., synchronous send (simple, immediate feedback, fragile under provider latency) vs. queue-based async (resilient, scalable, but eventual and more moving parts).

## Common Pitfalls

- Calling the email/SMS provider synchronously from the request path, so provider latency becomes your latency.
- Retrying without idempotency, so a transient failure causes the user to get three copies.
- Ignoring user preferences and quiet hours, generating spam and opt-out complaints.
- Having no dead-letter path, so permanently failing messages retry forever.

## Resources

- [System Design Primer: Asynchronism](https://github.com/donnemartin/system-design-primer#asynchronism) — queues and workers explained.
- [AWS SQS: Dead-letter queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html) — handling messages that can't be delivered.
- [Stripe: Designing robust and predictable APIs with idempotency](https://stripe.com/blog/idempotency) — idempotency keys in practice.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — backoff and load-shedding patterns.

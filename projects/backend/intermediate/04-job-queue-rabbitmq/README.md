# Job Queue System (RabbitMQ)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build the machinery that lets an API say "do this later" and get on with its life. A client submits a job — resize an image, send a report, charge a card — and your service accepts it instantly, hands it to a message broker, and returns a tracking ID. Separate worker processes pull jobs off the queue and run them, at their own pace, on their own machines. The interesting work is everything that surrounds the happy path: retrying a job that failed for a transient reason, giving up gracefully on one that never will, tracking status so a caller can poll, and making sure a worker crash never silently loses a job. RabbitMQ gives you the primitives — acknowledgements, dead-letter exchanges, prefetch — and your design decides whether the system is reliable or leaky.

## Prerequisites

- Comfort building an HTTP API and running a long-lived background process
- Understanding of why you decouple accepting work from doing work
- A running RabbitMQ instance (Docker is the easiest path) and a client library for your language
- Familiarity with JSON or another serialization format for message payloads

## Learning Objectives

By the end, you should be able to:

- Model a producer/consumer pipeline over a durable message broker
- Use acknowledgements and prefetch so a crashed worker never loses in-flight work
- Implement bounded retries with exponential backoff and a dead-letter path
- Design jobs to be idempotent so a redelivered message does no harm
- Track and expose a job's status through its full lifecycle
- Shut a worker down gracefully without dropping the job it is mid-processing

## Functional Requirements

1. The API must accept a job with a type and payload, enqueue it durably, and return a tracking ID immediately.
2. Workers must consume jobs asynchronously and acknowledge only after successful processing.
3. A job that fails transiently must be retried with exponential backoff up to a bounded attempt count.
4. A job that exhausts its retries must land in a dead-letter queue, never be silently dropped.
5. The system must persist and expose each job's status (queued, processing, succeeded, failed, dead-lettered).
6. Job processing must be idempotent, so a redelivered message does not double-execute side effects.
7. Workers must support graceful shutdown, finishing or requeuing in-flight jobs before exiting.
8. The queue must survive a broker restart (durable queues and persistent messages).

## Suggested Milestones

1. **Milestone 1 — Submit & process:** Accept jobs over HTTP, publish to a durable queue, and have a worker consume and ack them.
2. **Milestone 2 — Retry & dead-letter:** Add bounded retries with backoff and route exhausted jobs to a dead-letter queue.
3. **Milestone 3 — Status & lifecycle:** Persist job status, expose a status endpoint, and add idempotency plus graceful shutdown.

## Data & Interface Sketch

```text
Job
  id:         string
  type:       string   (e.g. "resize-image")
  payload:    object
  status:     enum { queued, processing, succeeded, failed, dead }
  attempts:   integer
  maxRetries: integer
  createdAt:  ISO-8601 string

POST /jobs          body: { type, payload }
                    -> 202 { id, status: "queued" }
GET  /jobs/{id}     -> 200 { status, attempts, ... } | 404

Broker topology
  jobs.exchange -> jobs.queue        (workers consume, manual ack)
  on nack/expire -> jobs.dlx -> jobs.deadletter.queue

Retry: republish with delay = base * 2^(attempt-1) (+ jitter), until maxRetries
```

## Stretch Goals

- Add priority levels so urgent jobs jump ahead of a backlog of low-priority ones.
- Support delayed/scheduled jobs that only become available at a future time.
- Add worker heartbeats and a dashboard showing queue depth and worker health.
- Implement job dependencies, where one job only runs after another succeeds.

## Definition of Done

- [ ] Submitting a job returns immediately with a tracking ID; work happens in a worker.
- [ ] A worker crash mid-job causes the message to be redelivered, not lost (verified by killing a worker).
- [ ] Transient failures retry with backoff; exhausted jobs are visible in the dead-letter queue.
- [ ] Job status is queryable and reflects the true lifecycle.
- [ ] Redelivering the same job does not duplicate its side effects.

## Common Pitfalls

- Auto-acking messages on receipt, so a crash between receiving and finishing loses the job.
- Non-durable queues or non-persistent messages, so a broker restart wipes the backlog.
- Retrying non-transient failures forever instead of dead-lettering them after a cap.
- Unbounded prefetch, so one worker grabs the whole queue and starves the others.
- Assuming exactly-once delivery — RabbitMQ is at-least-once, so jobs must be idempotent.

## Resources

- [RabbitMQ: Tutorials](https://www.rabbitmq.com/tutorials) — work queues, acknowledgements, and routing from the source.
- [RabbitMQ: Dead Letter Exchanges](https://www.rabbitmq.com/docs/dlx) — the standard dead-letter pattern.
- [RabbitMQ: Consumer Prefetch](https://www.rabbitmq.com/docs/consumer-prefetch) — fair dispatch and avoiding worker starvation.
- [AWS: Error retries and exponential backoff](https://docs.aws.amazon.com/general/latest/gr/api-retries.html) — backoff and jitter done right.
- [Enterprise Integration Patterns: Dead Letter Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/DeadLetterChannel.html) — the concept behind the queue.

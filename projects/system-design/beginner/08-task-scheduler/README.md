# Design a Task Scheduler

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design a system that runs tasks — some on a schedule (every night at 2am), some once at a future time, some as background jobs enqueued by other services. The interesting problems are making sure a task runs even if a worker crashes mid-execution, and making sure it runs exactly once rather than twice when workers race. You'll reason about a durable task store, worker coordination, retries, and visibility timeouts. Deliver a design document covering the scheduling model, worker flow, and reliability guarantees.

## Prerequisites

- Understanding of the difference between a scheduled job and an on-demand job
- Awareness that a worker can crash while holding a task
- Familiarity with queues and the idea of "claiming" work
- Comfort reasoning about at-least-once vs. exactly-once execution

## Learning Objectives

By the end, you should be able to:

- Design a durable task store that survives worker crashes
- Reason about how workers claim tasks without two running the same one
- Handle retries and failed tasks with backoff and a dead-letter path
- Support both cron-style recurring and one-shot delayed tasks
- State a trade-off between exactly-once and at-least-once execution

## Requirements & Constraints

1. Schedule a task to run once at a future time or on a recurring cron schedule.
2. A task must run even if the worker that picked it up crashes.
3. Avoid running the same task twice when multiple workers compete.
4. Retry failed tasks with backoff; route permanently failing tasks aside.
5. Expose task status (pending, running, succeeded, failed).
6. Estimate scale: 1M tasks/day, up to 200 concurrent workers.

## Suggested Approach

1. Separate the scheduler (decides what's due) from the workers (execute tasks).
2. Do the math: 1M/day ≈ 12 tasks/s average; design for peaks when many cron jobs fire at the same minute (e.g., midnight).
3. Design task claiming: a worker atomically marks a task "running" with a visibility timeout so a crash releases it back.
4. Design retries with backoff and a max-attempts cap into a dead-letter store.
5. Explain how a due-time index lets the scheduler find ready tasks cheaply.

## Architecture Sketch

```text
[ Scheduler ] --scan due tasks--> [ Task Store ] <--claim/ack-- [ Worker pool ]
   (cron + one-shot)                    │                          (N workers)
                                   status updates
                                        │
                          exhausted -> [ Dead-letter Store ]

Core API
  POST /tasks   { runAt | cron, type, payload }  -> 201 { taskId }
  GET  /tasks/{id}                                -> { status, attempts, lastError }
  DELETE /tasks/{id}                              -> 204  (cancel)

Data model
  tasks: task_id (PK) | type | payload | run_at | cron | status
         | attempts | locked_by | lock_expires_at
  index on (status, run_at) to find due, unclaimed tasks
```

## Deep-Dive Topics

- **Visibility timeout:** how a lease with an expiry lets a crashed worker's task be re-claimed automatically.
- **Exactly-once vs. at-least-once:** why true exactly-once is hard, and how idempotent tasks make at-least-once safe.
- **Thundering herd at midnight:** spreading recurring tasks with jitter so they don't all fire simultaneously.

## Deliverables

- An architecture diagram showing scheduler, task store, worker pool, and dead-letter path.
- The core API contract for scheduling, checking, and cancelling tasks.
- A data model for tasks including locking fields.
- One trade-off written up: e.g., at-least-once with idempotent tasks (simple, robust, may double-execute) vs. attempting exactly-once (no duplicates, but complex coordination and still not perfect).

## Common Pitfalls

- Assuming a task runs to completion — designing without a crash-recovery path.
- Letting two workers claim the same task because the claim isn't atomic.
- Retrying non-idempotent tasks, so a retry causes a duplicate side effect (e.g., double charge).
- All cron jobs firing at exactly midnight and overwhelming the workers at once.

## Resources

- [System Design Primer: Asynchronism](https://github.com/donnemartin/system-design-primer#asynchronism) — task queues and workers.
- [AWS SQS: Visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html) — how leases enable crash recovery.
- [Crontab guru](https://crontab.guru/) — cron schedule syntax reference.
- [Google SRE Book: Distributed cron](https://sre.google/sre-book/chapters/) — running scheduled jobs reliably at scale.

# Email Sender Mock Service

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Beginner · **Estimated time:** 5–8 hours

## Overview

Build an email-sending API that never actually sends email — it validates each request, enqueues it, "processes" it against a mock transport, and records the outcome to a log or file. This models the transactional-email services (SendGrid, SES, Postmark) that sit behind sign-up confirmations and password resets, but without external dependencies or the risk of spamming real inboxes. The value is in the surrounding machinery: request validation, a job queue, status tracking, and retries.

## Prerequisites

- Ability to build HTTP endpoints that accept and return JSON ([Simple REST API for Task Management](../01-simple-rest-api-task-management/) recommended)
- Understanding of file or in-memory persistence ([Notes API](../04-notes-api-file-persistence/) helps)
- Familiarity with the idea of a queue and asynchronous processing
- Awareness of email address format rules

## Learning Objectives

By the end, you should be able to:

- Design an API that accepts a job and returns immediately with a tracking id
- Validate structured input (recipient, subject, body) and reject malformed requests
- Model a job lifecycle with explicit states and transitions
- Process a queue and simulate success and failure against a mock transport
- Implement retry logic with a bounded number of attempts
- Render a message from a template with substituted variables

## Functional Requirements

1. The system must accept an email request with recipient, subject, and body and return a job id with 202.
2. The system must reject an invalid recipient address or a missing field with 400.
3. Each job must have a status that moves through `queued → sending → sent | failed`.
4. The system must expose an endpoint to query a job's current status by id.
5. Instead of sending, the system must record each processed message to a file or log with a timestamp.
6. Failed jobs must be retried up to a configured maximum, then marked permanently failed.
7. Jobs and their statuses must persist so a restart does not lose queued or in-flight work.

## Suggested Milestones

1. **Milestone 1 — Accept & validate:** Take an email request, validate it, store it as `queued`, return a job id.
2. **Milestone 2 — Process the queue:** Move jobs through their states, "delivering" to a mock transport and logging the result.
3. **Milestone 3 — Reliability & templates:** Add bounded retries for failures and support templated message bodies.

## Data & Interface Sketch

```text
Job
  id:        string
  to:        string   (validated email)
  subject:   string
  body:      string
  status:    "queued" | "sending" | "sent" | "failed"
  attempts:  integer
  updatedAt: ISO-8601 string

POST /emails         body: { to, subject, body | { template, vars } }
                     -> 202 { id, status: "queued" } | 400
GET  /emails/{id}    -> 200 { id, status, attempts } | 404

Mock transport: randomly (or by rule) succeed/fail; log each attempt
Retry: on failure, requeue until attempts == MAX, then status=failed
```

## Stretch Goals

- Add named templates with variable interpolation and a template-not-found error.
- Add scheduled send: accept a `sendAt` timestamp and hold the job until then.
- Add batch send: accept many recipients in one request, tracked as individual jobs.
- Expose queue metrics (counts per status) on a stats endpoint.

## Definition of Done

- [ ] A valid request returns 202 with a job id and creates a `queued` job.
- [ ] Invalid recipients or missing fields return 400 before enqueuing.
- [ ] Job status is queryable and reflects real transitions through the lifecycle.
- [ ] Every processed message is recorded (logged/written), never actually emailed.
- [ ] A simulated failure retries up to the max, then settles as `failed` — no infinite loop.

## Common Pitfalls

- Doing the "send" synchronously inside the POST handler, defeating the point of a queue.
- Validating email with a naive check that rejects valid addresses or accepts obvious junk.
- Retrying forever on failure with no attempt cap, spinning the queue endlessly.
- Losing in-flight jobs on restart because status was only kept in memory.

## Resources

- [Wikipedia: Message queue](https://en.wikipedia.org/wiki/Message_queue) — the concept your queue is a tiny instance of.
- [RFC 5321: SMTP](https://datatracker.ietf.org/doc/html/rfc5321) — how real email delivery works, for context.
- [regex101](https://regex101.com/) — build and test an email-validation pattern interactively.
- [SendGrid API reference](https://www.twilio.com/docs/sendgrid/api-reference) — a real transactional-email API to model your interface on.

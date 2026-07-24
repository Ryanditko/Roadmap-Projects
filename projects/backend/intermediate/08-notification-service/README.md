# Notification Service (email + retry)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build the service every product eventually needs: a central place other systems call to say "notify this user," without caring how. Behind one API you fan out to multiple channels — email, SMS, push — each backed by a provider that can fail, rate-limit you, or go silent. Your job is to accept the request quickly, deliver asynchronously, retry the transient failures with backoff, track what actually landed, and respect the user's preferences and quiet hours. The interesting engineering is not sending one email; it is guaranteeing delivery is attempted reliably while never spamming a user twice for the same event.

## Prerequisites

- Comfort building REST endpoints and background workers
- Familiarity with a message queue or at least a durable job table ([Job Queue System (RabbitMQ)](../04-job-queue-rabbitmq/) is a strong companion)
- Understanding of async processing and why you decouple accept-from-deliver
- Access to a sandbox email provider (or a local SMTP catcher like MailHog) for testing

## Learning Objectives

By the end, you should be able to:

- Design a channel-agnostic API that hides provider details behind a common interface
- Implement a durable send pipeline with exponential backoff and a dead-letter path
- Track per-notification delivery status through its full lifecycle
- Render templates with variable substitution and per-locale content
- Honor user preferences, opt-outs, and do-not-disturb windows before sending

## Functional Requirements

1. The system must accept a notification request naming a recipient, an event/template, and payload variables, and return immediately with a tracking ID.
2. Delivery must happen asynchronously via workers, never inline with the accept request.
3. The system must support at least two channels behind a common send interface, chosen per request or per user preference.
4. A failed send must be retried with exponential backoff up to a bounded attempt count, then moved to a dead-letter store.
5. The system must persist and expose each notification's delivery status (queued, sent, delivered, failed).
6. Templates must support variable interpolation and be validated so a missing variable fails fast, not mid-send.
7. The system must check user preferences and do-not-disturb periods, suppressing or deferring sends that violate them.
8. The system must be idempotent per (recipient, event key) so a duplicated request does not notify twice.

## Suggested Milestones

1. **Milestone 1 — Accept & deliver:** Accept requests, enqueue them, and have a worker deliver email through one provider with status tracking.
2. **Milestone 2 — Retry & channels:** Add exponential backoff, a dead-letter store, and a second channel behind the shared interface.
3. **Milestone 3 — Templates & preferences:** Add template rendering, user preferences, do-not-disturb, and per-event idempotency.

## Data & Interface Sketch

```text
Notification
  id:            string
  recipientId:   string
  channel:       enum { email, sms, push }
  templateId:    string
  status:        enum { queued, sending, sent, delivered, failed, suppressed }
  attempts:      integer
  eventKey:      string (for idempotency)
  createdAt:     ISO-8601 string

Preferences
  userId:        string
  channels:      { email: true, sms: false, push: true }
  quietHours:    { start: "22:00", end: "07:00", tz: "America/Sao_Paulo" }

POST /notifications   body: { recipientId, templateId, channel?, vars, eventKey }
                      -> 202 { id, status: "queued" }
GET  /notifications/{id}  -> 200 { status, attempts, ... } | 404

Retry schedule (backoff): attempt n -> delay = base * 2^(n-1) (+ jitter)
After max attempts -> dead-letter store
```

## Stretch Goals

- Add fallback channels: if push fails permanently, automatically try email.
- Support batch sends that expand one request into many recipients efficiently.
- Add per-channel rate limiting so you never exceed a provider's throughput cap.
- Generate delivery analytics: sent/delivered/failed rates per channel over time.

## Definition of Done

- [ ] The accept endpoint returns immediately; delivery always happens in a worker.
- [ ] Transient failures retry with backoff and jitter, then dead-letter after the cap.
- [ ] Delivery status is queryable and reflects the true lifecycle of each notification.
- [ ] Templates reject missing variables before a send is attempted.
- [ ] Preferences and quiet hours suppress or defer sends correctly, and duplicates are collapsed by event key.

## Common Pitfalls

- Sending inside the HTTP request, so a slow provider makes your API slow and drops notifications on crash.
- Retrying without jitter, causing a thundering herd that hammers a recovering provider.
- Retrying non-transient failures (a hard bounce, an invalid number) forever instead of dead-lettering them.
- Treating "provider accepted" as "delivered" — accepted only means queued at the provider; use their delivery webhooks for the real status.
- Ignoring idempotency, so a client retry double-notifies the user.

## Resources

- [Twilio: Delivery status and callbacks](https://www.twilio.com/docs/messaging/guides/track-outbound-message-status) — how real delivery tracking works.
- [SendGrid: Event webhook](https://www.twilio.com/docs/sendgrid/for-developers/tracking-events/event) — sent vs delivered vs bounced.
- [AWS: Error retries and exponential backoff](https://docs.aws.amazon.com/general/latest/gr/api-retries.html) — backoff and jitter done right.
- [MailHog](https://github.com/mailhog/MailHog) — a local SMTP server for testing email without sending real mail.
- [RabbitMQ: Dead Letter Exchanges](https://www.rabbitmq.com/docs/dlx) — the standard dead-letter pattern.

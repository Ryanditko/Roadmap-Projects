# Design a WhatsApp-like Messaging Platform

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Advanced · **Estimated time:** 3–7 days

## Overview

Design a mobile-first messaging platform in the style of WhatsApp: billions of users exchange one-to-one and group messages that must arrive exactly once, in order, with delivery and read receipts — even when the recipient is offline for hours. The core challenge is maintaining hundreds of millions of persistent connections, routing each message to the right device fan-out, and doing it all under end-to-end encryption where the server never sees plaintext. This is a design exercise: your deliverable is a written design document with diagrams and trade-off analysis, not running code.

## Prerequisites

- Strong understanding of TCP, WebSockets, and long-lived connection management
- Familiarity with message queues and at-least-once vs exactly-once delivery
- Basic cryptography: symmetric/asymmetric keys, forward secrecy, the Signal protocol
- Understanding of push notification systems (APNs, FCM)

## Learning Objectives

By the end, you should be able to:

- Design a connection layer that holds hundreds of millions of concurrent sessions
- Guarantee ordered, exactly-once delivery on top of an at-least-once transport
- Model offline delivery with a durable per-recipient inbox and receipts
- Reason about end-to-end encryption and why the server stores only ciphertext
- Design group fan-out that scales without quadratic message multiplication

## Requirements & Constraints

1. Deliver messages with per-conversation ordering and exactly-once semantics.
2. Support offline recipients: store and forward when they reconnect.
3. Hold ~500M concurrent connections; assume tens of billions of messages/day.
4. Provide sent/delivered/read receipts with minimal extra round-trips.
5. Enforce end-to-end encryption; servers must never hold plaintext or long-term keys.
6. Support group chats up to ~1024 members without quadratic blowup.
7. Target 99.99% availability with sub-second delivery for online users.

## Suggested Approach

Estimate the connection fan-out first: concurrent sessions per gateway node determines fleet size and memory. Split the system into a stateless-ish **connection layer** (gateways holding sockets) and a **routing layer** that maps user→active gateway via a presence registry. Persist undelivered messages in a per-user inbox keyed for ordering, delete on ack. Assign monotonic per-conversation sequence numbers for ordering and dedup on the client. Keep E2E encryption client-side (Signal-style double ratchet): the server routes opaque ciphertext blobs. For groups, encrypt once per sender key and fan out references rather than re-encrypting per member.

## Architecture Sketch

```text
Phone A ──WSS──> Gateway (holds socket) ──> Router ──> Presence registry (user->gateway)
                                              │
                                              ├──> Message store (per-user inbox, durable)
                                              └──> Push (APNs/FCM) if offline

Send flow:
A encrypts (double ratchet) -> Gateway -> Router -> B online? deliver : enqueue inbox + push
B reconnects -> drain inbox (ordered) -> ack -> server deletes -> receipts back to A

Key APIs (over persistent socket, framed):
SEND    { convId, seq, ciphertext, ts }        -> ACK { convId, seq }
RECEIPT { convId, seq, type: DELIVERED|READ }
PRESENCE{ userId, state: ONLINE|OFFLINE, lastSeen }

Data model (sketch):
Inbox{ userId, [ {convId, seq, ciphertext, enqueuedAt} ] }  # deleted on ack
Session{ userId, deviceId, gatewayNode, prekeys[] }
Group{ id, members[], senderKeys{} }  # fan-out by reference
```

## Deep-Dive Topics

- **Connection layer scale:** epoll/event loops, per-node socket budgets, graceful gateway failover.
- **Ordering & dedup:** per-conversation sequence numbers, client-side idempotent apply.
- **Offline store-and-forward:** inbox durability, ack-then-delete, retention limits.
- **End-to-end encryption:** double ratchet, prekeys, multi-device key sync.
- **Group fan-out:** sender keys vs pairwise encryption; large-group amplification limits.

## Deliverables

- [ ] A design document (~4–8 pages) with the send/receive flow and inbox model, refined.
- [ ] Capacity estimates for concurrent connections, gateway fleet, and daily message volume.
- [ ] The ordering and exactly-once mechanism spelled out end to end.
- [ ] A failure/DR analysis: gateway crash, presence-registry loss, inbox partition failure.
- [ ] An E2E encryption section explaining what the server can and cannot see.

## Common Pitfalls

- Treating the server as trusted with plaintext — E2E means the server routes ciphertext only.
- Relying on transport ordering; add explicit per-conversation sequence numbers.
- Never deleting the inbox after ack, so storage grows without bound.
- Group fan-out that re-encrypts per member, turning one send into thousands of crypto ops.
- Losing the presence registry and having no fallback to reroute reconnecting users.

## Resources

- [The Signal Protocol](https://signal.org/docs/) — the double-ratchet E2E encryption WhatsApp is based on.
- [High Scalability: WhatsApp architecture](http://highscalability.com/blog/2014/2/26/the-whatsapp-architecture-facebook-bought-for-19-billion.html) — how a tiny team held millions of connections.
- [System Design Primer](https://github.com/donnemartin/system-design-primer) — messaging, queues, and connection scaling.
- [RFC 6455: The WebSocket Protocol](https://datatracker.ietf.org/doc/html/rfc6455) — the persistent-connection transport.
- [Designing Data-Intensive Applications](https://dataintensive.net/) — delivery guarantees and ordering.

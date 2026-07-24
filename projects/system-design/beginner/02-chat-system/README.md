# Design a Real-Time Chat System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** System Design · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Design a one-to-one and small-group chat system — think a stripped-down WhatsApp or Slack DM. The heart of the problem is that a message must travel from one connected client to another in near real time, which means HTTP request/response alone won't cut it. You'll reason about persistent connections, how messages get persisted and ordered, and how a user knows who is online. Deliver a design document that explains delivery, storage, and presence — not a working server.

## Prerequisites

- Understanding of the client/server model and TCP connections
- Awareness that WebSocket keeps a connection open, unlike plain HTTP
- Familiarity with queues and how a message can be buffered
- Comfort reading a sequence of interactions between components

## Learning Objectives

By the end, you should be able to:

- Choose a real-time transport (WebSocket vs. long polling) and justify it
- Design a message model that supports ordering and delivery status
- Explain how offline messages are stored and later delivered
- Track user presence and reason about its accuracy
- State a trade-off between delivery guarantees and complexity

## Requirements & Constraints

1. Two online users can exchange messages with sub-second delivery.
2. Messages sent while a recipient is offline are stored and delivered on reconnect.
3. Messages within a conversation must appear in a consistent order.
4. Show whether a contact is online or offline (presence).
5. Support at least 1-to-1 chat; group chat is a stretch.
6. Estimate scale: 100K daily active users, average 40 messages/user/day.

## Suggested Approach

1. Separate the transport (how bytes move) from persistence (how messages are stored).
2. Do the math: 100K × 40 = 4M messages/day ≈ 46 messages/s average, with peaks several times higher.
3. Decide how a client maintains a connection and which server it talks to.
4. Design message storage with a conversation key and a time-ordered sort key.
5. Add a presence mechanism (heartbeat + last-seen) and describe its staleness window.

## Architecture Sketch

```text
Client A <--WebSocket--> [ Gateway ] --> [ Message Service ] --> [ Message Store ]
Client B <--WebSocket--> [ Gateway ]           │
                                          [ Presence ]  (heartbeat, last_seen)
                          offline? enqueue -> [ Inbox / Queue ] -> deliver on reconnect

Core API / events
  WS send   { conversationId, body }        -> ack { messageId, ts }
  WS deliver{ messageId, from, body, ts }
  GET /conversations/{id}/messages?before=ts -> page of messages

Data model
  messages: conversation_id (PK) | ts (SK) | sender_id | body | status
  presence: user_id | last_seen | online
```

## Deep-Dive Topics

- **Delivery guarantees:** at-least-once vs. exactly-once, and how message IDs enable client-side dedup.
- **Ordering:** server-assigned timestamps vs. sequence numbers per conversation.
- **Connection routing:** how a gateway maps an online user to the server holding their socket.

## Deliverables

- An architecture diagram showing transport, storage, and presence.
- The message send/deliver contract and a history-fetch endpoint.
- A data model for messages and presence.
- One trade-off written up: e.g., WebSocket (true real time, stateful servers, harder to scale) vs. long polling (simpler, higher latency and overhead).

## Common Pitfalls

- Assuming plain HTTP request/response can deliver messages the instant they're sent.
- Ignoring ordering, so messages can display out of sequence under concurrency.
- Forgetting the offline case — messages sent to a disconnected user must not vanish.
- Treating presence as exact; heartbeats always leave a staleness window.

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — patterns for real-time and messaging systems.
- [MDN: WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) — how persistent connections work.
- [RFC 6455: The WebSocket Protocol](https://datatracker.ietf.org/doc/html/rfc6455) — the protocol specification.
- [High Scalability: WhatsApp architecture](http://highscalability.com/blog/2014/2/26/the-whatsapp-architecture-facebook-bought-for-19-billion.html) — a real-world messaging system at scale.

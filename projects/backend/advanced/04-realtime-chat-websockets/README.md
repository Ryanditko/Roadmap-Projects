# Real-time Chat Backend (WebSockets)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build the backend behind a chat app like Slack or WhatsApp Web: messages appear the instant they are sent, you can see who is online, and a dropped connection recovers without losing anything. The core is a WebSocket server holding thousands of long-lived connections, but the hard part is what happens when you outgrow a single process. Two users connected to different server instances still need to see each other's messages, so you will introduce a pub/sub backbone and confront the questions that define real-time systems: how to fan a message out to the right sockets, how to keep messages in order, and how to make presence accurate when connections come and go silently.

## Prerequisites

- Confidence building HTTP APIs and reasoning about the request/response lifecycle
- Understanding of the WebSocket handshake and how it upgrades from HTTP
- Familiarity with an event loop / async I/O in your language of choice
- Basic exposure to a pub/sub or message broker (Redis Pub/Sub, NATS, Kafka)
- A stack of your choice (Node, Go, Elixir, Python) plus a datastore for message history

## Learning Objectives

By the end, you should be able to:

- Manage the lifecycle of thousands of persistent WebSocket connections
- Fan messages out across multiple server instances via a pub/sub backbone
- Reason about message ordering and delivery guarantees (at-least-once vs. exactly-once)
- Track presence accurately using heartbeats and detecting silent disconnects
- Buffer and replay missed messages so a reconnecting client catches up
- Apply backpressure so a slow consumer cannot exhaust server memory

## Functional Requirements

1. Clients must connect over WebSocket, authenticate on connect, and join one or more rooms.
2. A message sent to a room must be delivered to every connected member, including those on other server instances.
3. Messages within a room must be delivered in a consistent order to all recipients.
4. The system must track presence (online/offline/typing) and broadcast changes to interested clients.
5. It must persist message history so a client can load prior messages and resume after reconnecting.
6. It must detect dead connections via heartbeat/ping-pong and clean up their resources.
7. **Scalability:** the design must scale horizontally — adding instances must increase connection capacity without a shared in-process state assumption.
8. **Reliability:** a message accepted by the server must not be silently lost if a delivery target is briefly offline.
9. **Failure modes:** an instance crash must not drop other instances' connections; clients reconnect and resync.

## Suggested Milestones

1. **Milestone 1 — Single-node chat:** One server, in-memory rooms, broadcast to connected sockets.
2. **Milestone 2 — Scale out:** Add a pub/sub backbone so messages cross instances; make servers stateless.
3. **Milestone 3 — Persistence & resume:** Store history, assign sequence numbers, replay on reconnect.
4. **Milestone 4 — Presence & resilience:** Heartbeats, typing indicators, backpressure, and graceful shutdown.

## Data & Interface Sketch

```text
Component view

   clients ==ws==> [Instance A] --publish--> [ Pub/Sub bus ] <--publish-- [Instance B] <==ws== clients
                        |                          |                          |
                        +--> [ Message Store ] <----+---- sequence per room ---+
                        +--> [ Presence Store (TTL keys refreshed by heartbeat) ]

Client -> server frames
  { type: "join",    room }
  { type: "message", room, body, clientMsgId }
  { type: "typing",  room }
  { type: "ping" }

Server -> client frames
  { type: "message", room, seq, senderId, body, ts }
  { type: "presence", room, userId, status }
  { type: "ack", clientMsgId, seq }
  { type: "pong" }

Reconnect: client sends last seen seq per room -> server replays gap from store
```

## Stretch Goals

- Add read receipts and message reactions with correct fan-out to the sender.
- Support message search over history with a dedicated index.
- Add end-to-end or at-rest encryption for message bodies.
- Introduce moderation: rate limits, muting, and profanity filtering.

## Definition of Done

- [ ] Two clients on different server instances exchange messages in real time.
- [ ] Messages in a room arrive in the same order for every recipient.
- [ ] A client that disconnects and reconnects receives every message it missed, exactly once.
- [ ] Presence reflects reality within seconds, and silent disconnects are detected via heartbeat.
- [ ] Killing one instance does not drop connections on the others; clients resync automatically.

## Common Pitfalls

- Assuming a single process: in-memory room state breaks the moment you add a second instance.
- Trusting TCP to notice a dead connection — half-open sockets require application-level heartbeats.
- Ignoring backpressure, so a slow client's unbounded send buffer eventually crashes the server.
- Using timestamps for ordering across instances; clock skew reorders messages — use per-room sequences.
- Broadcasting presence on every keystroke, flooding the bus — debounce and coalesce typing events.

## Resources

- [MDN: The WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) — protocol and client basics.
- [RFC 6455: The WebSocket Protocol](https://datatracker.ietf.org/doc/html/rfc6455) — the authoritative spec, including ping/pong.
- [Redis: Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/) — a common cross-instance fan-out backbone.
- [The C10K problem](http://www.kegel.com/c10k.html) — the classic essay on handling many concurrent connections.
- [Ably: WebSockets vs. long polling](https://ably.com/topic/websockets-vs-long-polling) — trade-offs and scaling context.

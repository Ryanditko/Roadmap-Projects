# Chat UI (WebSocket integration)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Build a real-time chat front end that talks to a WebSocket server. Unlike a request/response app, the server can push a message at any moment, so your UI has to react to events it did not ask for. You will open and manage a socket, render an ever-growing message list that scrolls itself sensibly, show who is online and who is typing, and — the part that separates a toy from a real client — survive a dropped connection and reconnect gracefully. The interesting difficulty is not drawing bubbles; it is keeping the UI honest about connection state and message ordering when the network misbehaves.

## Prerequisites

- Comfort with component state and effects/lifecycle in your framework (React, Vue, Svelte, or Angular)
- Understanding of events and callbacks (the socket is event-driven)
- Familiarity with JSON message serialization
- Basic knowledge of what a WebSocket is versus HTTP

## Learning Objectives

By the end, you should be able to:

- Open, use, and cleanly close a WebSocket, tearing it down when the component unmounts
- Model connection state (connecting, open, reconnecting, closed) and reflect it in the UI
- Append incoming messages and keep the scroll pinned to the bottom without trapping the user
- Broadcast and debounce typing indicators and render presence
- Implement reconnection with backoff and recover missed state on reconnect
- Make the message log accessible to screen readers as new content arrives

## Functional Requirements

1. The app must open a WebSocket on load and display current connection status.
2. Sending a message must push it over the socket and echo it optimistically in the list.
3. Incoming messages must append in order and render sender, text, and a formatted timestamp.
4. The list must auto-scroll to the newest message, unless the user has scrolled up to read history.
5. A typing indicator must appear when another user is typing and clear after they stop.
6. On disconnect, the UI must show a reconnecting state and retry with increasing backoff.
7. The message log must be announced to assistive technology as a live region.

## Suggested Milestones

1. **Milestone 1 — Connect & send:** Open the socket, send a message, and render the echoed list.
2. **Milestone 2 — Receive & scroll:** Handle incoming messages with ordered append and smart auto-scroll.
3. **Milestone 3 — Presence & typing:** Show online users and debounced typing indicators.
4. **Milestone 4 — Resilience:** Detect drops, reconnect with backoff, and restore state.

## Data & Interface Sketch

```text
Layout
  [ Header: room name | status: ● connected ]
  [ Online users ][ Message list (auto-scroll) ]
                  [ "Alice is typing…"          ]
  [ Input box .................... | Send ]

State
  status:    'connecting' | 'open' | 'reconnecting' | 'closed'
  messages:  Message[]        (append-only, ordered by ts)
  users:     User[]           (presence)
  typing:    Set<userId>

Message  { id, userId, text, ts }        // ts = epoch millis
User     { id, name, online }

WebSocket contract (JSON frames)
  send ->  { type: "message",  text }
           { type: "typing",   isTyping }
  recv <-  { type: "message",  message: Message }
           { type: "presence", users: User[] }
           { type: "typing",   userId, isTyping }
```

## Stretch Goals

- Add message reactions or read receipts.
- Persist the last N messages to `localStorage` and rehydrate on load.
- Group consecutive messages from the same sender.
- Show a "N new messages" jump button when the user is scrolled up.
- Support multiple rooms with independent socket subscriptions.

## Definition of Done

- [ ] The socket opens on load and closes cleanly when the component unmounts.
- [ ] Sent and received messages appear in correct order with readable timestamps.
- [ ] Auto-scroll follows new messages but yields when the user scrolls up.
- [ ] A dropped connection triggers a visible reconnecting state and successful retry.
- [ ] New messages are announced via an ARIA live region.

## Common Pitfalls

- Opening a new socket on every render instead of once, leaking connections.
- Forgetting to close the socket on unmount, so handlers fire against a dead component.
- Force-scrolling to the bottom on every message, yanking users away from history they are reading.
- Retrying reconnection in a tight loop with no backoff, hammering the server.
- Trusting client clocks for ordering — sort by a server timestamp or sequence number.

## Resources

- [MDN: The WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) — connecting, sending, and handling events.
- [MDN: Writing WebSocket client applications](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications) — a practical client walkthrough.
- [MDN: ARIA live regions](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions) — announcing dynamic message content.
- [MDN: Intl.RelativeTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/RelativeTimeFormat) — formatting message timestamps.
- [roadmap.sh: Frontend](https://roadmap.sh/frontend) — where real-time and APIs sit in the frontend path.

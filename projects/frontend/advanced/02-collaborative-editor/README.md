# Real-time Collaborative Editor (like Google Docs)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a document editor where several people type into the same document at once and every keystroke shows up on everyone's screen within moments — the experience Google Docs, Notion, and Figma popularized. The deceptively simple demo hides the hardest problem in collaborative software: two users edit the same spot offline, then reconnect, and the system must merge both intents into one consistent document without a central lock. You will lean on a proven convergence algorithm (CRDT or operational transformation) rather than inventing your own, and spend your energy on the frontend concerns that make it feel alive: remote cursors, presence, and an editor that never blocks the local typist waiting on the network.

## Prerequisites

- Comfortable building interactive apps with real-time state (a [Chat Application](../../intermediate/04-chat-ui/) is a good stepping stone)
- Working knowledge of WebSockets and event-driven data flow
- Understanding of the browser Selection and Range APIs, or a rich-text framework
- Awareness that naive "last write wins" loses data — the motivation for this project

## Learning Objectives

By the end, you should be able to:

- Explain why concurrent edits need CRDTs or OT rather than a simple locking scheme
- Integrate a convergence library (Yjs or Automerge) with a rich-text editing surface
- Render remote cursors and selections mapped to positions in the live document
- Keep local edits instant (optimistic) while background sync reconciles state
- Preserve edits made offline and merge them cleanly on reconnection

## Functional Requirements

1. Two or more clients editing the same document must converge to identical content after all changes propagate.
2. A local edit must appear instantly, without waiting for a server round-trip.
3. Each connected user's cursor and text selection must be visible to others, labelled and colored per user.
4. A presence list must show who is currently in the document and update on join/leave.
5. Edits made while offline must be retained and merged automatically once the connection returns.
6. Concurrent edits to the same region must merge deterministically, never silently dropping a user's input.
7. Undo/redo must operate on the local user's own changes without reverting other users' edits.

## Suggested Milestones

1. **Milestone 1 — Single-user editor + transport:** Build the editing surface and a WebSocket channel that echoes changes.
2. **Milestone 2 — Convergence:** Adopt a CRDT/OT library so two clients merge concurrent edits correctly.
3. **Milestone 3 — Presence & cursors:** Broadcast and render remote cursors, selections, and a live presence list.
4. **Milestone 4 — Offline & history:** Queue offline edits, merge on reconnect, and add per-user undo/redo.

## Data & Interface Sketch

```text
   Client A                     Sync server                  Client B
 ┌──────────┐   local ops     ┌───────────────┐   ops      ┌──────────┐
 │ editor   │ ───────────────▶│ relay + doc   │───────────▶│ editor   │
 │ CRDT doc │◀─────────────── │ state (opt.)  │◀───────────│ CRDT doc │
 └────┬─────┘   remote ops    └───────────────┘            └────┬─────┘
      │ optimistic apply (instant, local-first)                 │
      └── presence: { userId, name, color, cursor, selection } ─┘

Op (conceptual):  { type: insert|delete, pos, value?, origin, lamport }
Awareness:        ephemeral, not persisted — cursors, presence, typing
Convergence:      CRDT (Yjs / Automerge) or OT — pick and justify

Non-functional targets:
  local keystroke -> on screen   < 16 ms (no network wait)
  edit -> peer visible           < 250 ms on a healthy link
  offline edits                  never lost on reconnect
```

## Stretch Goals

- Add a version history with named snapshots and a diff view between revisions.
- Support inline comments and suggestions anchored to a text range that survive edits.
- Add document-level permissions (view / comment / edit) enforced on the server.
- Show a "reconnecting" state with an edit queue counter, then a clean catch-up animation.

## Definition of Done

- [ ] Two browsers editing simultaneously end with byte-identical documents.
- [ ] Typing feels instant even with artificial network latency added in dev tools.
- [ ] Remote cursors track the correct character position as text is inserted above them.
- [ ] Disconnecting one client, editing on both, then reconnecting merges without data loss.
- [ ] Undo reverts only the local user's last action, leaving remote edits intact.

## Common Pitfalls

- Reinventing operational transformation from scratch — it is notoriously subtle; use a vetted library.
- Storing cursor positions as absolute offsets, so they point to the wrong place after remote inserts.
- Persisting ephemeral awareness data (cursors, presence) into the document and bloating it.
- Blocking the UI on server acknowledgement, destroying the instant-typing feel.
- Assuming ordered, reliable delivery — networks reorder and drop; the merge must not depend on arrival order.

## Resources

- [Yjs documentation](https://docs.yjs.dev/) — a mature CRDT framework with editor bindings and an awareness protocol.
- [Automerge](https://automerge.org/) — an alternative CRDT library with a strong data-structure model.
- [Martin Kleppmann: CRDTs — the hard parts](https://www.youtube.com/watch?v=x7drE24geUw) — a rigorous talk on convergence guarantees.
- [MDN: Selection API](https://developer.mozilla.org/en-US/docs/Web/API/Selection) — the browser primitive behind cursor and range handling.

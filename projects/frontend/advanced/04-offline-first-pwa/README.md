# Offline-First PWA

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build a Progressive Web App that treats the network as an enhancement, not a requirement — the app keeps working on a train, in a tunnel, or on a flaky connection, then quietly syncs when the network returns. This inverts the usual assumption that data lives on a server and the client is a thin view. Here the client owns a durable local store, renders from it instantly, and reconciles with the backend in the background. The hard parts are the ones users notice only when they break: a stale cache serving last week's data, a background sync that silently loses a write, or two edits that collide after a reconnect. You will design cache strategies deliberately and make the offline state a first-class, visible part of the UX.

## Prerequisites

- A working single-page app you can retrofit (an [Intermediate](../../intermediate/) project works well)
- Understanding of the request/response lifecycle and HTTP caching headers
- Familiarity with Promises and asynchronous data flow
- Awareness of IndexedDB or a wrapper as a client-side database

## Learning Objectives

By the end, you should be able to:

- Register a service worker and intercept network requests with chosen cache strategies
- Choose per-resource strategies (cache-first, network-first, stale-while-revalidate) and justify each
- Persist application data locally in IndexedDB and render from it before the network responds
- Queue writes made offline and replay them reliably on reconnection via Background Sync
- Communicate connectivity and sync status clearly so the user is never confused about data freshness

## Functional Requirements

1. The app must load and be usable on a repeat visit with the network fully disabled.
2. Static assets (app shell) must be served from cache and updated safely when a new version deploys.
3. Application data must be readable offline from a local store, not just static HTML.
4. A write performed offline must be queued and automatically synced once connectivity returns.
5. The UI must clearly indicate offline status and pending, unsynced changes.
6. A new service worker version must not serve a broken mix of old and new assets.
7. Conflicting edits (local vs. server) must be resolved by an explicit, documented strategy — not silent loss.

## Suggested Milestones

1. **Milestone 1 — Installable shell:** Add a manifest and a service worker that caches the app shell for offline load.
2. **Milestone 2 — Offline data:** Store and read application data in IndexedDB; render from it first.
3. **Milestone 3 — Write queue & sync:** Queue offline mutations and replay them with Background Sync on reconnect.
4. **Milestone 4 — Conflicts & updates:** Handle edit conflicts and safe service-worker version rollovers.

## Data & Interface Sketch

```text
   Browser tab                 Service worker              Network
 ┌────────────┐   fetch      ┌─────────────────┐   fetch   ┌────────┐
 │  UI reads  │────────────▶ │  strategy router │─────────▶ │  API   │
 │  from IDB  │◀──────────── │  cache | network │◀───────── │        │
 └─────┬──────┘   response   └────────┬────────┘           └────────┘
       │ writes                        │ cache
 ┌─────▼───────────┐         ┌─────────▼────────┐
 │  IndexedDB       │         │  Cache Storage    │
 │  data + outbox   │         │  app shell + assets│
 └─────┬───────────┘         └──────────────────┘
       │ Background Sync replays outbox on reconnect
       └──────────────────────────────────────────▶ API

Strategies:  shell=cache-first · data=stale-while-revalidate · writes=queued
Conflicts:   last-write-wins | version vector | manual merge — pick + document

Non-functional targets:
  repeat visit offline   fully usable
  app-shell cached size  <= 200 KB
  queued write on reconnect  never lost, replayed once (idempotent)
```

## Stretch Goals

- Add push notifications that surface even when the app is closed.
- Implement periodic background sync to refresh data before the user reopens the app.
- Add an install prompt with a custom, well-timed UX rather than the raw browser banner.
- Show a per-item sync badge (synced / pending / failed) with retry.

## Definition of Done

- [ ] With the network off, a repeat visitor can open the app, read data, and make an edit.
- [ ] That offline edit appears synced to the server automatically after reconnecting, exactly once.
- [ ] Deploying a new version updates the service worker without serving a mismatched asset set.
- [ ] The UI shows offline status and a count of unsynced changes.
- [ ] A deliberately created edit conflict resolves by the documented strategy, losing no user intent silently.

## Common Pitfalls

- Caching everything with cache-first, so users are stuck on stale data with no update path.
- Forgetting service-worker lifecycle (`waiting`/`skipWaiting`), leaving users on an old version indefinitely.
- Treating IndexedDB writes as synchronous, causing lost updates under rapid interaction.
- Replaying the offline outbox without idempotency, duplicating server-side records on flaky reconnects.
- Hiding offline state entirely, so users think their unsynced work is safely saved to the server.

## Resources

- [web.dev: Offline cookbook](https://web.dev/articles/offline-cookbook) — the definitive catalogue of service-worker caching strategies.
- [MDN: Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) — lifecycle, scope, and fetch interception.
- [MDN: IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) — the browser's durable client-side database.
- [web.dev: Background Sync](https://web.dev/articles/background-sync) — reliably deferring writes until connectivity returns.

# Microfrontend Architecture

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Split a single large frontend into several independently built and deployed applications that compose into one seamless product at runtime — the pattern behind large-scale UIs at Spotify, IKEA, and DAZN. A "shell" (host) loads feature apps (remotes) on demand, so teams ship on their own cadence without a shared release train. The hard parts are not the loading mechanics but the boundaries: shared dependency versions, cross-app communication, consistent routing, and a shell that degrades gracefully when a remote fails. This project makes you confront the trade-off at the heart of every microfrontend system — team autonomy versus product consistency — and forces you to defend it with real measurements rather than opinion.

## Prerequisites

- A production-grade single-page app under your belt (the [Admin Panel](../../intermediate/05-admin-panel/) is a good baseline)
- Solid grasp of ES modules, bundling, and code splitting
- Comfort with client-side routing and its history model
- Familiarity with a bundler that supports federation (Vite, Webpack 5, or Rspack)

## Learning Objectives

By the end, you should be able to:

- Compose multiple independently deployed apps behind one shell at runtime
- Share heavy libraries (framework runtime, design system) as singletons to avoid duplication
- Design a decoupled communication contract between remotes without shared global state
- Coordinate routing so deep links resolve correctly across app boundaries
- Isolate failures so one broken remote never blanks the whole page

## Functional Requirements

1. A host shell must dynamically load at least two independently built remote applications at runtime.
2. Each remote must be buildable and deployable on its own, without rebuilding the shell.
3. Shared framework and design-system libraries must resolve to a single shared instance, not one copy per remote.
4. The shell must own top-level routing and delegate sub-routes to the owning remote.
5. Remotes must communicate through an explicit contract (custom events or an injected event bus), never by reaching into each other's internals.
6. If a remote fails to load or throws on mount, the shell must render a fallback and keep the rest of the page usable.
7. Version metadata for each loaded remote must be observable at runtime (e.g. logged or shown in a debug panel).

## Suggested Milestones

1. **Milestone 1 — Shell + one remote:** Stand up a host that loads a single remote via module federation and renders it in a route.
2. **Milestone 2 — Second remote & shared deps:** Add a second remote and configure shared singletons; prove only one framework copy ships.
3. **Milestone 3 — Communication & routing:** Wire cross-remote messaging and end-to-end deep-link routing across boundaries.
4. **Milestone 4 — Resilience & versioning:** Add error boundaries, load fallbacks, and runtime version reporting per remote.

## Data & Interface Sketch

```text
                 ┌──────────────────────────────┐
                 │          Shell (host)         │
                 │  routing · layout · auth      │
                 │  shared singletons ↓          │
                 └───────┬───────────┬───────────┘
          loads at runtime │           │ loads at runtime
                 ┌─────────▼──┐    ┌────▼───────┐
                 │  Remote A  │    │  Remote B  │
                 │ (own repo, │    │ (own repo, │
                 │  own build)│    │  own build)│
                 └─────┬──────┘    └─────┬──────┘
                       └──── event bus ──┘   (decoupled contract)

Shared singletons: framework runtime, design-system, i18n
Communication:     window CustomEvent | injected pub/sub | URL state
Failure mode:      remote load rejects -> shell renders <Fallback/>

Non-functional targets:
  shell-only JS   <= 100 KB gzipped
  remote entry    <= 30 KB gzipped
  broken remote   -> rest of page stays interactive
```

## Stretch Goals

- Add a runtime registry so remotes can be added without editing the shell config.
- Implement independent CI/CD where each remote publishes a versioned `remoteEntry` to a CDN.
- Support two frameworks in different remotes (e.g. React + Vue) to prove true isolation.
- Add server-side composition or an app-shell prerender for first-paint performance.

## Definition of Done

- [ ] Two remotes deploy independently and the shell picks up new versions without a rebuild.
- [ ] Bundle analysis proves shared libraries load once, not once per remote.
- [ ] A deliberately broken remote shows a fallback while the rest of the page stays interactive.
- [ ] Deep links into a remote's sub-route load correctly on a cold page load.
- [ ] Remotes exchange at least one message through the agreed contract, with no direct imports between them.

## Common Pitfalls

- Mismatched shared-dependency versions causing two framework copies to load and hooks to break subtly.
- Coupling remotes through a shared global object instead of an explicit contract — it recreates the monolith you were escaping.
- Letting each remote own routing, producing conflicting history writes and broken back-button behavior.
- Ignoring the failure path, so one 404 on a `remoteEntry` blanks the entire screen.
- Duplicating CSS resets and design tokens per remote, causing visual drift across boundaries.

## Resources

- [Module Federation documentation](https://module-federation.io/) — the canonical guide to the federation runtime and shared scopes.
- [martinfowler.com: Micro Frontends](https://martinfowler.com/articles/micro-frontends.html) — the reference article on the architecture and its trade-offs.
- [web.dev: Reduce JavaScript payloads with code splitting](https://web.dev/articles/reduce-javascript-payloads-with-code-splitting) — the bundling foundation federation builds on.
- [MDN: CustomEvent](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent) — a framework-agnostic way to build a decoupled event bus.

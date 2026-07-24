# Performance-Optimized Large-Scale App

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Take an application that renders large datasets and heavy interactions and make it genuinely fast — not by guessing, but by measuring, setting budgets, and defending them. This is the discipline behind performant apps at scale: a first load that ships only the code the initial view needs, lists of tens of thousands of rows that scroll at 60fps because only the visible ones exist in the DOM, and interactions that never block the main thread long enough to feel janky. The trap is premature or superstitious optimization. Here you will profile first, find the real bottleneck, fix it, and prove the win against a budget — then guard that budget so a future change cannot silently regress it.

## Prerequisites

- A non-trivial app you can profile and optimize (any [Intermediate](../../intermediate/) project works)
- Comfort reading a flame chart and a network waterfall in browser dev tools
- Understanding of the browser rendering pipeline (layout, paint, composite)
- Familiarity with bundling, code splitting, and dynamic imports

## Learning Objectives

By the end, you should be able to:

- Set and enforce performance budgets (bundle size, Core Web Vitals) as build-time gates
- Profile with dev tools to locate the true bottleneck before changing code
- Apply route- and component-level code splitting to shrink the initial payload
- Virtualize long lists so DOM size stays constant regardless of dataset size
- Keep interactions responsive by avoiding long main-thread tasks and unnecessary re-renders

## Functional Requirements

1. The app must render a dataset of at least 10,000 items and scroll it smoothly.
2. The initial JavaScript payload must stay under an explicit, enforced budget.
3. Long lists must be virtualized so DOM node count stays roughly constant as data grows.
4. Non-critical routes and components must load on demand via code splitting, not in the initial bundle.
5. Core Web Vitals (LCP, INP, CLS) must be measured and meet defined thresholds.
6. A performance budget check must run in CI and fail the build when a threshold is exceeded.
7. Every optimization must be backed by a before/after measurement, not assumption.

## Suggested Milestones

1. **Milestone 1 — Baseline & budgets:** Profile the app, record baseline metrics, and define explicit budgets.
2. **Milestone 2 — Payload diet:** Apply code splitting, tree-shaking, and lazy loading to shrink the initial bundle.
3. **Milestone 3 — Render performance:** Virtualize long lists and eliminate wasted re-renders.
4. **Milestone 4 — Guardrails:** Wire budget and Web Vitals checks into CI with pass/fail gates.

## Data & Interface Sketch

```text
   Measure ───▶ Identify bottleneck ───▶ Fix ───▶ Verify vs budget ──┐
      ▲                                                              │
      └──────────────────── guard in CI ◀───────────────────────────┘

Virtualized list (constant DOM):
  data[0..N]  (N = 10,000+)
       │  only render items whose row intersects the viewport
       ▼
  ┌───────────── viewport ─────────────┐
  │ item 412  item 413  item 414 ...    │  ~20 DOM nodes, not 10,000
  └─────────────────────────────────────┘
  spacer above (412 rows) + spacer below (remaining rows)

Budgets (enforced in CI):
  initial JS      <= 170 KB gzipped
  LCP             <= 2.5 s
  INP             <= 200 ms
  CLS             <= 0.1
  list scroll     60 fps (no long task > 50 ms)
```

## Stretch Goals

- Add a Web Worker to move heavy computation (sorting, filtering) off the main thread.
- Implement route-based prefetching so the next likely view is warm before the user clicks.
- Add image optimization: responsive `srcset`, lazy loading, and modern formats.
- Record Web Vitals from real users (field data) and compare against lab measurements.

## Definition of Done

- [ ] A 10,000-item list scrolls at 60fps with a near-constant DOM node count.
- [ ] The initial bundle is under budget, verified by a bundle analyzer.
- [ ] LCP, INP, and CLS meet their thresholds on a mid-tier device profile.
- [ ] CI fails when a change pushes any metric past its budget.
- [ ] Each optimization has a documented before/after measurement.

## Common Pitfalls

- Optimizing without profiling first, fixing something that was never the bottleneck.
- Virtualizing a list but keeping expensive per-row work, so scroll still janks.
- Code splitting so aggressively that the app waterfalls through dozens of tiny lazy chunks.
- Measuring only on a fast machine and fast network, hiding real-world slowness.
- Fixing metrics once but adding no CI guard, letting the next PR regress them silently.

## Resources

- [web.dev: Core Web Vitals](https://web.dev/articles/vitals) — the definitions and thresholds for LCP, INP, and CLS.
- [web.dev: Performance budgets 101](https://web.dev/articles/performance-budgets-101) — how to set and enforce budgets.
- [MDN: Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance_API) — measuring timing in the browser.
- [Chrome DevTools: Performance features reference](https://developer.chrome.com/docs/devtools/performance) — profiling the main thread and rendering.

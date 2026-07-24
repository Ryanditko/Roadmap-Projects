# Frontend Observability Tool

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Frontend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build the kind of tool that Sentry, Datadog RUM, and LogRocket provide: a client-side SDK that captures errors, performance metrics, and user behavior from real browsers, ships them reliably to a backend, and surfaces them in a dashboard where you can diagnose what is breaking for whom. The interesting tension is that an observability tool must be nearly invisible — it cannot slow down the very app it measures, cannot lose data when the page unloads mid-report, and must respect user privacy. You will design a lightweight SDK, a resilient transport that batches and survives navigation, and an aggregation view that turns a flood of raw events into an actionable signal.

## Prerequisites

- Understanding of the browser error model (`error`, `unhandledrejection`) and the Performance API
- Familiarity with Core Web Vitals and what each measures
- Comfort with batching, queuing, and asynchronous transport
- Awareness of privacy concerns around collecting user data

## Learning Objectives

By the end, you should be able to:

- Capture uncaught errors and unhandled promise rejections with useful context (stack, breadcrumbs)
- Collect Core Web Vitals and custom performance timings from real users
- Ship telemetry reliably, including when the page is unloading, without harming app performance
- Aggregate and group high-volume events into a diagnosable dashboard (error grouping, trends)
- Handle privacy: scrubbing PII, sampling, and respecting user consent

## Functional Requirements

1. An SDK must capture uncaught errors and unhandled rejections with stack traces and contextual breadcrumbs.
2. The SDK must collect Core Web Vitals (LCP, INP, CLS) and allow custom timing marks.
3. Telemetry must be batched and sent without blocking the main thread or degrading app performance.
4. Data queued at page unload must still be delivered (e.g. via a keep-alive/beacon mechanism).
5. Similar errors must be grouped (fingerprinted) so a spike is one issue, not thousands of rows.
6. A dashboard must show error frequency, affected sessions, and performance trends over time.
7. PII must be scrubbable and collection must honor a consent/sampling configuration.

## Suggested Milestones

1. **Milestone 1 — Capture SDK:** Hook error and rejection handlers; record breadcrumbs and context.
2. **Milestone 2 — Performance & transport:** Collect Web Vitals and build a batched, unload-safe transport.
3. **Milestone 3 — Ingest & group:** Store events server-side and fingerprint errors into grouped issues.
4. **Milestone 4 — Dashboard & privacy:** Build the trends/issues view and add PII scrubbing + sampling.

## Data & Interface Sketch

```text
   Browser (instrumented app)
   ┌──────────────────────────────────────────────┐
   │ SDK                                            │
   │  window.onerror / onunhandledrejection ──▶ event│
   │  PerformanceObserver (LCP/INP/CLS)      ──▶ event│
   │  breadcrumbs (clicks, navigations, fetches)     │
   │           │ scrub PII · sample                  │
   │           ▼                                     │
   │  batch queue ──flush on: size | interval | unload│
   └───────────┬────────────────────────────────────┘
               │ navigator.sendBeacon / fetch keepalive
               ▼
   Ingest API ─▶ store ─▶ fingerprint & group ─▶ Dashboard
                                                 (issues, trends, sessions)

Event:  { type: error|vital|breadcrumb, ts, session, release, payload }
Fingerprint:  hash(normalized message + top stack frames)
Non-functional targets:
  SDK bundle          <= 15 KB gzipped
  main-thread cost    negligible (no long tasks from the SDK)
  unload delivery     no data lost when the tab closes
  privacy             PII scrubbed before send · sampling configurable
```

## Stretch Goals

- Add session replay-lite: a breadcrumb timeline reconstructing the steps before an error.
- Add alerting when an error's rate crosses a threshold, with a notification hook.
- Add release health: compare error rates across app versions to catch a bad deploy.
- Add source-map support so minified stack traces resolve to original code.

## Definition of Done

- [ ] A thrown error and a rejected promise both appear in the dashboard with a stack and breadcrumbs.
- [ ] Core Web Vitals from a real page load are captured and charted.
- [ ] Events queued right before a page unload are still delivered.
- [ ] Thousands of identical errors collapse into a single grouped issue with a count.
- [ ] PII is scrubbed before transmission and sampling reduces volume as configured.

## Common Pitfalls

- Sending each event immediately, hammering the network and slowing the app it measures.
- Using `fetch` without `keepalive` on unload, so the last (often most important) events are lost.
- Fingerprinting on the raw error message, so dynamic values (ids, URLs) explode one issue into thousands.
- Capturing full DOM or form contents as breadcrumbs, leaking passwords and personal data.
- Instrumenting synchronously in hot paths, adding long tasks that distort the very metrics you collect.

## Resources

- [MDN: GlobalEventHandlers.onerror](https://developer.mozilla.org/en-US/docs/Web/API/Window/error_event) — capturing uncaught errors in the browser.
- [web.dev: Measure performance with the RUM Data Model](https://web.dev/articles/vitals-measurement-getting-started) — collecting Web Vitals from real users.
- [MDN: Navigator.sendBeacon()](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon) — reliably sending data as the page unloads.
- [MDN: PerformanceObserver](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) — observing performance entries without polling.

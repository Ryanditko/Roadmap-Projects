# Health Check System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a small system that repeatedly asks your services "are you okay?" and does something useful when the answer is no. It probes each target on a schedule — an HTTP endpoint, a TCP port, a dependency like a database — records the result, and alerts when a target stays down past a tolerance you set. The subtlety is telling a real outage apart from a blip: one failed probe should not wake anyone, but three in a row should. You will also learn the difference between liveness ("is it running?") and readiness ("can it serve traffic?"), a distinction that underpins every orchestrator's health model.

## Prerequisites

- One or more services with a reachable endpoint or port
- A scripting language that can make HTTP/TCP requests
- Understanding of HTTP status codes and timeouts
- A way to notify (console, log, or chat webhook)

## Learning Objectives

By the end, you should be able to:

- Probe HTTP and TCP targets on a fixed interval with timeouts
- Distinguish liveness from readiness and check dependencies
- Require consecutive failures before declaring a target down
- Alert on a state change (up→down, down→up), not on every probe
- Record probe history to compute simple uptime

## Functional Requirements

1. The system must check a configurable list of targets on a fixed interval.
2. Each probe must enforce a timeout and treat a timeout as a failure.
3. A target must be marked down only after N consecutive failures (configurable).
4. It must alert once on each state transition, not on every failed probe.
5. It must support at least HTTP (status + body/keyword) and TCP-port checks.
6. It must record each result so uptime over a period can be reported.
7. Targets, intervals, thresholds, and timeouts must be configurable.

## Suggested Milestones

1. **Milestone 1 — Probe & report:** Check a list of HTTP targets on an interval and print up/down.
2. **Milestone 2 — Debounce & alert:** Require N consecutive failures and alert on state transitions only.
3. **Milestone 3 — History & uptime:** Persist results and expose an uptime summary and a simple status view.

## Data & Interface Sketch

```text
Config (structure, not full file)
  interval_seconds: 30
  targets:
    - name: api
      type: http
      url: http://localhost:8080/health
      expect_status: 200
      timeout_ms: 2000
      unhealthy_after: 3     # consecutive failures
    - name: db
      type: tcp
      host: localhost
      port: 5432
      timeout_ms: 1000

State machine per target
  UP --(N consecutive fails)--> DOWN   (alert)
  DOWN --(1 success)--> UP              (recovery alert)

Result record
  { target, ok, latency_ms, checked_at, consecutive_failures }
```

## Stretch Goals

- Add a readiness vs liveness distinction and check downstream dependencies separately.
- Serve a small status page listing each target's current state and 24h uptime.
- Add jitter to probe timing so all checks do not fire at the same instant.
- Escalate: warn after N failures, page after M.

## Definition of Done

- [ ] Targets are probed on the configured interval with enforced timeouts.
- [ ] A single failed probe does not trigger an alert; N in a row does.
- [ ] Recovery (down→up) produces a distinct alert.
- [ ] Uptime over a window can be reported from recorded history.
- [ ] All thresholds and targets come from config, not code.

## Common Pitfalls

- No timeout on probes, so one hung target stalls the whole check loop.
- Alerting on every failed probe instead of on the state change, causing alert fatigue.
- Treating any 2xx–3xx as healthy when the app returns 200 with an error body.
- Checking only liveness, so a process that is up but cannot reach its database looks healthy.

## Resources

- [Kubernetes: Liveness, readiness and startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) — the canonical model.
- [Microsoft: Health Endpoint Monitoring pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/health-endpoint-monitoring) — designing health endpoints.
- [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) — what "healthy" actually means.
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) — signals, symptoms, and alerting.

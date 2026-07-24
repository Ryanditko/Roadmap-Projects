# Service Restart Monitor

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Build a small supervisor that watches a set of local services, notices when one dies or stops responding, and restarts it automatically. This is the idea behind tools like systemd, supervisord, and Kubernetes liveness probes, distilled to its core: a loop that checks health, decides whether a process is alive, and acts. The interesting part is not the restart itself but the discipline around it — knowing the difference between "not running" and "running but unhealthy", backing off so you don't hammer a broken service, and stopping after too many failures instead of restarting forever. You will finish with a clear mental model of what a process supervisor actually does and why naive "just restart it" scripts cause more outages than they fix.

## Prerequisites

- Comfort running and stopping processes from a shell
- Basic scripting in any language (Bash, Python, Go, or Node)
- Understanding of exit codes and how a process reports success or failure
- Familiarity with reading logs and timestamps

## Learning Objectives

By the end, you should be able to:

- Distinguish liveness (is the process running?) from readiness (is it actually serving?)
- Implement a health check via process status, a TCP port, or an HTTP endpoint
- Apply exponential backoff so retries slow down instead of tightening into a loop
- Enforce a restart limit and enter a "give up" state that alerts instead of retrying
- Keep an auditable history of failures and restart decisions

## Functional Requirements

1. The monitor must track a configurable list of services, each with its own health check.
2. It must detect when a service is down (dead process, closed port, or failing endpoint).
3. On failure, it must attempt a restart and record that it did so.
4. Retries must use exponential backoff with a configurable base and cap.
5. After a configurable number of failures in a window, it must stop restarting and raise an alert instead.
6. A manual override must let an operator disable or force-restart a specific service.
7. Every state change (down, restarting, recovered, gave-up) must be logged with a timestamp.

## Suggested Milestones

1. **Milestone 1 — Detect & restart:** Poll one service, detect a dead process, restart it, log the event.
2. **Milestone 2 — Backoff & limits:** Add exponential backoff and a max-restart threshold with a give-up state.
3. **Milestone 3 — Multiple services & override:** Drive several services from config and add manual controls.

## Data & Interface Sketch

```text
config (per service)
  name           string
  start_cmd      string
  check          { type: process | tcp | http, target, interval_s }
  backoff        { base_s, max_s }
  max_restarts   int   (within window_s)

service state (in memory)
  status         running | down | restarting | gave_up
  failures       int
  next_attempt   timestamp
  last_change    timestamp

control surface (CLI or file)
  status                 -> table of services + state
  disable <name>         -> stop monitoring
  restart <name>         -> force one restart

restart decision:
  down -> attempt if failures < max_restarts
       -> wait min(base * 2^failures, max) before next try
       -> failures >= max within window -> gave_up + alert
```

## Stretch Goals

- Add service dependencies so a restart cascades in the correct order.
- Send notifications to a chat webhook (e.g. Slack) when a service enters give-up.
- Expose a tiny status dashboard or `/healthz` endpoint for the monitor itself.
- Persist history to a file so restart counts survive a monitor restart.

## Definition of Done

- [ ] A killed service is detected and restarted within one check interval.
- [ ] Repeated failures back off exponentially rather than retrying immediately.
- [ ] After the restart limit is hit, the service enters give-up and alerts instead of looping.
- [ ] A healthy-again service resets its failure counter and returns to normal monitoring.
- [ ] Every transition is logged with a timestamp and is readable after the fact.

## Common Pitfalls

- Treating "process exists" as "healthy" — a hung process passes a PID check but serves nothing.
- Restarting with no backoff, turning a crash loop into a CPU-pinning hot loop.
- Never giving up, so a permanently broken service masks the real problem behind endless restarts.
- Forgetting to reset the failure counter after recovery, causing premature give-up later.
- Racing on restart: launching a second copy before confirming the first is truly gone.

## Resources

- [systemd.service manual](https://www.freedesktop.org/software/systemd/man/systemd.service.html) — how a real supervisor models restart policy.
- [Supervisor documentation](http://supervisord.org/) — a process control system with the exact concepts here.
- [Kubernetes: Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) — liveness vs readiness in practice.
- [Google SRE Book: Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/) — why backoff and limits matter.

# Feature Flag Service

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build the service that powers controlled rollouts and A/B testing at companies like LaunchDarkly or the internal platforms at Netflix and Facebook. A feature flag service lets teams turn functionality on or off — and target it at specific users — without deploying new code. The interesting part is not the on/off switch; it is the evaluation engine that must resolve a flag for a given user in well under a millisecond, from thousands of services, consistently, even when the network to your control plane is down. You will design a targeting-rule model, a low-latency delivery path, and the trade-offs between real-time freshness and availability that make a flag system trustworthy enough to gate a payment flow.

## Prerequisites

- Solid REST API design and experience modeling non-trivial domains
- Familiarity with caching strategies and cache invalidation trade-offs
- Understanding of percentiles (p99 latency) and why tail latency matters
- Basic statistics for A/B testing (sample size, significance) is helpful
- Comfort reasoning about consistency vs. availability under partition

## Learning Objectives

By the end, you should be able to:

- Model flags, variations, and targeting rules as evaluable data
- Build a deterministic evaluation engine that resolves rules in a fixed order
- Design a delivery path (SDK + cache/stream) that survives control-plane outages
- Implement percentage rollouts with stable, sticky user bucketing
- Reason about evaluation consistency across many services and instances
- Emit exposure events that feed A/B test analysis without slowing evaluation

## Functional Requirements

1. The system must let an operator define a flag with a default value and one or more variations (boolean or multivariate).
2. The system must evaluate a flag against a user context (attributes like id, plan, country) and return exactly one variation deterministically.
3. Targeting rules must be evaluated in a defined, documented order; the first matching rule wins, else the default applies.
4. Percentage rollouts must be sticky: the same user must always fall in the same bucket unless the rule changes.
5. The evaluation path must serve reads at p99 well below 5 ms and must keep serving the last known ruleset if the control plane is unreachable (availability over freshness).
6. Flag changes must propagate to evaluators within a bounded, documented staleness window (e.g. streamed within seconds, or polled).
7. Every flag change must be recorded in an immutable audit trail (who, what, when).
8. The system must expose a kill switch that forces a flag to its default instantly across all consumers.
9. Evaluation must emit exposure events (user, flag, variation) for analytics without blocking the response.

## Suggested Milestones

1. **Milestone 1 — Flag model & CRUD:** Define flags with variations and defaults; build the management API and audit log.
2. **Milestone 2 — Evaluation engine:** Implement ordered targeting rules and deterministic single-variation resolution against a user context.
3. **Milestone 3 — Sticky rollouts:** Add percentage rollouts using a hash of `flagKey + userId` for stable bucketing.
4. **Milestone 4 — Delivery & resilience:** Add a client cache/stream so evaluators serve locally and survive control-plane downtime.
5. **Milestone 5 — Exposure & experiments:** Emit exposure events asynchronously and expose basic A/B metrics.

## Data & Interface Sketch

```text
Flag
  key:         string
  type:        boolean | multivariate
  variations:  [ v0, v1, ... ]
  default:     variationIndex
  rules:       [ { conditions:[...], variation | rollout } ]  (ordered)
  version:     integer

Evaluation flow
  Consumer SDK --(poll/stream ruleset)--> Control Plane API
       |  (local, in-memory eval, no network on hot path)
       v
  evaluate(flagKey, userContext) -> { variation, ruleMatched, reason }
       |
       +--async--> Exposure Event Bus --> Analytics store

Sticky rollout:  bucket = hash(flagKey + ":" + userId) % 100
                 in-rollout if bucket < rolloutPercentage

POST /flags                 -> 201 flag
PATCH /flags/{key}          -> 200 (new version, audit entry)
POST /evaluate              body {flagKey, userContext} -> 200 {variation, reason}
POST /flags/{key}/kill      -> 200 (force default everywhere)
```

## Stretch Goals

- Add prerequisite flags (flag B only evaluates if flag A is on).
- Support user segments as reusable, named rule fragments.
- Add scheduled rollouts that ramp a percentage up over time automatically.
- Compute A/B significance and guardrail metrics from exposure events.
- Provide a streaming delivery mode (SSE) and compare its staleness vs. polling.

## Definition of Done

- [ ] The same user context always resolves to the same variation for a fixed ruleset.
- [ ] Rule order is deterministic and documented; the default is returned when nothing matches.
- [ ] Percentage rollouts are sticky across restarts and instances.
- [ ] Evaluators serve the last known ruleset when the control plane is down.
- [ ] Every change is captured in the audit trail and the kill switch forces defaults instantly.
- [ ] Exposure events are emitted without adding latency to evaluation.

## Common Pitfalls

- Evaluating flags over the network on every request — the hot path must be local and in-memory.
- Bucketing on a random number instead of a stable hash, so users flicker between variations.
- Failing closed (erroring) when the control plane is down instead of failing to the last known value.
- Leaking the evaluation into analytics synchronously, coupling response latency to your event pipeline.
- Mutating flags in place with no version or audit, making a bad rollout impossible to explain.
- Ignoring cardinality of user attributes in targeting, letting rules explode in cost.

## Resources

- [Martin Fowler: Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) — the canonical taxonomy of flag types.
- [OpenFeature Specification](https://openfeature.dev/specification/) — a vendor-neutral standard for flag evaluation and SDKs.
- [LaunchDarkly Blog](https://launchdarkly.com/blog/) — practical engineering write-ups on delivery and consistency.
- [Consistent hashing (Wikipedia)](https://en.wikipedia.org/wiki/Consistent_hashing) — background for stable, well-distributed bucketing.
- [Trustworthy Online Controlled Experiments (Kohavi et al.)](https://experimentguide.com/) — the reference on running A/B tests correctly.

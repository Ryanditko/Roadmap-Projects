# Service Mesh Implementation

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Move the cross-cutting concerns of microservice communication — encryption, retries, timeouts, traffic splitting, and observability — out of every application and into a dedicated infrastructure layer. You will deploy a service mesh that injects a sidecar proxy (or a per-node proxy) alongside your services, so that traffic between them is intercepted and governed by policy rather than by code scattered across teams. The advanced work is not "run the installer"; it is deciding what belongs in the mesh versus the app, enabling mutual TLS without breaking existing traffic, writing authorization policy that is deny-by-default, and paying honest attention to the latency and resource overhead a proxy on every hop introduces. Done right, security and reliability become properties of the platform, not per-service homework.

## Prerequisites

- A Kubernetes cluster running at least two services that call each other
- Understanding of TLS, certificates, and mutual authentication
- Familiarity with L7 concepts: retries, timeouts, circuit breaking
- Basic observability so you can measure the overhead the mesh adds

## Learning Objectives

By the end, you should be able to:

- Deploy a service mesh and understand the sidecar/data-plane vs. control-plane split
- Enable mutual TLS between services without dropping traffic
- Write deny-by-default authorization policies between workloads
- Configure traffic management: retries, timeouts, circuit breaking, and splitting
- Quantify and reason about the mesh's latency and resource overhead

## Functional Requirements

1. Traffic between meshed services must be encrypted with mutual TLS automatically.
2. Service-to-service authorization must be deny-by-default with explicit allow rules.
3. The mesh must enforce configurable retries, timeouts, and circuit breaking.
4. Traffic splitting between service versions must be controllable without app changes.
5. The mesh must emit L7 telemetry (per-route latency, error rate) for observability.
6. Enabling the mesh must not require rewriting application networking code.
7. The added latency and resource overhead must be measured and documented.

## Suggested Milestones

1. **Milestone 1 — Mesh & sidecars:** Install the mesh and inject sidecars for two communicating services.
2. **Milestone 2 — mTLS & authz:** Turn on mutual TLS and write deny-by-default authorization policy.
3. **Milestone 3 — Traffic management:** Add retries, timeouts, circuit breaking, and version-based splitting.
4. **Milestone 4 — Observability & cost:** Wire up L7 telemetry and measure the mesh's overhead against a baseline.

## Data & Interface Sketch

```text
                 ┌───────────────────────┐
                 │   Control Plane        │  (config, certs, policy)
                 │  (istiod / linkerd)    │
                 └───────────┬───────────┘
             push config/certs│
        ┌──────────────┐      │      ┌──────────────┐
        │  Service A   │      │      │  Service B   │
        │ ┌──────────┐ │      │      │ ┌──────────┐ │
   ───▶ │ │  proxy   │◀┼──mTLS┼──────┼▶│  proxy   │ │ ───▶
        │ └────┬─────┘ │      │      │ └────┬─────┘ │
        │   app A      │      │      │   app B      │
        └──────────────┘      │      └──────────────┘
             data plane (sidecars enforce policy on every hop)

Policy example (conceptual):
  authz: default DENY
         allow  A -> B  on route /orders  method GET
  traffic: B retries=2 timeout=2s circuit-break at 50% 5xx
           split B: v1=90% v2=10%

Non-functional targets:
  added p99 latency  measured per hop, kept within budget
  proxy overhead     CPU/mem per sidecar quantified
  security           0 plaintext service-to-service traffic
```

## Stretch Goals

- Compare a sidecar mesh with a sidecarless/ambient mode and measure the overhead difference.
- Extend authorization to use request identity (JWT) not just workload identity.
- Add fault injection at the mesh layer to combine with a chaos-engineering practice.
- Federate the mesh across two clusters for cross-cluster mTLS and discovery.

## Definition of Done

- [ ] Service-to-service traffic is mutually TLS-encrypted with no app code changes.
- [ ] Authorization is deny-by-default with explicit, tested allow rules.
- [ ] Retries, timeouts, and circuit breaking are enforced and demonstrably trigger.
- [ ] Traffic can be split between versions purely via mesh config.
- [ ] Mesh latency and resource overhead are measured and documented against a baseline.

## Common Pitfalls

- Turning on strict mTLS globally at once and cutting off services not yet in the mesh.
- Leaving authorization at allow-all, so the mesh adds encryption but no real access control.
- Ignoring proxy overhead until latency-sensitive paths regress in production.
- Configuring aggressive retries that amplify load and turn a blip into a retry storm.
- Putting logic in the mesh that belongs in the app (or vice versa), blurring ownership.

## Resources

- [Istio documentation](https://istio.io/latest/docs/) — a widely used service mesh with rich traffic policy.
- [Linkerd documentation](https://linkerd.io/2/overview/) — a lightweight, security-focused mesh.
- [SMI: Service Mesh Interface](https://smi-spec.io/) — a vendor-neutral mesh API spec.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — retries, load shedding, and circuit breaking done safely.

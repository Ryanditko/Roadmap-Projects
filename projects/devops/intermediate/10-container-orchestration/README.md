# Container Orchestration

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Run a real workload across a small cluster of machines and let an orchestrator — Kubernetes or Docker Swarm — decide where containers go, keep the desired number running, and reschedule them when a node dies. This project moves you from "docker run on one box" to declarative, self-healing deployment. You will declare a desired state, watch the orchestrator reconcile reality toward it, expose services through cluster networking and discovery, attach persistent storage to stateful workloads, and roll out a new version without downtime. The lesson is the reconciliation loop mindset: you describe what you want, the control plane continuously closes the gap, and your job shifts from running commands to authoring correct desired state.

## Prerequisites

- Comfort building and running container images
- Two or more nodes (VMs, cloud instances, or a local multi-node cluster like kind/k3d)
- Understanding of container networking basics (ports, DNS, overlays)
- Familiarity with health checks and the difference between liveness and readiness
- A stepping stone: [Load Balancing System](../09-load-balancing/) explains the service routing this relies on

## Learning Objectives

By the end, you should be able to:

- Express a workload as declarative desired state and let the orchestrator reconcile it
- Use service discovery and cluster networking to connect components without hardcoded IPs
- Attach persistent volumes so stateful containers survive rescheduling
- Set resource requests/limits and understand how the scheduler places workloads
- Perform a rolling update and a rollback with zero downtime

## Functional Requirements

1. A multi-node cluster must run a workload of at least two communicating services.
2. The workload must be declared as desired state (replica count, image, resources), not imperative commands.
3. Killing a container or draining a node must trigger automatic rescheduling to restore desired state.
4. Services must reach each other via service discovery, not hardcoded addresses.
5. At least one service must use a persistent volume that survives a reschedule.
6. A rolling update must deploy a new version without dropping traffic, and rollback must be possible.
7. Resource requests/limits must be set so the scheduler can place and protect workloads.

## Suggested Milestones

1. **Milestone 1 — Cluster & deploy:** Stand up the cluster and deploy a replicated service from desired state.
2. **Milestone 2 — Networking & storage:** Wire service discovery between components and attach a persistent volume.
3. **Milestone 3 — Resilience & rollout:** Prove self-healing on node loss and perform a rolling update with rollback.

## Data & Interface Sketch

```text
cluster
  control-plane        reconciles desired vs actual
  node A, node B ...    run scheduled containers

desired state (per service)
  name           string
  image          repo:tag
  replicas       int
  resources      { requests, limits }
  probes         { liveness, readiness }
  network        service name -> stable virtual endpoint
  storage        volume claim (for stateful)

reconciliation loop:
  observe actual -> diff against desired -> act (create/kill/move) -> repeat
  node down -> its containers rescheduled onto healthy nodes

rolling update:
  bump image tag -> new replicas up + ready -> old drained -> repeat
  failure -> rollback to previous desired state
```

## Stretch Goals

- Add an ingress/gateway so external traffic reaches services by hostname or path.
- Add autoscaling of replicas based on a metric (ties into the auto-scaling project).
- Add config and secrets management injected into containers at runtime.
- Introduce affinity/anti-affinity rules so replicas spread across nodes and zones.

## Definition of Done

- [ ] The cluster runs the declared workload at the requested replica count across nodes.
- [ ] Killing a container or node restores desired state automatically without manual intervention.
- [ ] Services communicate via discovery, surviving container restarts and reschedules.
- [ ] A stateful service keeps its data across a reschedule via a persistent volume.
- [ ] A rolling update ships a new version with no dropped requests, and rollback works.

## Common Pitfalls

- Treating the orchestrator imperatively (manual `run`/`kill`) and fighting the reconciliation loop.
- Missing readiness probes, so rolling updates send traffic to containers that aren't ready yet.
- Assuming local disk persists — without a real volume, data vanishes on reschedule.
- No resource requests, letting one workload starve others and causing noisy-neighbor evictions.
- Rolling out with a single replica, so the update itself is the outage.

## Resources

- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/) — desired state, controllers, and the reconciliation model.
- [Kubernetes: Rolling Update Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment) — zero-downtime rollouts and rollback.
- [Docker Swarm mode overview](https://docs.docker.com/engine/swarm/) — a lighter orchestrator with the same core ideas.
- [The Twelve-Factor App](https://12factor.net/) — config, statelessness, and disposability that orchestration assumes.

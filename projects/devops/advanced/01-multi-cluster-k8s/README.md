# Multi-Cluster Kubernetes Architecture

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Run a single logical platform across several Kubernetes clusters so that the loss of one cluster — or an entire availability zone — does not take your workloads down. You will stand up at least two clusters, give them a shared identity and networking story, and put a control plane in front that decides where workloads run and how traffic reaches them. The interesting problems are not "install Kubernetes twice"; they are cluster discovery, cross-cluster service resolution, config propagation without drift, and a failover that is fast enough to matter but conservative enough not to flap. Treat this as an exercise in reasoning about blast radius: what breaks when one cluster dies, and how do you prove recovery beforehand?

## Prerequisites

- Solid single-cluster Kubernetes experience — Deployments, Services, Ingress, RBAC ([Kubernetes Deployment](../../intermediate/) work is a good stepping stone if you need one)
- Comfort with a managed or self-hosted cluster provider (EKS, GKE, AKS, or kubeadm)
- Understanding of DNS, load balancing, and the L4/L7 distinction
- Familiarity with declarative config and a tool like Helm or Kustomize

## Learning Objectives

By the end, you should be able to:

- Design a multi-cluster topology and justify federation vs. independent clusters
- Register clusters with a central control plane and propagate configuration consistently
- Route traffic across clusters with health-aware failover and locality preference
- Replicate or partition state deliberately, reasoning about consistency trade-offs
- Define availability targets and measure recovery time against them

## Functional Requirements

1. The platform must run across at least two clusters with a single point of workload placement.
2. A service in one cluster must be resolvable and reachable from another cluster.
3. Configuration changes must propagate to all target clusters without manual per-cluster editing.
4. When a cluster becomes unhealthy, traffic must fail over to a surviving cluster automatically.
5. The system must expose per-cluster and aggregate health so operators can see blast radius.
6. Failover and failback must be reversible and must not lose in-flight requests silently.
7. Access control and security policies must apply uniformly across all clusters.

## Suggested Milestones

1. **Milestone 1 — Two clusters, shared identity:** Provision the clusters, establish networking and a common trust root, verify pod-to-pod reachability across clusters.
2. **Milestone 2 — Placement & propagation:** Introduce a control plane (Karmada, Cluster API, or fleet tooling) that schedules workloads and syncs config to both clusters.
3. **Milestone 3 — Cross-cluster traffic & failover:** Add global service resolution and a health-aware router, then kill a cluster and measure recovery.
4. **Milestone 4 — Guardrails:** Uniform RBAC/network policy, aggregate observability, and a documented disaster-recovery runbook.

## Data & Interface Sketch

```text
                       ┌────────────────────────┐
   operators / GitOps ─▶│  Control Plane / Fleet │  (placement, config sync)
                       └───────────┬────────────┘
                        ┌──────────┴──────────┐
                        ▼                     ▼
                 ┌────────────┐        ┌────────────┐
   Global LB ───▶│ Cluster A  │        │ Cluster B  │◀─── Global LB
   (GeoDNS /     │  us-east   │◀──mTLS─▶│  eu-west   │
    Anycast)     └────────────┘        └────────────┘
                   svc: api      cross-cluster    svc: api
                                 service discovery

Non-functional targets to state up front:
  availability   >= 99.95% platform-wide (survives 1 cluster loss)
  failover RTO   < 60s to shift traffic off a dead cluster
  config drift   0 undetected divergences between clusters
```

## Stretch Goals

- Add a third cluster and test placement policies that respect region/zone locality.
- Introduce stateful workload replication (e.g. a replicated datastore) and reason about its RPO.
- Automate failback with a canary re-entry so recovered clusters don't take full load instantly.
- Add cost-awareness so the scheduler prefers cheaper clusters when SLOs allow.

## Definition of Done

- [ ] Two or more clusters run under a single control plane with config synced automatically.
- [ ] A service is reachable cross-cluster and survives one cluster being deleted.
- [ ] Failover happens within your stated RTO and is observable in a dashboard.
- [ ] There is no undetected config drift — a check flags divergence.
- [ ] A disaster-recovery runbook exists and has been rehearsed at least once.

## Common Pitfalls

- Treating multi-cluster as "prod + DR" and never actually exercising failover, so it fails when needed.
- Overlapping pod/service CIDRs across clusters, making cross-cluster routing impossible without NAT.
- Letting each cluster drift because config is applied manually instead of reconciled from one source.
- Replicating state naively and discovering split-brain the hard way during a partition.
- Ignoring the cost and operational tax of the extra cluster until the bill or the pager arrives.

## Resources

- [Kubernetes: Multi-cluster concepts](https://kubernetes.io/docs/concepts/cluster-administration/) — cluster administration fundamentals.
- [Karmada documentation](https://karmada.io/docs/) — a CNCF multi-cluster orchestration control plane.
- [Cluster API](https://cluster-api.sigs.k8s.io/) — declarative cluster lifecycle management.
- [Google SRE Book: Managing Critical State](https://sre.google/sre-book/managing-critical-state/) — consistency and failover trade-offs.

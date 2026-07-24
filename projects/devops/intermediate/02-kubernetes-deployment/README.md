# Kubernetes Deployment

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Take a containerized web application and run it properly on a Kubernetes cluster — not just "kubectl run and hope", but a real deployment with declarative manifests, a stable service address, external ingress, configuration split from secrets, resource requests, health probes, and a rolling update that ships a new version without dropping traffic. You can use any cluster: a local one like kind, k3d, or minikube, or a managed cloud cluster. The lesson is how Kubernetes turns a set of desired-state YAML files into a self-healing, updatable running system.

## Prerequisites

- A container image of an app, pushed to a registry the cluster can pull from
- A working cluster and `kubectl` configured against it
- Understanding of containers, ports, and environment variables
- Familiarity with YAML and the request/response lifecycle of a web app

## Learning Objectives

By the end, you should be able to:

- Express desired state with Deployment, Service, and Ingress manifests
- Separate configuration (ConfigMap) from secrets and inject both into pods
- Set resource requests/limits and reason about scheduling and eviction
- Configure liveness and readiness probes so traffic only hits healthy pods
- Perform a rolling update and roll it back when the new version misbehaves

## Functional Requirements

1. The app must be described by a Deployment running at least two replicas.
2. A Service must give the pods a stable in-cluster address independent of pod churn.
3. External traffic must reach the app through an Ingress (or an equivalent LoadBalancer).
4. Non-secret config must come from a ConfigMap; sensitive values from a Secret.
5. Every pod must declare resource requests and limits.
6. Readiness and liveness probes must keep traffic off pods that are starting or unhealthy.
7. A rolling update must ship a new image with zero dropped requests, and `kubectl rollout undo` must restore the previous version.

## Suggested Milestones

1. **Milestone 1 — Run it:** Deployment + Service, app reachable inside the cluster.
2. **Milestone 2 — Expose & configure:** Add Ingress, ConfigMap, and Secret; wire config into pods.
3. **Milestone 3 — Resilience & updates:** Add probes and resource limits; do a rolling update and a rollback.

## Data & Interface Sketch

```text
Object relationships:
  Ingress ──routes──> Service ──selects(labels)──> Pods (from Deployment)
  ConfigMap ─┐
  Secret ────┴─mounted/env─> Pods

Deployment spec (structure only):
  replicas: 2
  strategy: RollingUpdate (maxUnavailable, maxSurge)
  template:
    containers:
      - image: registry/app:<tag>
        resources: { requests: {cpu, mem}, limits: {cpu, mem} }
        readinessProbe: httpGet /healthz
        livenessProbe:  httpGet /healthz
        envFrom: [configMapRef, secretRef]

Update:   kubectl set image ...  -> new ReplicaSet scales up, old scales down
Rollback: kubectl rollout undo deployment/app
```

## Stretch Goals

- Add a HorizontalPodAutoscaler that scales replicas on CPU.
- Add a PodDisruptionBudget so voluntary evictions never take the app below a floor.
- Split into a StatefulSet for a component that needs stable identity and storage.
- Add NetworkPolicies restricting which pods can talk to each other.

## Definition of Done

- [ ] Deleting a pod results in Kubernetes recreating it automatically.
- [ ] The Service address stays stable while pods are replaced.
- [ ] Config and secrets are injected from ConfigMap/Secret, not baked into the image.
- [ ] A rolling update completes with no failed requests against the app.
- [ ] `kubectl rollout undo` restores the prior version and it serves traffic.

## Common Pitfalls

- Omitting readiness probes, so traffic hits a pod before it can serve and users see errors.
- Setting no resource requests, letting one noisy pod starve its neighbors.
- Baking secrets into the image or a ConfigMap instead of a Secret.
- Mismatched label selectors between Deployment and Service, leaving the Service with zero endpoints.
- Assuming a rolling update is safe without a probe — Kubernetes will happily route to a broken new pod.

## Resources

- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/) — the object model, explained by the project.
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) — rolling updates and rollbacks.
- [Configure liveness, readiness and startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) — probe semantics.
- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) — routing external traffic into the cluster.
</content>

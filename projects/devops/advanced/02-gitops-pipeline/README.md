# GitOps Pipeline

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Make Git the single source of truth for what runs in your clusters, and let a reconciler — not a human running `kubectl apply` — converge reality to the declared state. You will build a pipeline where a merge to a repository triggers an automatic, auditable deployment, where any manual change to the cluster is detected as drift and reverted, and where rollbacks are just `git revert`. The hard parts are the ones people skip in demos: keeping secrets out of plaintext Git, structuring repos so multiple environments don't step on each other, adding approval gates without turning GitOps back into ClickOps, and proving that "the repo describes the cluster" is actually true. Done well, your deploy history and your Git history become the same thing.

## Prerequisites

- Working Kubernetes knowledge and comfort with declarative manifests
- Experience with Git branching, pull requests, and review workflows
- Familiarity with Helm or Kustomize for environment overlays
- Basic understanding of secret management concepts (encryption at rest, KMS)

## Learning Objectives

By the end, you should be able to:

- Model desired state in Git and have an operator continuously reconcile it
- Detect and remediate configuration drift automatically
- Structure repositories for multiple environments and progressive promotion
- Handle secrets safely in a Git-driven workflow
- Roll back a deployment purely through Git history and observe the reconciler recover

## Functional Requirements

1. A merge to the target branch must trigger a deployment with no manual `kubectl` step.
2. The operator must continuously reconcile cluster state against the repository.
3. Manual changes to managed resources must be detected as drift and reported or reverted.
4. Secrets must never be committed in plaintext; they must be encrypted or externally referenced.
5. Promotion from one environment to the next must be an explicit, reviewable action.
6. A `git revert` of a deployment commit must roll the cluster back to the prior state.
7. Every change to the cluster must be traceable to a commit, author, and approval.

## Suggested Milestones

1. **Milestone 1 — Reconcile from Git:** Install an operator (Argo CD or Flux), point it at a repo, and watch it deploy and self-heal.
2. **Milestone 2 — Drift & rollback:** Make a manual change and confirm drift detection; then roll back via `git revert`.
3. **Milestone 3 — Multi-environment:** Structure dev/staging/prod with overlays and add a promotion workflow with approval.
4. **Milestone 4 — Secrets & policy:** Add sealed/encrypted secrets and a policy check that blocks non-compliant manifests before merge.

## Data & Interface Sketch

```text
   developer ──PR──▶ Git repo (source of truth)
                         │  desired state (manifests / Helm / Kustomize)
                         ▼
                 ┌───────────────┐   reconcile loop (every N sec)
                 │ GitOps Operator│──────────────┐
                 │  (Argo / Flux) │              │
                 └───────┬───────┘               ▼
                         │ apply / prune   ┌─────────────┐
                         └────────────────▶│  Cluster    │
                                           │ live state  │
                         drift? ◀──compare─┤             │
                                           └─────────────┘

Repo layout (one option):
  apps/<name>/base/          shared manifests
  apps/<name>/overlays/dev/  env-specific patches
  apps/<name>/overlays/prod/
  secrets/  -> SealedSecrets / SOPS-encrypted only

Non-functional targets:
  sync latency   < 3 min from merge to converged
  drift MTTR     auto-reverted or alerted < 5 min
  auditability   100% of changes traceable to a commit
```

## Stretch Goals

- Add progressive delivery (Argo Rollouts / Flagger) so promotions are canaried automatically.
- Wire in policy-as-code (OPA/Gatekeeper or Kyverno) as a merge gate and an admission gate.
- Support multiple clusters from one control repo with per-cluster targeting.
- Add notifications and a deploy dashboard so the team sees sync status without opening the CLI.

## Definition of Done

- [ ] A merge deploys automatically; no human runs `kubectl apply` in the happy path.
- [ ] Manual drift is detected and either reverted or clearly alerted.
- [ ] Secrets are encrypted or externalized — nothing sensitive is in plaintext in Git.
- [ ] Multiple environments are promoted through an explicit, reviewable flow.
- [ ] A `git revert` demonstrably rolls the cluster back.

## Common Pitfalls

- Committing plaintext secrets "temporarily" — they live forever in history.
- One giant repo with no environment separation, so a dev change can reach prod.
- Disabling auto-sync/prune to "be safe," which quietly reintroduces manual drift.
- Treating the operator as fire-and-forget and never watching for failed syncs.
- Approval gates so heavy that people bypass GitOps entirely for "urgent" fixes.

## Resources

- [Argo CD documentation](https://argo-cd.readthedocs.io/) — declarative GitOps for Kubernetes.
- [Flux documentation](https://fluxcd.io/flux/) — the CNCF GitOps toolkit.
- [OpenGitOps principles](https://opengitops.dev/) — the four core GitOps principles.
- [SOPS](https://github.com/getsops/sops) — encrypting secrets for safe storage in Git.

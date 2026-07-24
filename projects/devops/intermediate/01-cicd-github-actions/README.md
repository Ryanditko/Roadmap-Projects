# CI/CD Pipeline (GitHub Actions)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Wire up a full continuous integration and delivery pipeline for a small application using GitHub Actions. Every push should lint, test, and build the code; every merge to the main branch should produce a versioned artifact and promote it through staging and then production behind a manual approval gate. The goal is not to ship one workflow file, but to feel how a pipeline turns "it works on my machine" into a repeatable, gated, auditable path to release — including the escape hatch of rolling back when a deploy goes wrong.

## Prerequisites

- A GitHub repository with an app that has a test suite and a build step
- Comfort with YAML and reading command exit codes
- Basic understanding of environment separation (staging vs production)
- A deploy target you can reach from a runner (container registry, static host, or SSH box)

## Learning Objectives

By the end, you should be able to:

- Structure a multi-job workflow with dependencies between jobs
- Cache dependencies and share build artifacts between jobs
- Gate production deploys behind environments and required reviewers
- Inject secrets safely without leaking them into logs
- Trigger a rollback to a previously known-good artifact

## Functional Requirements

1. Every push to any branch must run lint and the full test suite; a failure must fail the run.
2. Merges to `main` must build a versioned, immutable artifact tagged with the commit SHA.
3. The pipeline must deploy automatically to staging after a successful build.
4. Production deploys must require a manual approval from a designated reviewer.
5. Secrets must be sourced from GitHub encrypted secrets, never hard-coded.
6. A documented, one-action path must exist to redeploy a prior artifact (rollback).
7. Each stage must report clear pass/fail status back to the pull request or commit.

## Suggested Milestones

1. **Milestone 1 — CI:** Lint + test on every push, with dependency caching.
2. **Milestone 2 — Build & artifact:** Produce a SHA-tagged artifact and upload it once, reuse it downstream.
3. **Milestone 3 — Deploy & gate:** Auto-deploy staging, gate production with an approval and a rollback path.

## Data & Interface Sketch

```text
Trigger events:
  push (any branch)       -> lint, test
  push to main            -> lint, test, build, deploy-staging
  environment: production -> manual approval -> deploy-production

Job graph:
  lint ─┐
        ├─> build ─> deploy-staging ─(approval)─> deploy-production
  test ─┘

Artifact naming:  app-<git-sha>   (immutable, never overwritten)
Secrets:          REGISTRY_TOKEN, DEPLOY_KEY   (from repo/env secrets)
Rollback:         re-run deploy job pinned to a previous app-<sha>
```

## Stretch Goals

- Add a matrix build to test across multiple runtime versions in parallel.
- Post deploy status and the artifact version to a Slack or Discord channel.
- Add a smoke-test job that hits a health endpoint after staging deploy and blocks promotion on failure.
- Cut releases automatically with semantic version tags on merge.

## Definition of Done

- [ ] A failing test blocks the merge and is visible on the pull request.
- [ ] Artifacts are tagged by commit SHA and never mutated after build.
- [ ] Production deploy cannot proceed without an explicit human approval.
- [ ] No secret value appears in any workflow log.
- [ ] Rolling back to the previous artifact is a documented, repeatable action.

## Common Pitfalls

- Rebuilding the artifact separately for staging and production, so the two environments run different bytes.
- Echoing environment variables for debugging and leaking a secret into public logs.
- Skipping caching, making every run slow enough that people start bypassing CI.
- Treating a green pipeline as "deployed and healthy" without any post-deploy verification.
- Having no rollback plan, so a bad deploy turns into a frantic hotfix under pressure.

## Resources

- [GitHub Actions documentation](https://docs.github.com/en/actions) — workflows, jobs, and triggers from the source.
- [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) — approval gates and protection rules.
- [Encrypted secrets in Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) — the safe way to handle credentials.
- [Martin Fowler: Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html) — the principles behind the practice.
</content>

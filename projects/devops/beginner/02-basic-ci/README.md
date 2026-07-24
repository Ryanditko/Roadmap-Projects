# Basic CI Pipeline

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Beginner · **Estimated time:** 3–6 hours

## Overview

Wire up a continuous integration pipeline that runs on its own every time you push. It checks out the repository, installs dependencies, runs the linter, executes the tests, and reports pass or fail — turning "did I break something?" from a manual chore into an automatic gate. This is the first line of defense in any modern project: a red check on a pull request stops a bug before anyone else sees it. You will pick a CI platform, express the workflow as declarative stages, and learn why a fast, cache-aware pipeline is the difference between a check people trust and one they route around.

## Prerequisites

- A repository with tests you can run locally ([Dockerize a Simple App](../01-dockerize-simple-app/) pairs well)
- A Git host with CI (GitHub Actions, GitLab CI, or similar)
- The exact local commands to install deps, lint, and test
- Basic YAML familiarity

## Learning Objectives

By the end, you should be able to:

- Express a build/test workflow as declarative pipeline stages
- Trigger a pipeline automatically on push and pull request
- Cache dependencies to keep runs fast
- Fail the pipeline correctly when a lint or test step fails
- Surface pipeline status back onto the pull request

## Functional Requirements

1. The pipeline must run automatically on every push and pull request to the main branch.
2. It must check out code, install dependencies, lint, and test as distinct steps.
3. A failing lint or test step must fail the whole pipeline with a non-zero exit.
4. Dependency installation must use caching so unchanged dependencies are not re-downloaded each run.
5. The pass/fail result must be visible on the pull request.
6. The pipeline configuration must live in the repository as version-controlled code.
7. The pipeline must run on a clean environment, assuming nothing from the developer's machine.

## Suggested Milestones

1. **Milestone 1 — Hello pipeline:** Add a config that checks out code and prints toolchain versions on push.
2. **Milestone 2 — Build & test:** Add install, lint, and test steps; make a broken test turn the run red.
3. **Milestone 3 — Speed & signal:** Add dependency caching and a status badge; confirm status appears on PRs.

## Data & Interface Sketch

```text
Repo layout
  .github/workflows/ci.yml   (or .gitlab-ci.yml)
  <project source + tests>

Pipeline stages (structure, not full YAML)
  trigger: on push + pull_request to main
  job "build-and-test":
    step: checkout
    step: setup runtime (pinned version)
    step: restore dependency cache   (key = hash of lockfile)
    step: install dependencies
    step: lint       -> non-zero exit fails job
    step: test       -> non-zero exit fails job
    step: save dependency cache

Status surface
  PR check: ci / build-and-test  ->  passing | failing
  README badge: ![CI](https://github.com/OWNER/REPO/actions/workflows/ci.yml/badge.svg)
```

## Stretch Goals

- Run the test job across a matrix of runtime versions or operating systems.
- Split lint and test into parallel jobs and observe the wall-clock speedup.
- Upload test reports or coverage as build artifacts.
- Require the CI check to pass before a pull request can merge (branch protection).

## Definition of Done

- [ ] Pushing a commit triggers the pipeline with no manual action.
- [ ] A deliberately broken test turns the run red and blocks the PR.
- [ ] A second run with unchanged dependencies is noticeably faster thanks to the cache.
- [ ] The pipeline config is committed to the repository.
- [ ] A status badge or PR check reflects the latest run.

## Common Pitfalls

- Relying on tools already on your laptop that the clean CI runner does not have.
- Swallowing a step's exit code (piping through a command that always returns 0), so failures look green.
- Caching on a key that never changes (stale deps) or one that always changes (never hits).
- Storing secrets in the workflow file instead of the platform's encrypted secrets store.

## Resources

- [GitHub Actions: About workflows](https://docs.github.com/en/actions/using-workflows/about-workflows) — triggers, jobs, and steps.
- [GitLab CI/CD documentation](https://docs.gitlab.com/ee/ci/) — pipeline stages and configuration.
- [GitHub Actions: Caching dependencies](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows) — cache keys done right.
- [Martin Fowler: Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html) — the practice behind the tooling.

# Platform Engineering Toolkit

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Advanced · **Estimated time:** 1–2 weeks

## Overview

Build an internal developer platform (IDP) that lets a product engineer go from "I need a new service" to a running, observable, policy-compliant deployment without filing a ticket or learning the full depth of your infrastructure. You will design golden-path templates, a self-service interface (portal, CLI, or Git-based), and the automation behind it that provisions infrastructure, wires in observability and secrets, and enforces policy — all while keeping the guardrails invisible until someone hits one. The real challenge is product thinking applied to infrastructure: the right level of abstraction, escape hatches for the 10% of cases the golden path doesn't cover, and adoption driven by making the paved road genuinely faster than doing it by hand. A platform nobody uses is just more infrastructure to maintain.

## Prerequisites

- Solid grounding in Kubernetes, IaC, and CI/CD (this project composes them)
- Experience deploying a service end-to-end at least once manually
- Familiarity with templating (Helm/Kustomize) and policy-as-code concepts
- Understanding of the developer workflow you intend to abstract

## Learning Objectives

By the end, you should be able to:

- Design golden-path templates that encode best practice by default
- Build a self-service interface that provisions a service without manual ops
- Enforce policy-as-code so compliance is automatic, not a review step
- Provide escape hatches for cases the paved road doesn't cover
- Measure platform adoption and developer time-to-first-deploy

## Functional Requirements

1. A developer must be able to create a new service from a template via self-service, no ticket.
2. Provisioning must wire in observability, secrets, and CI/CD automatically.
3. Policy-as-code must enforce standards (naming, resource limits, security) at creation and deploy.
4. The golden path must have a documented escape hatch for non-standard needs.
5. The platform must expose the status of a developer's services (deploy, health) in one place.
6. Onboarding a new service must be measurably faster than the manual baseline.
7. The platform's own configuration must be versioned and reproducible.

## Suggested Milestones

1. **Milestone 1 — Golden-path template:** Define a template that produces a compliant, observable service.
2. **Milestone 2 — Self-service interface:** Let a developer instantiate the template (portal, CLI, or Git PR) without ops.
3. **Milestone 3 — Policy & guardrails:** Enforce policy-as-code and surface violations with clear feedback.
4. **Milestone 4 — Visibility & adoption:** Add a service catalog/status view and measure time-to-first-deploy vs. baseline.

## Data & Interface Sketch

```text
   developer
      │  "new service: checkout"  (portal / CLI / Git PR)
      ▼
   ┌──────────────────┐   golden-path template
   │ Self-service      │   (repo + CI + manifests + observability wired)
   │ interface         │
   └────────┬─────────┘
            │ orchestrate
            ▼
   ┌──────────────────┐    policy-as-code gate (OPA/Kyverno)
   │ Platform          │───▶ enforce: naming, limits, security
   │ orchestrator      │    pass -> provision ; fail -> clear feedback
   └───┬────────┬──────┘
       ▼        ▼            ▼             ▼
   infra     observability  secrets     CI/CD pipeline
   (IaC)     wired-in       injected    created
       └────────┴────────────┴─────────────┘
                     ▼
              ┌───────────────┐
              │ Service catalog│  status: deploys, health, owner
              └───────────────┘

Escape hatch: golden path covers ~90%; document how to extend/opt-out for the rest.

Non-functional targets:
  time-to-first-deploy  minutes, not days (measure vs. manual baseline)
  policy compliance     100% enforced at creation, not post-hoc review
  adoption              % of new services using the paved road, tracked
```

## Stretch Goals

- Add multiple golden paths (stateless service, cron job, data pipeline) with shared building blocks.
- Integrate the platform with a real developer portal (Backstage) and a service catalog.
- Add cost and security scorecards per service so owners see their posture.
- Support ephemeral preview environments spun up per pull request via the same templates.

## Definition of Done

- [ ] A developer creates a compliant, observable service via self-service with no ticket.
- [ ] Observability, secrets, and CI/CD are wired in automatically.
- [ ] Policy-as-code enforces standards at creation and deploy with clear feedback.
- [ ] A documented escape hatch exists for non-standard cases.
- [ ] Time-to-first-deploy is measured and beats the manual baseline; adoption is tracked.

## Common Pitfalls

- Over-abstracting so the platform can't express the 10% of real cases, forcing shadow tooling.
- Building the platform with no user research, then wondering why nobody adopts it.
- Making the paved road slower or more confusing than doing it by hand — adoption dies.
- Policy that blocks with cryptic errors, training developers to route around the platform.
- Treating the platform as a one-time project instead of a product with users and a roadmap.

## Resources

- [CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/) — what a platform is and how to reason about it.
- [Backstage documentation](https://backstage.io/docs/overview/what-is-backstage/) — an open developer portal framework.
- [team-topologies.com](https://teamtopologies.com/) — platform teams and cognitive load.
- [Google SRE Workbook: Engagement](https://sre.google/workbook/engagement-model/) — platform-as-product and adoption thinking.

# Infrastructure as Code (Terraform)

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Provision a small but realistic piece of infrastructure — say a network, a compute instance or container service, and a managed database — entirely from Terraform code, with no clicking in a cloud console. The point is not to memorize resource names but to internalize the IaC loop: write configuration, `plan` to preview the diff, `apply` to converge reality to the desired state, and store state remotely so a team can collaborate. Above all, your configuration must be idempotent: running `apply` twice in a row with no code changes must produce no changes.

## Prerequisites

- A cloud account (AWS, GCP, Azure) or a local provider like Docker
- Terraform (or OpenTofu) installed and authenticated to the provider
- Understanding of the resources you intend to create (network, compute, storage)
- Comfort with the command line and reading a diff

## Learning Objectives

By the end, you should be able to:

- Declare resources and understand the plan/apply/destroy lifecycle
- Parameterize configuration with input variables and expose results as outputs
- Factor repeated resources into a reusable module
- Store state remotely with locking so concurrent applies can't corrupt it
- Detect and reconcile drift between code and real infrastructure

## Functional Requirements

1. All infrastructure must be defined in version-controlled Terraform files — no manual console changes.
2. `terraform plan` must show an accurate preview before any change is applied.
3. Configuration must be idempotent: a second `apply` with no code change reports zero changes.
4. Environment-specific values (region, sizes, names) must be variables, not hard-coded.
5. At least one group of resources must be extracted into a reusable module.
6. State must be stored in a remote backend with locking enabled.
7. `terraform destroy` must cleanly tear down everything the configuration created.

## Suggested Milestones

1. **Milestone 1 — First apply:** Define a couple of resources, run plan/apply, verify they exist.
2. **Milestone 2 — Variables & modules:** Parameterize with variables/outputs and extract a module.
3. **Milestone 3 — Remote state & drift:** Move state to a remote backend with locking; change a resource by hand and reconcile the drift.

## Data & Interface Sketch

```text
Repository layout (structure only):
  main.tf          root composition, calls module(s)
  variables.tf     typed inputs (region, instance_size, env)
  outputs.tf       exported values (ip, db_endpoint)
  backend.tf       remote state config (bucket + lock table)
  modules/network/ reusable module (vpc, subnets, ...)

Lifecycle:
  write .tf  ->  terraform plan  ->  review diff  ->  terraform apply
                                                        |
  drift (manual change)  <───────  terraform plan shows delta  <──┘

State: remote backend, locked during apply (no concurrent writers)
```

## Stretch Goals

- Add multiple environments via workspaces or separate variable files.
- Run `terraform validate` and `fmt` in CI, and post the `plan` output on pull requests.
- Add a policy check (OPA/Sentinel or tflint) that blocks disallowed resource configs.
- Import an existing, manually-created resource into Terraform state.

## Definition of Done

- [ ] A fresh clone can `plan`/`apply` and reproduce the infrastructure from scratch.
- [ ] A second `apply` with no changes reports "No changes."
- [ ] No credentials or environment values are hard-coded in the .tf files.
- [ ] State lives in a remote backend and is locked during apply.
- [ ] `terraform destroy` leaves no orphaned resources behind.

## Common Pitfalls

- Editing infrastructure by hand after applying, causing drift that the next apply silently reverts or fights.
- Committing the local `terraform.tfstate` (with secrets) to git instead of using a remote backend.
- Building non-idempotent config (e.g. timestamps in names) so every apply shows spurious changes.
- Skipping `plan` and applying blind, then discovering it wanted to destroy the database.
- Hard-coding region/account values, making the config impossible to reuse across environments.

## Resources

- [Terraform documentation](https://developer.hashicorp.com/terraform/docs) — language, workflow, and providers.
- [Terraform state](https://developer.hashicorp.com/terraform/language/state) — why state exists and how remote backends work.
- [Standard module structure](https://developer.hashicorp.com/terraform/language/modules/develop/structure) — how to lay out a reusable module.
- [What is Infrastructure as Code?](https://developer.hashicorp.com/terraform/intro) — the concept and its benefits.
</content>

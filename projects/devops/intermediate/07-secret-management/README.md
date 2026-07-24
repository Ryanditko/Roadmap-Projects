# Secret Management System

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** DevOps · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Stop scattering passwords, API keys, and tokens across `.env` files and CI variables. Build a workflow around a real secret manager — HashiCorp Vault, AWS Secrets Manager, or an equivalent — that stores secrets encrypted, controls who can read them, keeps an audit trail of every access, and rotates them without downtime. Then integrate an application so it fetches secrets at runtime instead of embedding them. The goal is to understand the full lifecycle of a secret: store, access-control, deliver, audit, rotate, and revoke.

## Prerequisites

- A secret manager you can run or access (Vault dev server, cloud Secrets Manager)
- An application that needs at least one secret (a DB password or API key)
- Understanding of encryption at rest vs in transit, and of access policies
- Familiarity with environment variables and how apps read configuration

## Learning Objectives

By the end, you should be able to:

- Store secrets encrypted at rest with access mediated by policy, not file permissions
- Grant least-privilege read access scoped per application or identity
- Deliver secrets to an app at runtime without writing them to disk or an image
- Produce an audit log answering "who read which secret, when"
- Rotate a secret and have consumers pick up the new value without a hard outage

## Functional Requirements

1. Secrets must be stored encrypted at rest, never in plaintext files or the app image.
2. Access must be governed by policies granting least privilege per identity.
3. An application must retrieve its secret at runtime from the manager, not from a baked-in value.
4. Every secret access must be recorded in an audit log with identity and timestamp.
5. A secret must be rotatable, and consumers must obtain the new value without manual redeploy where possible.
6. A revoked or expired credential must stop working after revocation.
7. No secret value may appear in application logs or process listings.

## Suggested Milestones

1. **Milestone 1 — Store & read:** Put a secret in the manager and read it back with a scoped token.
2. **Milestone 2 — Integrate & audit:** Have an app fetch the secret at runtime; verify the access shows in the audit log.
3. **Milestone 3 — Rotate & revoke:** Rotate the secret and confirm consumers pick it up; revoke a token and confirm it fails.

## Data & Interface Sketch

```text
Lifecycle of a secret:
  create ─> store(encrypted) ─> policy(who can read) ─> deliver(runtime)
     ^                                                      |
     └──────────── rotate / revoke <── audit(who,what,when)─┘

Access model (conceptual):
  identity (app/role) ── authenticates ──> manager
                         └── policy: read secret/app/db-password only
  app requests secret at boot / on lease renewal, holds in memory only

Rotation:
  new version written -> old version deprecated -> consumers re-read -> old revoked
```

## Stretch Goals

- Use dynamic secrets: have the manager generate short-lived DB credentials on demand.
- Add automatic lease renewal so long-running apps never hold a stale credential.
- Integrate secret injection into Kubernetes (CSI driver or an init sidecar).
- Alert when a secret is accessed by an unexpected identity or outside a time window.

## Definition of Done

- [ ] No secret is stored in plaintext in the repo, image, or a committed `.env`.
- [ ] An app reads its secret at runtime and it never touches disk.
- [ ] The audit log shows each access with identity and timestamp.
- [ ] Rotating a secret does not require editing application code.
- [ ] A revoked credential is rejected on its next use.

## Common Pitfalls

- "Solving" secrets by moving them from code into a committed `.env` — still plaintext in git history.
- Logging the secret at startup "just to confirm it loaded", leaking it into log storage.
- Granting one broad policy to everything, so a single leaked token reads every secret.
- Rotating the stored value but never signaling consumers, so they keep using the old one until they crash.
- Treating the audit log as optional, then being unable to answer "was this secret exposed?".

## Resources

- [HashiCorp Vault documentation](https://developer.hashicorp.com/vault/docs) — storage, policies, dynamic secrets, and audit.
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) — managed storage and rotation.
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) — practices and anti-patterns.
- [NIST SP 800-57: Key Management](https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final) — foundations of key and secret handling.
</content>

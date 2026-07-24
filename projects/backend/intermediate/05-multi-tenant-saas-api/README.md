# Multi-Tenant SaaS API

> 🌐 **English** · [Português](./README.pt-BR.md)

**Domain:** Backend · **Level:** Intermediate · **Estimated time:** 1–2 days

## Overview

Most SaaS products serve many customers — tenants — from a single running application, while promising each of them that their data is completely invisible to the others. This project asks you to build the machinery that makes that promise true. A request arrives, you figure out which tenant it belongs to, you attach that identity to everything downstream, and every query you run is silently scoped so one tenant can never read or write another's rows. The hard part is not the feature list — it is that a single missing filter becomes a data leak, so isolation has to be the default, not something each handler remembers to do.

## Prerequisites

- Comfort building and securing a REST API with authentication ([E-commerce API with JWT Authentication](../01-ecommerce-api-jwt/) is a good precursor)
- A relational database and working knowledge of schemas, indexes, and `WHERE` clauses
- Understanding of middleware / request-lifecycle hooks in your framework
- Familiarity with how request context is propagated (thread-locals, async context, or explicit passing)

## Learning Objectives

By the end, you should be able to:

- Resolve a tenant from a subdomain, header, or token and reject requests that resolve to none
- Compare isolation models — shared schema with a tenant column, schema-per-tenant, database-per-tenant — and justify a choice
- Enforce data isolation so that forgetting a filter fails safe rather than leaking
- Propagate tenant context cleanly from the edge down to the data layer
- Provision and de-provision tenants as a first-class lifecycle
- Toggle features and quotas per tenant without redeploying

## Functional Requirements

1. The system must resolve every authenticated request to exactly one tenant (via subdomain, header, or a claim inside the token) and return 400/401 when it cannot.
2. A tenant must never be able to read, update, or delete another tenant's records, even by guessing IDs.
3. Data access must be scoped centrally so a new endpoint is isolated by default without per-handler code.
4. The system must expose an onboarding endpoint that provisions a new tenant and its initial admin.
5. The system must support per-tenant feature flags that change behaviour without a deploy.
6. The system must record per-tenant usage (requests, storage, or a domain metric) for later billing.
7. Tenant deletion must remove or irreversibly anonymize that tenant's data.

## Suggested Milestones

1. **Milestone 1 — Tenant resolution:** Extract the tenant from the request, load it, and reject unknown or inactive tenants in middleware.
2. **Milestone 2 — Enforced isolation:** Choose an isolation model and route all reads/writes through a scoped layer; prove a cross-tenant read is impossible.
3. **Milestone 3 — Lifecycle & flags:** Add tenant onboarding, per-tenant feature flags, usage metering, and deletion.

## Data & Interface Sketch

```text
Tenant
  id:        uuid
  slug:      string   (subdomain, e.g. "acme")
  status:    enum(active, suspended, deleting)
  plan:      string
  features:  map<string, bool>
  createdAt: ISO-8601 string

TenantScopedRecord (every business table)
  id:        uuid
  tenantId:  uuid   (indexed, never client-supplied)
  ...domain fields

Resolution: Host "acme.api.example.com" | header X-Tenant-Id | JWT claim "tid"
            -> load Tenant -> attach to request context

POST /tenants               -> 201 { id, slug }           (provision)
GET  /projects              -> 200 [ ... ]                 (auto-scoped to tenant)
GET  /admin/usage           -> 200 { requests, storageMb } (per tenant)
DELETE /tenants/{id}        -> 202  (async purge)

Isolation options: shared schema + tenantId filter (+ Postgres RLS),
                   schema-per-tenant, database-per-tenant
```

## Stretch Goals

- Enforce isolation in the database itself with PostgreSQL Row-Level Security so a missing filter cannot leak.
- Add per-tenant rate limiting and quota enforcement tied to the plan.
- Provide a self-service data export ("download all my data") per tenant.
- Support a "noisy neighbour" safeguard: cap the resources any one tenant can consume.

## Definition of Done

- [ ] Every request without a resolvable, active tenant is rejected before reaching business logic.
- [ ] A test proves tenant A cannot fetch tenant B's record even with a valid ID.
- [ ] Adding a new endpoint requires no extra code to be tenant-isolated.
- [ ] A tenant can be provisioned and deleted end to end, with deletion removing its data.
- [ ] At least one feature flag changes behaviour for one tenant and not others.

## Common Pitfalls

- Relying on each handler to remember the `tenantId` filter — one forgotten `WHERE` is a breach. Scope centrally.
- Trusting a client-supplied tenant ID in the body while the token says something else; always derive it server-side.
- Leaking tenant existence through error messages or ID enumeration (404 vs 403 differences).
- Caching without namespacing keys by tenant, so one tenant's cached response is served to another.
- Running migrations against shared tables without considering tenants that are mid-provisioning.

## Resources

- [Microsoft: Multi-tenant SaaS database tenancy patterns](https://learn.microsoft.com/en-us/azure/azure-sql/database/saas-tenancy-app-design-patterns) — the canonical comparison of isolation models.
- [PostgreSQL: Row Security Policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html) — enforce tenant isolation in the database itself.
- [OWASP: Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) — preventing the broken-object-level-authorization class of leak.
- [Stripe: Designing robust and predictable APIs with idempotency](https://stripe.com/blog/idempotency) — useful context for tenant provisioning workflows.

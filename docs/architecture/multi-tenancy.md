# Multi-Tenancy

> The tenant-isolation model every Zodize product MUST implement. Builds on
> [`overview.md`](./overview.md) and is the architectural foundation that
> [`../security/rbac-permissions.md`](../security/rbac-permissions.md) and
> [`../security/audit-logging.md`](../security/audit-logging.md) assume.

## Model: single database, `tenant_id` scoping

Every Zodize product uses a **single shared database with a `tenant_id`
column** on every tenant-owned table, rather than database-per-tenant. This
is the default for every product, including financial-grade and
healthcare-grade products; a product MUST NOT switch to database-per-tenant
without an ADR justifying the specific regulatory or scale requirement that
demands it.

Rationale:

- **Operational simplicity at Zodize's scale.** One schema, one migration
  run, one connection pool to manage per product, regardless of tenant
  count. Database-per-tenant multiplies migration and monitoring overhead
  linearly with customer count — untenable for ~20 products each onboarding
  potentially hundreds of tenants.
- **Cost.** Shared infrastructure amortizes compute and connection overhead
  across tenants; database-per-tenant means idle capacity reserved per
  small customer.
- **Cross-tenant platform features.** ZodiCore's billing, plugin
  marketplace, and platform-level analytics need to query across tenants
  (e.g., "which tenants are on the enterprise plan"); this is a single query
  in the shared model and a fan-out across N databases otherwise.
- **The isolation risk is manageable with discipline.** The primary
  objection to shared-database multi-tenancy is data leakage across
  tenants. Zodize treats this as a solved problem via mandatory global
  query scoping and automated cross-tenant leakage testing (below), which
  is a stronger and more auditable guarantee than "the tenants are in
  different databases" alone — a query bug can leak whether isolation is
  physical or logical if authorization is also missing.
- **Exception path exists.** A specific enterprise or government contract
  requiring dedicated infrastructure (e.g. ZodiGov, a sovereign deployment)
  is handled as a **dedicated single-tenant deployment** of the entire
  product stack (its own database, its own application nodes), not as an
  in-app database-per-tenant feature. This is an infrastructure decision
  made per contract, documented in that tenant's provisioning record, not a
  core architectural pattern every product must build for.

## Global query scopes for tenant isolation

- Every Eloquent model representing tenant-owned data MUST use a global
  scope (`BelongsToTenant`, provided by `zodize/core`) that automatically
  adds `WHERE tenant_id = ?` to every query built against that model,
  sourced from the currently resolved tenant context — never from a
  client-supplied parameter.
- The tenant context is resolved once per request, early in the middleware
  stack (before any controller or Policy runs), from the authenticated
  user's session/token — not re-derived per query and not trusted from a
  request body or query string field.
- Any query that must intentionally cross tenants (ZodiCore platform admin
  tooling, billing reconciliation jobs) MUST explicitly opt out via
  `Model::withoutTenantScope()` or equivalent, and that call site MUST be
  restricted to platform-admin-only code paths, never reachable from a
  tenant-facing controller.
- Every tenant-owned table MUST have `tenant_id` as part of a composite
  index with its other common lookup columns (e.g.
  `(tenant_id, created_at)`, `(tenant_id, status)`) — an unindexed
  `tenant_id` filter on a large table is a performance defect, not just a
  correctness one; see
  [`../quality/performance-standards.md`](../quality/performance-standards.md).

## Tenant identification

- **Subdomain routing** is the default and MUST be supported by every
  product: `{tenant-slug}.{product}.zodize.com` resolves the tenant from the
  subdomain in a routing middleware before any controller runs.
- **Custom domain** MUST be supported for enterprise-tier tenants: a tenant
  maps a domain they own (via CNAME) to the product, verified by a DNS TXT
  record challenge, after which the product resolves tenant identity from
  the `Host` header against a `tenant_domains` lookup table (cached, not a
  query on every request — see
  [`caching-queues-events.md`](./caching-queues-events.md#caching-strategy)).
- A request whose subdomain or custom domain does not resolve to an active
  tenant MUST return a `404` (or a friendly "workspace not found" page for
  browser requests), never fall through to a default tenant.

## Cross-tenant data leakage prevention

- Every product's automated test suite MUST include a dedicated cross-tenant
  isolation test category (e.g. `tests/Feature/TenantIsolation/`) that, for
  every tenant-owned resource type, asserts a user authenticated as Tenant A
  cannot read, update, delete, or list a record belonging to Tenant B —
  including via direct ID guessing on show/update/delete routes, via
  relationship/nested-resource endpoints, and via search/filter endpoints.
  This is a mandatory CI gate: a product cannot merge a new tenant-owned
  model without an accompanying isolation test, enforced by code review per
  [`../quality/definition-of-done.md`](../quality/definition-of-done.md).
- API responses MUST NOT include another tenant's identifiers even
  incidentally (e.g., an autocomplete endpoint must be tenant-scoped, not
  just its result rendering).
- File storage paths for tenant-owned files MUST be namespaced by
  `tenant_id` (e.g. `tenants/{tenant_id}/uploads/...`) so a storage-layer
  misconfiguration cannot serve one tenant's file to another via a guessed
  path.

## Tenant provisioning and deprovisioning lifecycle

1. **Provisioning**: triggered by signup or by an internal sales
   handoff. Creates the `Tenant` row, seeds the default system roles (see
   [`../security/rbac-permissions.md`](../security/rbac-permissions.md#default-system-roles)),
   creates the Owner user, assigns the declared data residency region (see
   [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md#data-residency-and-multi-region)),
   and provisions a default subdomain. Product-specific seed data (default
   settings, starter templates) runs as a module-registered provisioning
   hook, not hardcoded into the core provisioning flow.
2. **Active**: normal operation. Plan changes, team/branch creation, and
   custom-domain attachment happen without re-provisioning.
3. **Suspension**: triggered by billing failure or a Trust & Safety action.
   The tenant's data is retained and its `status` flips to `suspended`; all
   authenticated routes for that tenant return a "workspace suspended" state
   instead of resolving normally. No data is deleted at this stage.
4. **Deprovisioning**: triggered by tenant-initiated cancellation or
   contract end. Enters a 30-day grace period (configurable per contract)
   during which the tenant can be reactivated by an Owner; after the grace
   period, a scheduled job hard-deletes the tenant's data following the
   retention and deletion rules in
   [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md#data-retention-and-deletion),
   with audit log entries retained per
   [`../security/audit-logging.md`](../security/audit-logging.md#retention-periods).

## Multi-company and multi-branch as nested scoping

Products that need a tenant to represent a customer organization with
multiple legal entities or physical locations (common in ZodiBusiness,
ZodiHotel, ZodiFleet, ZodiBuild) use **Team** as a nested scoping layer
inside the tenant, not a second tenancy system:

- A `teams` table, `tenant_id`-scoped, represents a company or branch.
  Tenant-owned tables that need branch-level segregation add a nullable
  `team_id` alongside `tenant_id`; `team_id IS NULL` means "visible
  tenant-wide" for resources that are intentionally shared across
  branches (e.g. a shared product catalog).
- Role assignments are scoped at the tenant level or the team level, per
  [`../security/rbac-permissions.md`](../security/rbac-permissions.md#core-entities) —
  this is how a Manager ends up restricted to one branch while an Admin sees
  the whole tenant.
- Team-level data still inherits the tenant's `tenant_id` global scope
  first; the team scope is a second, narrower filter applied on top, never
  a replacement for it.

## Related standards

- [`overview.md`](./overview.md)
- [`../security/rbac-permissions.md`](../security/rbac-permissions.md)
- [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md)
- [`caching-queues-events.md`](./caching-queues-events.md)

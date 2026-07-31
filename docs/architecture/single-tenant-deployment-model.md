# Single-Tenant Deployment Model

> Replaces the earlier multi-tenant SaaS model this handbook used to
> specify. Builds on [`overview.md`](./overview.md) and is the
> architectural foundation that
> [`../security/rbac-permissions.md`](../security/rbac-permissions.md) and
> [`../security/audit-logging.md`](../security/audit-logging.md) now assume.

## Model: one buyer, one deployment, one database

Every Zodize product is **single-tenant by construction**: one buyer buys
the source code, deploys it to their own hosting, and imports one database
that belongs entirely to their business. There is no `tenant_id` column,
no tenant-resolution middleware, and no concept of multiple customer
organizations sharing one running instance of the application.

This replaces the shared-database, `tenant_id`-scoped multi-tenancy model
this handbook specified in an earlier draft. That model assumed Zodize
operated every product as a hosted service for many customer organizations
at once. It does not — see
[`overview.md`](./overview.md#the-business-model-this-architecture-serves).
Any reference elsewhere in this handbook to `tenant_id`, tenant scoping,
tenant provisioning/suspension/deprovisioning, subdomain-per-tenant routing,
or a shared platform tenants are provisioned onto is superseded by this
document.

## What "single-tenant" changes in the data model

- No `tenant_id` column on any table. A product's schema models exactly one
  business's data.
- No tenant global query scope. Every Eloquent query already operates within
  the one business's data by construction — there is nothing to scope
  against.
- No cross-tenant isolation test category. The isolation Zodize's earlier
  multi-tenant model tested for (Tenant A cannot see Tenant B's data) is
  structurally impossible to violate, because there is no Tenant B in the
  same running application. See
  [`../development/testing-standards.md`](../development/testing-standards.md)
  for the mandatory test cases that remain relevant (authorization denial,
  not-found) with tenant-isolation cases removed.
- No `tenant_domains` lookup, no subdomain-per-tenant routing. The
  application serves exactly one domain: whichever domain the buyer points
  at their hosting.

## What replaces multi-company / multi-branch scoping

Products that need to represent a buyer's business with multiple legal
entities or physical locations (common in ZodiBusiness, ZodiHotel,
ZodiFleet, ZodiBuild) still use a `companies`/`branches` nested scoping
layer exactly as documented in
[`../development/database-standards.md`](../development/database-standards.md)
and each product's own
[`docs/products/<product>/SPEC.md`](../products/) data model — this was
never tenancy in the SaaS sense and is unaffected by this change. A branch
belongs to a company; a company belongs to the one business that owns the
whole deployed instance. There is still exactly one owning business per
deployment.

## No shared platform service

Earlier drafts of this handbook described a "ZodiCore" platform every other
product depended on at runtime for identity, billing, notifications, and
tenancy. **This does not exist and MUST NOT be built.** Each product:

- Implements its own authentication, authorization, and RBAC in its own
  codebase — inherited from the sanitized base codebase's existing engine
  (guards, custom `Role`/`Permission` models), not from a call to another
  product. See
  [`../security/authentication-authorization.md`](../security/authentication-authorization.md)
  and [`../security/rbac-permissions.md`](../security/rbac-permissions.md).
- Implements its own billing/payment-gateway configuration, wallet ledger,
  and notification dispatch — inherited from the base codebase's existing
  engines. See [`../standards/payment-gateways.md`](../standards/payment-gateways.md)
  and [`../standards/wallet-system.md`](../standards/wallet-system.md).
- Has no runtime dependency on any other Zodize product or on any
  Zodize-operated central service being reachable. A product must be fully
  functional on a buyer's hosting account with zero connectivity to any
  other Zodize system, aside from the third-party services the buyer
  themselves configures (payment gateway APIs, SMS/email providers).
- `ZodiCore` remains in the product catalog as one of the twenty sellable
  products (a general-purpose back-office/ERP starter), stripped of any
  "platform" role — see
  [`../../PRODUCT_CATALOG.md`](../../PRODUCT_CATALOG.md). Its spec is
  rewritten accordingly in
  [`../products/ZodiCore/SPEC.md`](../products/ZodiCore/SPEC.md).

## Licensing and update model

Because each deployment is a standalone codebase the buyer controls, Zodize
does not push updates to a running instance. Product updates are
distributed as a new source-code release the buyer downloads and applies
manually (or via an in-admin "check for update" / update-package flow
documented per product), following
[`../development/versioning-releases.md`](../development/versioning-releases.md).
License validation (where a product requires a purchase code, e.g. via a
CodeCanyon-style licensing check) is a concern for the base codebase's
`GeneralSetting`/licensing flow, configured once by the buyer at install
time — not an ongoing "phone home" dependency required for the product to
function.

## Related standards

- [`overview.md`](./overview.md)
- [`base-codebase-strategy.md`](./base-codebase-strategy.md)
- [`../security/rbac-permissions.md`](../security/rbac-permissions.md)
- [`../security/data-protection-privacy.md`](../security/data-protection-privacy.md)
- [`../standards/admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md)

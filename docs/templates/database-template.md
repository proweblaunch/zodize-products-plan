# Database Template

This document specifies the standard base schema every Zodize product's
database starts with, before any product-specific tables are added. The
conventions these tables follow (naming, indexing, migration discipline) are
defined in [`../development/database-standards.md`](../development/database-standards.md);
the single-tenant deployment model these tables implement is defined in
[`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md).
A product customizes columns beyond the required set and adds its own domain
tables; it does not remove or rename any table or required column listed
here, and it MUST NOT reintroduce a `tenant_id` column or any other
multi-tenant construct.

Every product's database belongs entirely to one buyer's one business — see
[`single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md#model-one-buyer-one-deployment-one-database).
There is no `tenants` table. Products that need to represent a buyer's
business with multiple companies, branches, or locations use the
`companies`/`branches` scoping layer defined at the end of this document,
which is a data-modeling concern within one deployment, not tenancy.

## Base tables

Every product's initial migration set MUST create the following tables,
before any product-specific migration runs.

### `users`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `name`, `email` | `email` unique globally — there is one business per deployment, so there is nothing else to scope uniqueness against. |
| `email_verified_at` | Nullable. |
| `password` | Hashed; see [`../security/authentication-authorization.md`](../security/authentication-authorization.md). |
| `mfa_enabled`, `mfa_secret` (encrypted) | See [authentication-template.md](./authentication-template.md). |
| `last_login_at` | Nullable. |
| `created_at`, `updated_at`, `deleted_at` | Soft deletes required. |

### `roles`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `name`, `slug` | Unique. |
| `is_system` | Boolean; system roles MUST NOT be deletable via UI. |

### `permissions`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `slug` | Unique, format `{resource}.{action}` per [permission-template.md](./permission-template.md). |
| `module` | Which module registered it. |
| `description` | Human-readable, shown in admin permission UI. |

### `role_user` (pivot)

| Column | Notes |
|---|---|
| `role_id`, `user_id` | Composite key. |

A separate `permission_role` pivot follows the same pattern and is assumed
present alongside `role_user`.

### `audit_logs`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `actor_id` | Nullable (system-initiated events). |
| `action` | Machine-readable event slug. |
| `subject_type`, `subject_id` | Polymorphic target. |
| `metadata` | JSON, event-specific detail. |
| `ip_address`, `user_agent` | Captured at time of action. |
| `created_at` | No `updated_at` — audit rows are immutable. |

Full requirements for what MUST be logged live in
[`../security/audit-logging.md`](../security/audit-logging.md); this table
is the storage contract every product uses.

### `notifications`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `notifiable_type`, `notifiable_id` | Polymorphic recipient. |
| `type` | Notification class/slug. |
| `channel` | `mail` \| `sms` \| `in_app` \| `push`. |
| `data` | JSON payload. |
| `read_at` | Nullable, for in-app notifications. |
| `created_at` | |

### `settings`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `key` | Unique — one deployment, one settings namespace. |
| `value` | JSON. |
| `updated_at` | |

### `api_tokens`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `user_id` | Owner. |
| `name` | Human-readable label. |
| `token_hash` | Never store the raw token. |
| `abilities` | JSON array of permission slugs the token is scoped to. |
| `last_used_at`, `expires_at` | Nullable. |

### `webhooks`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `url` | Destination. |
| `events` | JSON array of subscribed event slugs. |
| `secret` | Encrypted, used for HMAC signing per [`../development/api-standards.md`](../development/api-standards.md). |
| `is_active` | Boolean. |
| `last_triggered_at`, `last_response_code` | Nullable, for delivery health. |

### `feature_flags`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `key` | Unique slug. |
| `description` | |
| `is_enabled` | Boolean — a single on/off switch for the whole deployment. There is no per-tenant override map, because there is no second tenant to vary the value for. |

## Multi-company / multi-branch scoping (not tenancy)

Only products whose own [`SPEC.md`](../products/) calls for multi-company or
multi-branch operation (e.g. ZodiBusiness, ZodiHotel, ZodiFleet, ZodiBuild)
add this scoping layer; it is not part of the mandatory base set above. Where
present, it follows
[`../standards/localization-i18n.md`](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)
and
[`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md#what-replaces-multi-company--multi-branch-scoping):

- **`companies`** — one row per legal entity the buyer's business operates;
  `id` (ULID), `name`, `settings` (JSON), timestamps.
- **`branches`** — one row per physical location/branch; `id` (ULID),
  `company_id` (FK to `companies`), `name`, `settings` (JSON), timestamps.
- Every scoped business record (accounts, orders, staff assignments,
  inventory) carries a `company_id` and/or `branch_id` foreign key, never an
  implicit scope inferred from session state alone.

This is explicitly not the `tenants`/`tenant_id` pattern this template used
in an earlier draft: a company/branch always belongs to the one business
that owns the entire deployed instance, never to a different customer
sharing the same running application.

## What the base codebase provides vs. what a product customizes

The sanitized base codebase (see
[`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md))
provides the migrations, Eloquent models, and admin screens backing most of
the tables above (`users`, `roles`/`permissions` via its own RBAC engine,
`settings` via `GeneralSetting`, and more) as part of its inherited admin
engine — see
[`../standards/admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md)
for the full inventory. A product's own migrations extend this base set
directly in the product's codebase; there is no separate platform codebase
these migrations are pulled from at runtime.

A product customizes: additional columns on `settings` for product-specific
configuration, the `companies`/`branches` scoping tables where its own
`SPEC.md` calls for them, and all product-domain tables layered on top. A
product MUST NOT modify the inherited base migrations directly —
product-specific columns on a base table are added via a new migration in
the product's own module, never by editing the base codebase's original
migration files (see
[`../architecture/product-genericization-checklist.md`](../architecture/product-genericization-checklist.md)).

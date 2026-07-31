# Database Template

This document specifies the standard base schema every Zodize product's
database starts with, before any product-specific tables are added. The
conventions these tables follow (naming, indexing, migration discipline) are
defined in [`../development/database-standards.md`](../development/database-standards.md);
the tenancy model these tables implement is defined in
[`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md). A
product customizes columns beyond the required set and adds its own domain
tables; it does not remove or rename any table or required column listed
here.

## Base tables

Every product's initial migration set MUST create the following tables,
before any product-specific migration runs.

### `tenants`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `name` | Display name. |
| `slug` | Unique, used in tenant-scoped URLs where applicable. |
| `plan` | FK/reference to ZodiCore billing plan. |
| `status` | `trial` \| `active` \| `suspended` \| `churned`. |
| `trial_ends_at` | Nullable. |
| `settings` | JSON, tenant-level configuration (see `settings` table below for the general pattern this may delegate to). |
| `created_at`, `updated_at` | Standard timestamps. |

### `users`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `tenant_id` | FK to `tenants`, nullable only for Zodize-staff accounts. |
| `name`, `email` | `email` unique per `tenant_id`, not globally unique. |
| `email_verified_at` | Nullable. |
| `password` | Hashed; see [`../security/authentication-authorization.md`](../security/authentication-authorization.md). |
| `mfa_enabled`, `mfa_secret` (encrypted) | See [authentication-template.md](./authentication-template.md). |
| `last_login_at` | Nullable. |
| `created_at`, `updated_at`, `deleted_at` | Soft deletes required. |

### `roles`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `tenant_id` | Nullable for system-defined roles shared across tenants. |
| `name`, `slug` | Unique per `tenant_id`. |
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
| `tenant_id` | Denormalized for query performance and tenant-isolation enforcement. |

A separate `permission_role` pivot follows the same pattern and is assumed
present alongside `role_user`.

### `audit_logs`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `tenant_id` | Nullable for cross-tenant/system-level events. |
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
| `tenant_id`, `notifiable_type`, `notifiable_id` | Polymorphic recipient. |
| `type` | Notification class/slug. |
| `channel` | `mail` \| `sms` \| `in_app` \| `push`. |
| `data` | JSON payload. |
| `read_at` | Nullable, for in-app notifications. |
| `created_at` | |

### `settings`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `tenant_id` | Nullable for global/system settings. |
| `key` | Unique per `tenant_id`. |
| `value` | JSON. |
| `updated_at` | |

### `api_tokens`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `tenant_id`, `user_id` | Owner. |
| `name` | Human-readable label. |
| `token_hash` | Never store the raw token. |
| `abilities` | JSON array of permission slugs the token is scoped to. |
| `last_used_at`, `expires_at` | Nullable. |

### `webhooks`

| Column | Notes |
|---|---|
| `id` (ULID) | Primary key. |
| `tenant_id` | Owner. |
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
| `is_globally_enabled` | Boolean default. |
| `tenant_overrides` | JSON map of `tenant_id` → boolean, or a normalized `feature_flag_tenant` pivot table where override volume is high. |

## Multi-tenancy enforcement

Every tenant-scoped table above MUST carry `tenant_id` and MUST be covered
by the row-level tenant isolation mechanism defined in
[`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md). A
migration that adds a tenant-scoped table without `tenant_id` fails review.

## What ZodiCore provides vs. what a product customizes

ZodiCore provides: the migrations for every table above, shipped as a
foundational migration set every product runs first, plus the Eloquent
models and tenant-scoping global scope that enforce isolation automatically.

A product customizes: additional columns on `settings`/`tenant` JSON blobs
for product-specific configuration, and all product-domain tables layered on
top. A product MUST NOT modify the base migrations directly — product-specific
columns on a base table are added via a new migration in the product's own
module, never by editing ZodiCore's migration files.

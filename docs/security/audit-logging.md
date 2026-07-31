# Audit Logging

> Every Zodize product MUST implement audit logging using this data model and
> retention schedule. Audit logs are how Zodize proves, to a customer or a
> regulator, exactly who did what and when. Gaps in audit coverage are treated
> as security defects, not enhancements.

## What must be audited

The following event categories are mandatory in every product, with no
opt-out:

- **Authentication events**: login success, login failure (with reason:
  bad password, MFA failure, account locked), logout, password change,
  password reset request and completion, MFA enrollment/disable, session
  revocation, new-device login.
- **Permission and role changes**: role created/updated/deleted, permission
  granted/revoked on a role, user assigned to or removed from a role, custom
  role created via the role builder (see
  [`rbac-permissions.md`](./rbac-permissions.md#custom-role-builder-enterprise-tier)).
- **Data exports**: any bulk export (CSV/PDF/API pull of more than one
  record) of resources classified `confidential` or `restricted` under
  [`data-protection-privacy.md`](./data-protection-privacy.md#data-classification-levels).
- **Destructive actions**: delete, bulk delete, restore-from-trash, and any
  action that voids/reverses a previously committed record (e.g.
  `invoices.void`, `trades.cancel`).
- **Financial transactions**: every state-changing action on a monetary
  record — payment captured, refund issued, transfer executed, trade filled,
  ledger entry posted — regardless of amount.
- **Settings changes**: general settings/branding changes, payment gateway
  configuration changes, license/plan changes, SSO configuration changes,
  API token issuance and revocation.

Read-only access to non-sensitive resources is not audited by default (it
would drown the log); read access to `restricted`-classified resources
(e.g., a patient chart, a bank account's full number) MUST be audited on
financial-grade and healthcare-grade products even though it is a read.

## Audit log data model

Every audit entry MUST record the following fields, stored in an
append-only `audit_logs` table (or equivalent event store):

| Field | Description |
|---|---|
| `id` | Unique identifier (UUID, not sequential, to avoid leaking volume). |
| `actor_id` / `actor_type` | The user (or system process, e.g. `system:scheduler`) that performed the action. |
| `actor_display_name` | Denormalized snapshot of the actor's name at the time of the event, so the entry remains legible after the user record changes or is deleted. |
| `action` | The permission-style identifier of what happened, e.g. `invoices.delete`, `auth.login_failed`, `roles.permission_granted`. |
| `subject_type` / `subject_id` | The model class and ID the action was performed on. |
| `before` / `after` | JSON snapshots of the changed attributes only (not the full record) — a diff, not a duplicate of the row. Null `before` for creates, null `after` for deletes. |
| `ip_address` | The originating IP, captured from the request, not from a client-supplied header. |
| `user_agent` | Raw user agent string of the originating request. |
| `metadata` | Free-form JSON for action-specific context (e.g. export format and row count, reason entered for a manual balance adjustment). |
| `occurred_at` | UTC timestamp with millisecond precision, set server-side, never client-supplied. |

## Immutability requirement

- The `audit_logs` table MUST be append-only at the application layer: no
  Eloquent `update()` or `delete()` path may target it, and the database
  user the application connects as SHOULD be granted `INSERT`/`SELECT` only
  on that table in production (no `UPDATE`/`DELETE` grant).
- Financial-grade products (ZodiBank, ZodiTrade, ZodiXchange, ZodiCapital,
  ZodiYield) MUST additionally implement cryptographic hash-chaining: each
  entry stores `previous_hash` (the SHA-256 hash of the prior entry's core
  fields) and `hash` (the SHA-256 hash of its own core fields concatenated
  with `previous_hash`), making any retroactive edit or deletion detectable
  by a chain-verification job. This job MUST run at least daily and alert
  on any break per
  [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md).

## Retention periods

| Product tier | Retention period |
|---|---|
| Standard products | 1 year, minimum, from `occurred_at`. |
| Financial-grade products | 7 years, minimum, from `occurred_at`, matching typical financial recordkeeping requirements. |
| Healthcare-grade products (ZodiMed) | 6 years, minimum, from `occurred_at`, or longer where the buyer's operating jurisdiction requires it — the product SPEC MUST document the applicable jurisdictional minimum and use the longer of the two. |

Audit logs MUST NOT be deleted by the standard data-retention/deletion
tooling described in
[`data-protection-privacy.md`](./data-protection-privacy.md#data-retention-and-deletion) —
including in response to a user's right-to-deletion request. A deletion
request removes or anonymizes the *subject's* personal data on the subject
record itself; the audit trail retains a reference to the actor/subject ID
and the denormalized display name captured at the time, which is treated as
a legitimate record-keeping exception under the product's privacy policy.

## Audit log / activity timeline UI

Every product MUST expose an activity timeline UI, referenced from every
resource's detail page and from a tenant-wide "Activity" section:

- **Resource-level timeline**: on any record's detail page, a chronological
  list of every audit entry where that record is the `subject`, rendered as
  human-readable sentences (e.g. "Jane Doe changed the invoice total from
  $500.00 to $650.00 — 2 hours ago") derived from the `before`/`after` diff.
- **Tenant-level activity log**: an Admin/Owner-only screen listing all
  audit entries for the tenant, filterable by actor, action category, date
  range, and subject type, with export to CSV restricted to users holding
  `audit_logs.export`.
- Both views MUST be paginated (see
  [`../quality/performance-standards.md`](../quality/performance-standards.md#pagination))
  and MUST NOT allow client-side filtering to bypass the server-side tenant
  scope.

## Related standards

- [`rbac-permissions.md`](./rbac-permissions.md)
- [`data-protection-privacy.md`](./data-protection-privacy.md)
- [`../quality/monitoring-observability.md`](../quality/monitoring-observability.md)
- [`security-standards.md`](./security-standards.md)

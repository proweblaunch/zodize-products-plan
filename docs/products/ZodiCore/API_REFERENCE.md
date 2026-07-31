# ZodiCore — API Reference

Companion to [SPEC.md](./SPEC.md). Conforms to
[api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md). Base path:
`/api/v1`. This is the API surface **one ZodiCore deployment** exposes to
its own frontend and to integrations its buyer configures — there is no
`/admin/tenants`, no impersonation endpoint, and no tenant-scoped route,
since there is no shared platform and no other tenant in this deployment;
see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
Full OpenAPI 3.1 spec is generated from route/request/resource annotations
at implementation time; this table is the authoritative endpoint inventory
the generated spec must match.

## Auth & Session

| Method | Path | Purpose |
|---|---|---|
| POST | `/auth/register` | Create a user account on this deployment |
| POST | `/auth/login` | Email/password login |
| POST | `/auth/logout` | Revoke current session |
| POST | `/auth/password/forgot` | Request reset email |
| POST | `/auth/password/reset` | Complete reset |
| GET | `/auth/social/{provider}/redirect` | Begin social login flow (Google/Facebook/LinkedIn) |
| GET | `/auth/social/{provider}/callback` | Complete social login flow |
| GET | `/me` | Current user detail |

## Users

| Method | Path | Purpose |
|---|---|---|
| GET | `/users` | List users (admin) |
| POST | `/users/invite` | Invite a staff user by email + role (admin) |
| GET | `/users/{id}` | User detail |
| PATCH | `/users/{id}` | Update user |
| POST | `/users/{id}/suspend` | Suspend user (admin) |

## Roles & Permissions

| Method | Path | Purpose |
|---|---|---|
| GET | `/roles` | List roles (system + custom) |
| POST | `/roles` | Create custom role |
| PATCH | `/roles/{id}` | Update role permissions |
| DELETE | `/roles/{id}` | Delete custom role |
| POST | `/users/{id}/roles` | Assign role to user |
| DELETE | `/users/{id}/roles/{roleId}` | Revoke role |
| GET | `/permissions` | List all available permissions |

## Wallet & Transactions

| Method | Path | Purpose |
|---|---|---|
| GET | `/wallet/balance` | Current user's balance |
| GET | `/wallet/transactions` | Paginated transaction history |
| POST | `/wallet/deposits` | Initiate a deposit via a configured gateway |
| POST | `/wallet/withdrawals` | Request a withdrawal via a configured withdraw method |
| GET | `/wallet/withdrawals/{id}` | Withdrawal request status |
| POST | `/wallet/withdrawals/{id}/approve` | Approve a pending withdrawal (admin) |
| POST | `/wallet/withdrawals/{id}/reject` | Reject a pending withdrawal (admin) |
| POST | `/wallet/transfers` | Peer-to-peer balance transfer |

## Payment Gateways

| Method | Path | Purpose |
|---|---|---|
| GET | `/admin/gateways` | List gateways and their enabled state (admin) |
| PATCH | `/admin/gateways/{id}` | Enable/configure a gateway (admin) |
| POST | `/webhooks/gateways/{gateway}/callback` | Inbound gateway payment webhook receiver |

## Plans

| Method | Path | Purpose |
|---|---|---|
| GET | `/plans` | List available plans |
| POST | `/admin/plans` | Create a plan (admin) |
| PATCH | `/admin/plans/{id}` | Update a plan (admin) |
| DELETE | `/admin/plans/{id}` | Deactivate a plan (admin) |

## KYC

| Method | Path | Purpose |
|---|---|---|
| GET | `/kyc/form` | Get the active KYC form schema |
| POST | `/kyc/submit` | Submit KYC data |
| GET | `/admin/kyc/submissions` | List pending KYC submissions (admin) |
| POST | `/admin/kyc/submissions/{id}/approve` | Approve a submission (admin) |
| POST | `/admin/kyc/submissions/{id}/reject` | Reject a submission (admin) |

## Referral Program

| Method | Path | Purpose |
|---|---|---|
| GET | `/referral/link` | Current user's referral link |
| GET | `/referral/tree` | Current user's referral tree and commission history |
| GET | `/admin/referral/settings` | Get referral configuration (admin) |
| PATCH | `/admin/referral/settings` | Update commission tiers/triggers (admin) |

## Language / i18n

| Method | Path | Purpose |
|---|---|---|
| GET | `/languages` | List active languages |
| POST | `/admin/languages` | Add a language (admin) |
| PATCH | `/admin/languages/{code}/translations` | Update translation strings (admin) |

## CMS / Page Builder

| Method | Path | Purpose |
|---|---|---|
| GET | `/pages/{slug}` | Public page content and sections |
| GET | `/admin/frontends` | List CMS-managed pages (admin) |
| POST | `/admin/frontends/{id}/sections` | Add/reorder a page section (admin) |
| PATCH | `/admin/frontends/{id}/seo` | Update SEO metadata (admin) |

## Task Tracker

| Method | Path | Purpose |
|---|---|---|
| GET | `/projects` | List projects |
| POST | `/projects` | Create a project |
| GET | `/projects/{id}/tasks` | List a project's tasks |
| POST | `/projects/{id}/tasks` | Create a task |
| PATCH | `/tasks/{id}` | Update a task (status, assignee, due date) |
| POST | `/tasks/{id}/comments` | Add a comment |
| POST | `/tasks/{id}/attachments` | Upload an attachment |

## Generic Records

| Method | Path | Purpose |
|---|---|---|
| GET | `/record-types` | List record types |
| POST | `/admin/record-types` | Create a record type with its field schema (admin) |
| GET | `/record-types/{id}/records` | List records of a type, filterable |
| POST | `/record-types/{id}/records` | Create a record |
| GET | `/records/{id}` | Record detail |
| PATCH | `/records/{id}` | Update a record |
| DELETE | `/records/{id}` | Soft-delete a record |
| POST | `/record-types/{id}/records/import` | Bulk CSV import with field mapping |

## Plugins & Marketplace (present only when this deployment enables the plugin system)

| Method | Path | Purpose |
|---|---|---|
| GET | `/marketplace/plugins` | Browse this product's marketplace listings, per [marketplace-architecture.md](../../architecture/marketplace-architecture.md) |
| GET | `/plugins` | List plugins installed on this deployment |
| POST | `/plugins/{id}/install` | Install a plugin into this deployment |
| POST | `/plugins/{id}/uninstall` | Uninstall a plugin from this deployment |
| PATCH | `/plugins/{id}` | Enable/disable an installed plugin |

## Webhooks (buyer-configured, for the buyer's own integrations)

| Method | Path | Purpose |
|---|---|---|
| GET | `/webhooks` | List webhook endpoints this deployment's admin has registered |
| POST | `/webhooks` | Register a webhook endpoint |
| PATCH | `/webhooks/{id}` | Update subscribed event types |
| DELETE | `/webhooks/{id}` | Remove endpoint |
| GET | `/webhooks/{id}/deliveries` | Delivery log |
| POST | `/webhooks/{id}/deliveries/{deliveryId}/replay` | Replay a delivery |

## Audit & Search

| Method | Path | Purpose |
|---|---|---|
| GET | `/audit-logs` | Query this deployment's audit log (filter by actor/action/subject/date) |
| GET | `/search` | Search across this deployment's own modules (users, transactions, tasks, records) |

Every endpoint above returns the standard envelope and error format defined
in [api-standards.md](../../development/api-standards.md#requestresponse-envelope),
is rate-limited per [api-standards.md](../../development/api-standards.md#rate-limiting),
and every mutating endpoint is covered by the mandatory authorization-denial
and not-found test cases in
[testing-standards.md](../../development/testing-standards.md#non-negotiable-test-cases) —
there is no cross-tenant isolation test category for this API, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md#what-single-tenant-changes-in-the-data-model).

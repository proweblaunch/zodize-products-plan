# ZodiCore — API Reference

Companion to [SPEC.md](./SPEC.md). Conforms to
[api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md). Base path:
`/api/v1`. Full OpenAPI 3.1 spec is generated from route/request/resource
annotations at implementation time; this table is the authoritative endpoint
inventory the generated spec must match.

## Identity & Session

| Method | Path | Purpose |
|---|---|---|
| POST | `/auth/register` | Create tenant + owner user |
| POST | `/auth/login` | Email/password login |
| POST | `/auth/logout` | Revoke current session |
| POST | `/auth/mfa/challenge` | Submit MFA code |
| POST | `/auth/mfa/enroll` | Begin MFA enrollment (TOTP/WebAuthn) |
| POST | `/auth/password/forgot` | Request reset email |
| POST | `/auth/password/reset` | Complete reset |
| GET | `/auth/sso/{provider}/redirect` | Begin SSO flow |
| GET | `/auth/sso/{provider}/callback` | Complete SSO flow |
| GET | `/me` | Current user + tenant context |
| GET | `/me/sessions` | List active sessions/devices |
| DELETE | `/me/sessions/{id}` | Revoke a session |

## Tenancy

| Method | Path | Purpose |
|---|---|---|
| GET | `/tenant` | Current tenant details |
| PATCH | `/tenant` | Update tenant settings |
| GET | `/companies` | List companies |
| POST | `/companies` | Create company |
| GET | `/companies/{id}/branches` | List branches |
| POST | `/companies/{id}/branches` | Create branch |

## Users & Invitations

| Method | Path | Purpose |
|---|---|---|
| GET | `/users` | List tenant users |
| POST | `/users/invite` | Invite a user by email + role |
| GET | `/users/{id}` | User detail |
| PATCH | `/users/{id}` | Update user |
| POST | `/users/{id}/suspend` | Suspend user |
| DELETE | `/users/{id}` | Soft-delete (remove) user |

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

## Billing

| Method | Path | Purpose |
|---|---|---|
| GET | `/billing/plans` | List available plans |
| GET | `/billing/subscription` | Current subscription |
| POST | `/billing/subscription` | Create/change subscription |
| POST | `/billing/subscription/cancel` | Cancel (at period end) |
| GET | `/billing/invoices` | List invoices |
| GET | `/billing/invoices/{id}` | Invoice detail |
| GET | `/billing/invoices/{id}/pdf` | Download invoice PDF |
| GET | `/billing/payment-methods` | List payment methods |
| POST | `/billing/payment-methods` | Add payment method |
| POST | `/billing/payment-methods/{id}/default` | Set default |

## Notifications

| Method | Path | Purpose |
|---|---|---|
| GET | `/notifications` | List (paginated, filterable by read/unread) |
| POST | `/notifications/{id}/read` | Mark read |
| POST | `/notifications/read-all` | Mark all read |
| GET | `/notification-preferences` | Get preferences |
| PATCH | `/notification-preferences` | Update preferences |

## Plugins & Marketplace

| Method | Path | Purpose |
|---|---|---|
| GET | `/marketplace/plugins` | Browse marketplace listings |
| GET | `/plugins` | List installed plugins |
| POST | `/plugins/{id}/install` | Install plugin |
| POST | `/plugins/{id}/uninstall` | Uninstall plugin |
| PATCH | `/plugins/{id}` | Enable/disable |

## Developer Portal

| Method | Path | Purpose |
|---|---|---|
| GET | `/api-tokens` | List tokens |
| POST | `/api-tokens` | Create scoped token |
| DELETE | `/api-tokens/{id}` | Revoke token |
| GET | `/webhooks` | List webhook endpoints |
| POST | `/webhooks` | Register webhook endpoint |
| PATCH | `/webhooks/{id}` | Update subscription/event types |
| DELETE | `/webhooks/{id}` | Remove endpoint |
| GET | `/webhooks/{id}/deliveries` | Delivery log |
| POST | `/webhooks/{id}/deliveries/{deliveryId}/replay` | Replay a delivery |

## Audit & Search

| Method | Path | Purpose |
|---|---|---|
| GET | `/audit-logs` | Query audit log (filter by actor/action/subject/date) |
| GET | `/search` | Global search across installed modules |

## Admin (Zodize-internal, requires `impersonation.perform`-tier role)

| Method | Path | Purpose |
|---|---|---|
| GET | `/admin/tenants` | List all tenants across products |
| POST | `/admin/tenants/{id}/suspend` | Suspend tenant |
| POST | `/admin/impersonation/start` | Begin impersonation session |
| POST | `/admin/impersonation/end` | End impersonation session |
| GET | `/admin/feature-flags` | List/manage feature flags |
| PATCH | `/admin/feature-flags/{id}` | Update flag rollout |
| GET | `/admin/system-health` | Aggregate platform health |

Every endpoint above returns the standard envelope and error format defined
in [api-standards.md](../../development/api-standards.md#requestresponse-envelope),
is rate-limited per [api-standards.md](../../development/api-standards.md#rate-limiting),
and every mutating endpoint is covered by the mandatory test cases in
[testing-standards.md](../../development/testing-standards.md#non-negotiable-test-cases).

# REST Standards

Concrete URL, verb, and resource conventions layered on top of
[api-standards.md](./api-standards.md).

## Resource naming

- Plural, kebab-case nouns: `/api/v1/purchase-orders`, never verbs
  (`/api/v1/createOrder` is invalid).
- Nested resources only one level deep in the URL; deeper relationships are
  expressed via filters, not deeper nesting:
  `/api/v1/invoices?filter[customer_id]=123`, not
  `/api/v1/customers/123/invoices/456/line-items/789/taxes`.

## HTTP verbs and status codes

| Verb | Use | Success status |
|---|---|---|
| `GET` | Read (list or single) | 200 |
| `POST` | Create, or a non-idempotent action (`/invoices/{id}/send`) | 201 (create) / 200 (action) |
| `PUT` | Full replace of a resource | 200 |
| `PATCH` | Partial update | 200 |
| `DELETE` | Soft delete (default) or hard delete (explicit, restricted) | 204 |

- Actions that are not pure CRUD are modeled as a `POST` to a sub-resource
  verb path: `POST /invoices/{id}/void`, `POST /orders/{id}/cancel` — never
  overload `PATCH status=voided` for an action with side effects (audit,
  notification, webhook).

## Resource representation

- Every resource response includes `id` (UUID, not sequential integer,
  exposed externally), `created_at`, `updated_at` (ISO 8601 UTC), and a
  `type` discriminator for polymorphic endpoints.
- Monetary values are represented as integer minor units plus an explicit
  currency code (`{"amount": 150000, "currency": "USD"}`), never as floats.
  See [localization-i18n.md](../standards/localization-i18n.md).
- Relationships are represented as nested `id` + optionally expanded object
  via `?include=customer,line_items`, never auto-expanded by default (keeps
  default payloads small).

## Soft delete semantics

`DELETE` soft-deletes by default (see
[data-protection-privacy.md](../security/data-protection-privacy.md)); the
resource is excluded from default list/read responses but retrievable via
`?with_trashed=true` for authorized roles, and restorable via
`POST /{resource}/{id}/restore`. Hard delete is a separate, permission-gated
endpoint (`DELETE /{resource}/{id}?permanent=true`) restricted to Owner/Admin
roles and always audit-logged per
[audit-logging.md](../security/audit-logging.md).

## Bulk operations

Bulk endpoints (`POST /invoices/bulk-void`) accept an array of IDs (capped at
500 per request), return a per-item result array so partial failures are
visible, and are always queued (202 Accepted + a job status endpoint) above a
configurable item-count threshold rather than processed synchronously.

## Consistency with the design system

Every list endpoint's filter/sort/pagination contract matches the frontend
[table-standards.md](../standards/table-standards.md) implementation exactly
— the frontend does not invent query parameters the API doesn't define.

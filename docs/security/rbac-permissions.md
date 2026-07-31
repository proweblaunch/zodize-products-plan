# RBAC & Permissions Model

> The single role-based access control model every Zodize product inherits
> from ZodiCore. Product specs MAY add product-specific permissions but MUST
> use this naming convention and MUST NOT introduce a parallel access-control
> mechanism.

## Core entities

- **Tenant**: the top-level isolation boundary — one customer organization.
  See [`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md).
- **Team** (a.k.a. Company/Branch within a tenant): a nested scoping unit
  inside a tenant, used by multi-company and multi-branch products (see
  `../architecture/multi-tenancy.md#multi-company-and-multi-branch`). A user's
  role can be assigned at the tenant level (applies everywhere in the tenant)
  or at the team level (applies only within that team).
- **Role**: a named, ordered set of Permissions, assignable to a user within
  the scope of a tenant or a team.
- **Permission**: an atomic, checkable capability, always expressed as
  `resource.action`.
- **User-Role Assignment**: a row binding `(user_id, role_id, tenant_id,
  team_id nullable)` — a user can hold different roles in different teams of
  the same tenant, and different roles across different tenants entirely.

This model is implemented with `spatie/laravel-permission` conventions
(roles and permissions as first-class Eloquent models, `HasRoles` on the
`User` model) with a tenant/team-aware `team_id` foreign key on the pivot
tables, per the package's built-in team support.

## Permission naming convention

Every permission MUST be named `<resource>.<action>`, both lowercase,
singular resource names avoided in favor of the plural table-like form used
elsewhere in the codebase (e.g. `invoices`, not `invoice`):

- Standard actions: `view` (single record), `viewAny` (list/index),
  `create`, `update`, `delete`, `restore` (undo a soft delete), `export`.
- Domain-specific actions are permitted when the standard verbs do not fit,
  e.g. `invoices.void`, `payments.refund`, `users.impersonate`. A
  domain-specific action MUST still follow the `resource.action` shape.
- Examples: `invoices.create`, `patients.view`, `trades.execute`,
  `users.impersonate`, `settings.billing.update`.
- A permission string MUST map 1:1 to a Policy method of the same name (see
  [`authentication-authorization.md`](./authentication-authorization.md)) —
  `invoices.create` corresponds to `InvoicePolicy::create()`.

## Default system roles

Every product MUST ship the following roles out of the box, seeded per
tenant at provisioning time (see
[`../architecture/multi-tenancy.md#tenant-provisioning-and-deprovisioning`](../architecture/multi-tenancy.md)).
Product specs assign the product's own permissions to each role; the
hierarchy and intent below are fixed:

| Role | Intent | Notes |
|---|---|---|
| Owner | Full control of the tenant, including billing and tenant deletion. Exactly one per tenant may hold `tenant.delete`; ownership is transferable but never plural for that specific permission. | Cannot be removed by any other role; can only be reassigned by the current Owner. |
| Admin | Full operational control (all resource CRUD, user management, role assignment) except tenant deletion and Owner-only settings. | MFA mandatory per [`authentication-authorization.md`](./authentication-authorization.md#multi-factor-authentication-mfa). |
| Manager | Full CRUD within their assigned team/branch scope; can manage Members and Viewers within that scope; cannot manage other Managers or Admins. | Team-scoped by default. |
| Member | Standard end-user: create/update/view their own records and any records their role's permissions grant across the team; no user-management or settings access. | The default role for a newly invited user. |
| Viewer | Read-only across the resources granted to the role; cannot create, update, or delete anything. | Used for auditors, external accountants, read-only stakeholders. |
| Billing | Access to billing, invoicing, and subscription management screens only; no access to the product's core domain data unless additionally granted. | MFA mandatory. Scoped narrowly by design — a Billing role MUST NOT default to `invoices.view` on the product's operational invoices unless the product's SPEC explicitly grants it. |
| Support / Impersonator | Internal Zodize staff role (not tenant-assignable) permitting time-boxed impersonation of a tenant user for support purposes. | Every impersonation session MUST be audit-logged per [`audit-logging.md`](./audit-logging.md#impersonation) and visibly banner'd in the UI for the duration of the session. |

Product specs MUST document, in a permissions matrix in
`docs/products/<product>/SPEC.md`, exactly which permissions each of these
roles holds for that product's resources.

## Custom role builder (enterprise tier)

- Every product MUST offer a custom role builder to tenants on the
  enterprise pricing tier: an admin UI that lists all permissions registered
  for the product, grouped by resource, with checkboxes to compose a new
  role.
- Custom roles are tenant-scoped (never visible to or usable by other
  tenants) and MUST be assignable at either the tenant or team level, same
  as system roles.
- A custom role MUST NOT be granted a permission the creating admin does not
  themselves hold (no privilege escalation via role composition) — this
  check happens server-side in the role-creation Policy, not just hidden in
  the UI.
- Deleting a custom role that is still assigned to users MUST be blocked
  until the role is unassigned, or MUST prompt for a reassignment target
  role as part of the deletion flow.

## Permission inheritance and override rules

- A user's **effective permissions** in a given team are the union of: (a)
  permissions from any role assigned at the tenant level, and (b)
  permissions from any role assigned at that specific team level. Tenant-level
  role assignments apply across every team in the tenant; team-level
  assignments apply only within that team.
- There is no "deny" permission type in the default model — permissions are
  additive only. A role never subtracts a capability another assigned role
  grants. Products requiring an explicit deny/exception model (e.g., "Manager
  X may do everything a Manager can except approve their own team's
  expenses") MUST implement it as a resource-level ownership rule inside the
  relevant Policy, not as a new permission-model primitive.
- The Owner role's permission set is not editable, including through the
  custom role builder, and always includes every permission registered for
  the product plus `tenant.delete`.
- When a permission is added to a product after a tenant's custom roles
  already exist, the new permission is unassigned by default on every
  existing custom role (fails closed) and MUST be explicitly granted by a
  tenant Admin or Owner.

## Related standards

- [`authentication-authorization.md`](./authentication-authorization.md)
- [`audit-logging.md`](./audit-logging.md)
- [`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md)
- [`../development/`](../development/) for Policy/Gate implementation conventions.

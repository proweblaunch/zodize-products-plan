# RBAC & Permissions Model

> The single role-based access control model every Zodize product
> implements in its own codebase, inherited from the base codebase's
> existing custom `Role`/`Permission` engine — never rebuilt, never
> replaced with a third-party package. Product specs MAY add
> product-specific permissions but MUST use this naming convention and MUST
> NOT introduce a parallel access-control mechanism. See
> [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)
> and [`../templates/admin-template.md`](../templates/admin-template.md).

## Core entities

- **Guard**: the base codebase ships three Laravel auth guards — `web` (the
  business's end customers/users), `admin` (the buyer's own back-office
  staff), and `branch_staff` (location/branch-scoped staff, kept only where
  the product's own [`SPEC.md`](../products/) calls for it). Roles and
  permissions are defined per guard; see
  [`../templates/admin-template.md`](../templates/admin-template.md#auth-guards).
- **Company / Branch** (optional): a nested scoping unit *within* the one
  business's deployment, used only by products that model multi-company or
  multi-branch operation — see
  [`localization-i18n.md`](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping).
  This is not tenancy: there is exactly one business per deployment, and a
  branch always belongs to that one business. A role can be assigned
  deployment-wide (applies everywhere in the business) or scoped to a
  specific company/branch.
- **Role**: a named, ordered set of Permissions on the base codebase's
  `Role` model, assignable to a staff account within the scope of the whole
  deployment or, on a multi-branch product, a specific branch.
- **Permission**: an atomic, checkable capability on the base codebase's
  `Permission` model, always expressed as `resource.action`.
- **User-Role Assignment**: a row binding `(admin_id, role_id, branch_id
  nullable)` on a multi-branch product — a staff member can hold different
  roles at different branches of the same business (e.g. branch manager at
  one location, read-only viewer at another).

This model is implemented with the base codebase's own first-party
`Role`/`Permission` Eloquent models and `AdminPermissionMiddleware` — **not**
`spatie/laravel-permission` or any other RBAC package. The base engine
already has a working, first-party implementation; introducing a second one
creates two competing sources of truth for authorization. See
[`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md#inherited-as-is-the-admin-engine-every-product-keeps).

## Permission naming convention

Every permission MUST be named `<resource>.<action>`, both lowercase,
singular resource names avoided in favor of the plural table-like form used
elsewhere in the codebase (e.g. `invoices`, not `invoice`):

- Standard actions: `view` (single record), `viewAny` (list/index),
  `create`, `update`, `delete`, `restore` (undo a soft delete), `export`.
- Domain-specific actions are permitted when the standard verbs do not fit,
  e.g. `invoices.void`, `payments.refund`, `withdrawals.approve`. A
  domain-specific action MUST still follow the `resource.action` shape.
- Examples: `invoices.create`, `patients.view`, `trades.execute`,
  `settings.gateways.update`.
- A permission string MUST map 1:1 to a Policy method of the same name (see
  [`authentication-authorization.md`](./authentication-authorization.md)) —
  `invoices.create` corresponds to `InvoicePolicy::create()`.

## Default system roles

Every product MUST ship the following roles out of the box for the
business's own staff, seeded once via the product's `DemoSeeder` at first
install (see
[`../development/migration-seeder-standards.md`](../development/migration-seeder-standards.md#seeders)),
editable afterward from the admin panel per
[`admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md#roles--permissions).
Product specs assign the product's own permissions to each role; the
hierarchy and intent below are fixed:

| Role | Intent | Notes |
|---|---|---|
| Owner | Full control of the business's own deployed instance, including every setting and every other staff member's role assignment. | Cannot be removed by any other role; ownership is reassignable only by the current Owner. |
| Admin | Full operational control (all resource CRUD, staff management, role assignment) except Owner-only settings. | MFA mandatory per [`authentication-authorization.md`](./authentication-authorization.md#multi-factor-authentication-mfa). |
| Manager | Full CRUD within their assigned branch scope on a multi-branch product; can manage Staff and Viewers within that scope; cannot manage other Managers or Admins. | Branch-scoped by default on products with multi-branch operation; deployment-wide otherwise. |
| Staff | Standard back-office user: create/update/view the records their role's permissions grant; no staff-management or settings access. | The default role for a newly invited staff account. |
| Viewer | Read-only across the resources granted to the role; cannot create, update, or delete anything. | Used for auditors, external accountants, read-only stakeholders. |
| Billing | Access to the business's own billing/plan/subscription-facing screens only (where the product's own domain models billing to the business's customers); no access to the product's core domain data unless additionally granted. | MFA mandatory. Scoped narrowly by design — a Billing role MUST NOT default to `invoices.view` on the product's operational invoices unless the product's SPEC explicitly grants it. |

Product specs MUST document, in a permissions matrix in
`docs/products/<product>/SPEC.md`, exactly which permissions each of these
roles holds for that product's resources.

## Custom role builder

Every product MUST offer a custom role builder in the admin panel,
inherited from the base codebase's existing Role management screen (see
[`admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md#roles--permissions)):
an admin UI that lists every permission registered for the product, grouped
by resource, with checkboxes to compose a new role — no code required.

- A custom role MUST NOT be granted a permission the creating admin does not
  themselves hold (no privilege escalation via role composition) — this
  check happens server-side in the role-creation Policy, not just hidden in
  the UI.
- On a multi-branch product, a custom role MUST be assignable at either the
  deployment-wide or the branch level, same as system roles.
- Deleting a custom role that is still assigned to staff MUST be blocked
  until the role is unassigned, or MUST prompt for a reassignment target
  role as part of the deletion flow.

## Permission inheritance and override rules

- On a multi-branch product, a staff member's **effective permissions** at a
  given branch are the union of: (a) permissions from any role assigned
  deployment-wide, and (b) permissions from any role assigned at that
  specific branch. A deployment-wide role assignment applies across every
  branch of the business; a branch-level assignment applies only within
  that branch.
- There is no "deny" permission type in the default model — permissions are
  additive only. A role never subtracts a capability another assigned role
  grants. Products requiring an explicit deny/exception model (e.g., "Manager
  X may do everything a Manager can except approve their own branch's
  expenses") MUST implement it as a resource-level ownership rule inside the
  relevant Policy, not as a new permission-model primitive.
- The Owner role's permission set is not editable, including through the
  custom role builder, and always includes every permission registered for
  the product.
- When a permission is added to a product after the business's own custom
  roles already exist, the new permission is unassigned by default on every
  existing custom role (fails closed) and MUST be explicitly granted by an
  Admin or Owner.

## Related standards

- [`authentication-authorization.md`](./authentication-authorization.md)
- [`audit-logging.md`](./audit-logging.md)
- [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)
- [`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md)
- [`../standards/localization-i18n.md`](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)
- [`../templates/admin-template.md`](../templates/admin-template.md)
- [`../development/`](../development/) for Policy/Gate implementation conventions.

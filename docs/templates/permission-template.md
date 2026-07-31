# Permission Template

This document specifies the standard permission set a new module registers
with the RBAC system, and how that registration happens automatically on
module install. The authorization model this template implements is defined
in [`../security/rbac-permissions.md`](../security/rbac-permissions.md). A
product customizes module-specific actions beyond the CRUD baseline; it does
not customize the CRUD permission naming convention or bypass auto-registration.

## Permission slug format

Every permission slug MUST follow `{resource}.{action}`, all lowercase,
singular resource name, e.g. `invoice.view`, `invoice.create`,
`team.member.invite`. A permission slug MUST NOT be introduced ad hoc inside
a controller or Policy — it MUST first exist as a row created by module
registration (see below).

## Standard CRUD pattern

Every module that exposes a manageable resource MUST register the following
five permissions for that resource at minimum:

| Slug | Grants |
|---|---|
| `{resource}.view` | Read a single resource and list resources. |
| `{resource}.create` | Create a new resource. |
| `{resource}.update` | Edit an existing resource. |
| `{resource}.delete` | Delete (or soft-delete) a resource. |
| `{resource}.export` | Export the resource list (CSV/PDF/etc.) — kept
distinct from `view` because export volume is a separate risk surface (bulk
data exfiltration) from single-record read access. |

A module MUST NOT collapse these into a single "manage" permission — the
five-way split is required so that roles can be composed with least
privilege (e.g. a support role with `view` but not `export` or `delete`).

## Module-specific actions

Beyond the CRUD baseline, a module MAY register additional action
permissions for state-changing operations that are not plain CRUD, e.g.
`invoice.send`, `invoice.void`, `timesheet.approve`. Each MUST follow the
same `{resource}.{action}` format and MUST have a `description` (see the
`permissions` table in [database-template.md](./database-template.md)) that
is specific enough to show meaningfully in the admin permission-management
UI from [admin-template.md](./admin-template.md) — "Allows voiding a
finalized invoice," not "Voids invoices."

## Auto-registration on module install

Every module MUST declare its permissions in `routes.permissions.php` at
its root (see [module-template.md](./module-template.md)):

```php
return [
    'module' => 'invoicing',
    'permissions' => [
        'invoice.view'    => 'View invoices and their line items.',
        'invoice.create'  => 'Create new invoices.',
        'invoice.update'  => 'Edit existing, non-finalized invoices.',
        'invoice.delete'  => 'Delete draft invoices.',
        'invoice.export'  => 'Export invoice lists to CSV or PDF.',
        'invoice.send'    => 'Send a finalized invoice to the customer.',
        'invoice.void'    => 'Void a finalized invoice.',
    ],
];
```

On module install (or first boot after a module is added to a product),
ZodiCore's module loader MUST:

1. Read every enabled module's `routes.permissions.php`.
2. Upsert each declared permission into the `permissions` table (see
   [database-template.md](./database-template.md)), keyed by slug, so
   re-running install is idempotent.
3. Attach newly-registered permissions to the `is_system` `Owner`/`Admin`
   roles automatically for every existing tenant, so that an upgrade never
   silently locks tenant admins out of new functionality.
4. Leave newly-registered permissions unattached to any non-system role by
   default — a tenant's custom roles MUST opt in explicitly.
5. Never delete a permission automatically when a module is disabled;
   disabling a module MUST retain the permission rows (for audit-log
   readability of historical grants) but MUST make the permission
   unassignable in the UI while the module remains disabled.

A module's Policy classes (see [module-template.md](./module-template.md))
MUST reference these slugs directly (e.g.
`$user->can('invoice.void', $invoice)`), never re-derive an equivalent check
from role name — role names are a UI/assignment concept, permissions are the
authorization primitive.

## What ZodiCore provides vs. what a product customizes

ZodiCore provides: the `permissions`, `roles`, `role_user`, and
`permission_role` tables, the module loader's permission-sync step, the
system-role auto-attach behavior, and the admin permission-management UI.

A product customizes: the contents of each module's
`routes.permissions.php` — its resource list and module-specific action
list. A product MUST NOT write directly to the `permissions` table outside
of the module-registration sync process.

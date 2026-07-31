# Admin (Back-Office) Template

Every Zodize product ships an internal administration surface used by the
**buyer's own staff** to run their business — not a Zodize-operated support
console. There is no cross-product or Zodize-staff admin layer: each
product's admin panel is part of that product's own single codebase,
inherited from the base codebase's existing, working admin engine (see
[`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md))
and extended with the product's own domain-specific admin screens. This
document specifies what every product inherits as-is and how a product adds
to it — it does not describe a SaaS tenant-management console, since no
such thing exists in this architecture (see
[`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md)).

## Directory structure (inherited, flat namespace)

```
core/
  app/Http/Controllers/
    Admin/                     # ~40 controllers, inherited as-is per product
      GeneralSettingController.php
      ManageUsersController.php
      DepositController.php
      WithdrawalController.php
      WithdrawMethodController.php
      ReferralSettingController.php
      KycController.php
      LanguageController.php
      CronConfigurationController.php
      ExtensionController.php
      FrontendController.php
      ...                      # + product-specific admin controllers added per module
    Gateway/                   # payment gateway controllers, see ../standards/payment-gateways.md
    Api/
    User/
    BranchStaff/                # only kept where the product's own SPEC.md calls for it
  app/Models/
    GeneralSetting.php Transaction.php BalanceTransfer.php WithdrawMethod.php
    Withdrawal.php Form.php Language.php CronJob.php Frontend.php Page.php
    Role.php Permission.php ...
routes/
  admin.php
```

This structure — and the ~40 controllers / 64 models it starts with — is
**inherited, not rebuilt**. A product's own admin work is additive: new
controllers/views for that product's domain modules (e.g. ZodiHotel adds
`Admin/RoomRateController.php`), following
[`../development/coding-standards-php-laravel.md`](../development/coding-standards-php-laravel.md)
and registered into the same `admin.php` route file and admin navigation.

## Auth guards

The base codebase ships three guards. Every product keeps the first two;
the third is conditional:

| Guard | Represents | Default per product |
|---|---|---|
| `web` | The business's end customers/users of the product | Always kept |
| `admin` | The buyer's own back-office staff (owner, managers, support) | Always kept |
| `branch_staff` | Location/branch-scoped staff with restricted access | **Dropped by default.** Only re-added if the product's own [`SPEC.md`](../products/) explicitly models branch-scoped staff (e.g. ZodiHotel front-desk, ZodiFleet depot staff), per [`../architecture/product-genericization-checklist.md`](../architecture/product-genericization-checklist.md#step-4--confirm-guard-configuration-matches-the-products-needs) |

## Roles & permissions (inherited, not Spatie)

Authorization uses the base codebase's own custom `Role`/`Permission`
models and `AdminPermissionMiddleware` — this is a first-party
implementation the base codebase already has working, and it is what every
product inherits. **Do not introduce Spatie's `laravel-permission` package
or any other RBAC library** — doing so duplicates a system that already
exists and creates two competing sources of truth for admin authorization.
See [`../security/rbac-permissions.md`](../security/rbac-permissions.md) for
the permission-naming and default-role conventions layered on top of this
inherited mechanism.

- Admin roles and their granular permissions are created and assigned
  entirely from the admin panel (`Admin/RoleController`-equivalent,
  inherited) — no code required, matching
  [`../standards/admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md#roles--permissions).
- A product's new domain modules register their own permissions into this
  same system per
  [`../templates/permission-template.md`](../templates/permission-template.md) —
  never a parallel permission table.

## Inherited admin sections every product ships

Every product's admin panel includes, unmodified from the base engine (full
detail per section in
[`../standards/admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md)):
General Settings & Branding, Manage Users (with wallet/ledger visibility per
[`../standards/wallet-system.md`](../standards/wallet-system.md)), Deposits
& Withdrawals, Withdraw Methods, Payment Gateways
([`../standards/payment-gateways.md`](../standards/payment-gateways.md)),
Referral Settings, Plans, KYC, Language Management, Cron Configuration,
Extensions, Frontend/CMS Page Builder
([`../architecture/frontend-backend-bridge.md`](../architecture/frontend-backend-bridge.md)),
and Roles & Permissions.

## What a product adds

A product's own admin work is limited to its domain-specific screens: entity
management for its own module's data (e.g. ZodiMed's patient records admin
list, ZodiBuild's project/phase admin views), following
[`../standards/table-standards.md`](../standards/table-standards.md) and
[`../standards/form-standards.md`](../standards/form-standards.md) for the
UI patterns and [`../templates/module-template.md`](../templates/module-template.md)
for the module structure. These new screens are added into the same
`Admin/` controller namespace and the same admin navigation shell (sidebar
per [`../standards/navigation-standards.md`](../standards/navigation-standards.md)),
never a separate admin application.

## System health

Every product's admin panel exposes a system health view built on the
inherited cron engine's DB-logged run history
(`CronJob`/`CronJobLog`/`CronSchedule`) plus queue status where the buyer's
hosting runs a persistent queue worker (see
[`../architecture/overview.md`](../architecture/overview.md#deployment-topology-per-product-per-buyer)) —
so the buyer's staff can confirm scheduled tasks and background jobs are
actually running without needing SSH/CLI access to their own server.

## Audit logging

Every mutating action taken from the admin panel — a settings change, a
withdrawal approval, a role change, a KYC decision — MUST produce an audit
log entry per [`../security/audit-logging.md`](../security/audit-logging.md).
This applies uniformly to the inherited admin sections and to any
product-specific admin screen a product adds; a new admin controller that
mutates data without an accompanying audit log entry fails
[`../quality/definition-of-done.md`](../quality/definition-of-done.md).

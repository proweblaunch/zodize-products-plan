# Product Genericization Checklist

> The exact, repeatable steps to run when cloning the sanitized base
> codebase for a new product. Builds on
> [`base-codebase-strategy.md`](./base-codebase-strategy.md). Run this once
> per product, before any of that product's own domain modules are written.

## Prerequisite

The base codebase has already been through the
[one-time base cleanup](./base-codebase-strategy.md#one-time-base-cleanup-fix-once-before-first-clone)
(dead build tooling removed, hardcoded product identity genericized, config
mutation and sitemap/robots file-write requirements documented, Plan pattern
genericized) and the
[frontend-backend bridge](./frontend-backend-bridge.md) is wired. Do not run
this checklist against the raw, un-cleaned audited codebase.

## Step 1 — Clone the base

Clone the sanitized base's `assets/` and `core/` directories into the new
product's repository. Clone the Zodize frontend shell's Blade components,
tokens, and layout into the same codebase per
[`frontend-backend-bridge.md`](./frontend-backend-bridge.md). The result is
one Laravel application containing both the inherited back office and the
inherited marketing frontend shell.

## Step 2 — Strip banking-domain-specific tables, models, and controllers

Unless the product is `ZodiBank` itself, remove:

- [ ] `loans` table, `loan_plans` table, `LoanPlanController`, `Loan`/`LoanPlan` models and their migrations
- [ ] `dps` table, `dps_plans` table, `DpsPlanController`, `Dps`/`DpsPlan` models and their migrations
- [ ] `fdr` table, `fdr_plans` table, `FdrPlanController`, `Fdr`/`FdrPlan` models and their migrations
- [ ] `branches` table, `branch_staff` table, `BranchStaff` model, and every controller/route under the `branch_staff` guard
- [ ] `other_banks` table and related model/controller
- [ ] `beneficiaries` table and related model/controller
- [ ] `airtime_operators` / `airtime_configs` tables and related model/controller
- [ ] The `branch_staff` auth guard entry in `config/auth.php`, unless the product's own spec calls for a branch-staff-equivalent role (re-added deliberately as a domain module, not left over from the base)

Removing a table means removing its migration, model, controllers, routes,
Blade views, and any seeder/factory referencing it — a partial removal that
leaves a dangling foreign key or an unreachable admin menu item is not
complete.

## Step 3 — Rename hardcoded product identity

- [ ] Confirm `systemDetails()` in `app/Http/Helpers/helpers.php` no longer
      returns a hardcoded name (should already be fixed by the one-time base
      cleanup — verify, don't assume).
- [ ] Set the product's actual name, default currency symbol, and default
      timezone in the seeded `general_settings` row via `DemoSeeder`.
- [ ] Grep `routes/web.php`, `routes/admin.php`, and every Blade
      `@section('title', ...)` for any remaining literal banking-specific
      string and replace with the product's identity or a
      `GeneralSetting`-sourced value.

## Step 4 — Confirm guard configuration matches the product's needs

- [ ] `web` guard (User) — always kept.
- [ ] `admin` guard (Admin) — always kept.
- [ ] `branch_staff` guard — dropped by default per Step 2. Only re-add if
      the product's own [`SPEC.md`](../products/) explicitly models a
      branch/location staff role with restricted, branch-scoped access
      (e.g. ZodiHotel front-desk staff, ZodiFleet depot staff) — and if so,
      re-add it as a deliberately-scoped module using the base's existing
      guard/RBAC pattern, not by un-deleting the banking `BranchStaff` code.

## Step 5 — Build the product's own domain tables and modules

- [ ] Read the product's [`docs/products/<product>/SPEC.md`](../products/),
      specifically its Core Data Model and Modules & Submodules sections.
- [ ] Add the product's domain tables as new migrations, following
      [`../development/database-standards.md`](../development/database-standards.md)
      and [`../development/migration-seeder-standards.md`](../development/migration-seeder-standards.md).
- [ ] Add the product's domain modules following
      [`../templates/module-template.md`](../templates/module-template.md),
      consuming the inherited engine's services (wallet, notifications,
      RBAC/permissions, settings, KYC, i18n) rather than duplicating them.
- [ ] Register the product's domain-specific admin permissions into the
      inherited `Role`/`Permission` system per
      [`../templates/permission-template.md`](../templates/permission-template.md).

## Step 6 — Verify the Plan pattern fits, or extend it

- [ ] If the product has a subscription-tier, membership, or
      rate/term/limit-configured offering, confirm it maps onto the
      genericized `Plan` model from
      [`base-codebase-strategy.md`](./base-codebase-strategy.md#genericizing-the-plan-pattern)
      before building a parallel concept.
- [ ] `ZodiBank` specifically re-adds Loan/DPS/FDR as its own domain modules
      built on top of the generic `Plan` model, per its own spec — this is
      the one product where Step 2's removals are reversed deliberately, not
      skipped.

## Step 7 — Payment gateways and wallet

- [ ] Confirm the gateway set enabled by default matches the product's
      target market per
      [`../standards/payment-gateways.md`](../standards/payment-gateways.md)
      (every product ships with the full inherited gateway list available;
      the admin enables/configures only the ones relevant to their business
      from the panel — no code change either way).
- [ ] Confirm wallet/ledger semantics match the product's domain per
      [`../standards/wallet-system.md`](../standards/wallet-system.md) — a
      product like ZodiCommerce uses the wallet for store credit/refunds; a
      product like ZodiBank uses it as the core account balance. The
      ledger engine is identical; only the product's own modules decide what
      triggers a ledger entry.

## Step 8 — Seed data and demo content

- [ ] Write a product-specific `DemoSeeder` per
      [`../development/migration-seeder-standards.md`](../development/migration-seeder-standards.md#seeders),
      covering both the inherited engine's tables (settings, a realistic
      gateway configuration, seeded CMS pages/sections per
      [`frontend-backend-bridge.md`](./frontend-backend-bridge.md)) and the
      product's own domain tables — the Demo Standard in
      [`../../README.md`](../../README.md) applies to the whole product, not
      just its new modules.

## Step 9 — Run the full checklist gate

- [ ] [`../checklists/production-readiness-checklist.md`](../checklists/production-readiness-checklist.md)
- [ ] [`../checklists/security-checklist.md`](../checklists/security-checklist.md)
- [ ] Confirm zero references to another Zodize product, to a shared
      "ZodiCore platform," or to any `tenant_id`/multi-tenant construct
      remain anywhere in the product's codebase or its
      [`SPEC.md`](../products/) — see
      [`single-tenant-deployment-model.md`](./single-tenant-deployment-model.md).

## Related standards

- [`base-codebase-strategy.md`](./base-codebase-strategy.md)
- [`frontend-backend-bridge.md`](./frontend-backend-bridge.md)
- [`single-tenant-deployment-model.md`](./single-tenant-deployment-model.md)
- [`../templates/module-template.md`](../templates/module-template.md)

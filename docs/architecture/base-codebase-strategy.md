# Base Codebase Strategy

> How Zodize builds twenty independent products from one audited base
> codebase instead of twenty from-scratch builds. Builds on
> [`overview.md`](./overview.md#one-base-codebase-twenty-independent-products)
> and [`single-tenant-deployment-model.md`](./single-tenant-deployment-model.md).

## Source of the base codebase

Every Zodize product's back office starts as a clone of one audited,
production Laravel application (internally referred to during audit as
"ViserBank/ViserLab Core Engine v3.0", sourced from
`/home/qfsfountains/public_html`). This is not a template written for this
handbook — it is a working banking application with a complete admin back
office already covering settings, payment gateways, wallet/ledger, KYC,
referrals, plans, i18n, and CMS. Treat it as the inherited engine every
product's back office is cloned and adapted from, never rebuilt from
scratch.

## Directory structure

```
public_html/
├── assets/     # public static assets (served directly by the web server)
└── core/       # the Laravel application
    ├── app/
    ├── config/
    ├── database/
    ├── lang/
    ├── resources/
    └── routes/
```

Stack: Laravel 11, PHP ^8.3, Vite 5 as the build tool. A legacy
`webpack.mix.js` is present in the audited codebase and MUST be deleted as
part of the one-time base cleanup below — it is dead tooling, Vite 5 is
what actually builds the frontend assets.

Namespace layout is flat (non-modular), matching the base engine as
audited: `App\Http\Controllers\Admin`, `...\Api`, `...\Gateway`, `...\User`,
`...\BranchStaff`. A product's own domain modules MAY adopt the modular
namespace pattern described in
[`overview.md`](./overview.md#modular-monolith-one-codebase-per-product) for
new code, but MUST NOT restructure the inherited engine's existing flat
namespace — that refactor is out of scope and risks destabilizing a working
system for no product-facing benefit. As audited, the base has 40 admin
controllers and 64 Eloquent models.

## Inherited as-is: the admin engine every product keeps

The following systems already exist, already work, and MUST be inherited
unmodified (beyond the one-time cleanup in
[§ One-time base cleanup](#one-time-base-cleanup-fix-once-before-first-clone))
into every product. Do not reimplement any of these — see
[`../standards/admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md)
for the buyer-facing configuration surface each one exposes.

| System | Controllers / Models | What it provides |
|---|---|---|
| General settings & branding | `Admin/GeneralSettingController.php`, `Models/GeneralSetting.php` | Single-row `general_settings` table, cached under key `GeneralSetting`; site name, logo, favicon, currency symbol, timezone, base config |
| Wallet / ledger | `Admin/ManageUsersController.php`, `DepositController.php`, `WithdrawalController.php`, `Models/User.php`, `Transaction.php`, `BalanceTransfer.php` | Double-entry balance ledger — see [`../standards/wallet-system.md`](../standards/wallet-system.md) |
| Payment gateways | `app/Http/Controllers/Gateway/` (30+ controllers), `gateways`/`gateway_currencies` tables | See [`../standards/payment-gateways.md`](../standards/payment-gateways.md) |
| Withdrawal methods | `Admin/WithdrawMethodController.php`, `WithdrawMethod.php`, `Withdrawal.php` | Configurable payout methods with dynamic JSON-defined user input forms |
| Referral system | `Admin/ReferralSettingController.php` | Multi-level tree via `ref_by` on `users`, admin-configurable commission percentages per level |
| Plan pattern | `LoanPlanController.php`, `DpsPlanController.php`, `FdrPlanController.php` | **Genericize, do not inherit literally** — see [§ Genericizing the Plan pattern](#genericizing-the-plan-pattern) |
| KYC | `Admin/KycController.php`, `Models/Form.php` (dynamic JSON form-builder schema), `users.kyc_data` | Admin-defined KYC form schema, per-user submission and review flow |
| Language / i18n | `Admin/LanguageController.php`, `Models/Language.php`, `core/lang/{code}.json`, `LanguageMiddleware` | See [`../standards/localization-i18n.md`](../standards/localization-i18n.md) |
| Cron engine | `Admin/CronConfigurationController.php`, `CronJob`/`CronJobLog`/`CronSchedule` models | DB-logged scheduled task engine, admin-visible run history |
| Extensions | `Admin/ExtensionController.php` | Toggle/configure Analytics, Tawk.to, Facebook Pixel, custom captcha — no Blade edits required |
| SEO + CMS/page builder | `Admin/FrontendController.php`, `Models/Frontend.php`, `Page.php` | Dynamic section-based page builder (`frontends`, `pages` tables), SEO metadata under `seo.data` — see [`frontend-backend-bridge.md`](./frontend-backend-bridge.md) |
| Social login | `GeneralSettingController@socialiteCredentials` | Google/Facebook/LinkedIn via `laravel/socialite`, enable/disable + credentials per provider from settings |
| Policy pages, maintenance mode, GDPR cookie | `FrontendController.php` / `Frontend` model | All admin-editable |
| Sitemap / robots | `FrontendController@sitemap` / `@robot` | Admin code-editor that overwrites physical `sitemap.xml`/`robots.xml` — see cleanup note below |
| Notifications | `App\Notify\Notify` dispatcher | Mailjet, MessageBird, SendGrid, Twilio, Vonage already integrated for email/SMS/push |
| Auth | Three guards: `web` (User), `admin` (Admin), `branch_staff` (BranchStaff) | Custom RBAC (not Spatie) via `Role`/`Permission` models + `AdminPermissionMiddleware` |

## Must be stripped: domain-specific tables that don't transfer

The audited base is a banking application. The following tables, models,
and controllers are banking-domain-specific and MUST be removed as part of
[genericizing a clone](./product-genericization-checklist.md) for any
non-banking product (they are kept only for `ZodiBank` itself):

- `loans` / `loan_plans` and `LoanPlanController`
- `dps` / `dps_plans` and `DpsPlanController`
- `fdr` / `fdr_plans` and `FdrPlanController`
- `branches` / `branch_staff` and the `branch_staff` guard
- `other_banks`
- `beneficiaries`
- `airtime_operators` / `airtime_configs`

## Genericizing the Plan pattern

`LoanPlanController`, `DpsPlanController`, and `FdrPlanController` implement
three variations of the same underlying shape: an admin-defined plan with a
rate/term/limit configuration that a user subscribes to or is issued
against. Before this becomes a reusable template, this pattern MUST be
extracted into one domain-neutral `Plan` concept (configurable
name/description/price-or-rate/term/limits/features JSON) that a product
repurposes for its own use case — a subscription tier, a membership plan, an
insurance product, a service package — rather than each product inheriting
three literal banking plan types. This extraction is a one-time base-cleanup
task, not a per-product task; once genericized, `ZodiBank` itself re-adds
its loan/DPS/FDR-specific behavior on top of the generic `Plan` model rather
than the base engine carrying banking-specific plan types for every product.

## One-time base cleanup (fix once, before first clone)

The following tech debt is fixed exactly once in the base codebase before
it is cloned for the first product — never repeated per product:

1. **Remove `webpack.mix.js`.** Vite 5 is the actual build tool; the legacy
   Mix config is dead and confusing to keep.
2. **Genericize hardcoded product identity.** `systemDetails()` in
   `app/Http/Helpers/helpers.php` hardcodes `'name' => 'viserbank'`. Audit
   and rename every other hardcoded `viserbank`/banking-specific string
   across `routes/web.php`, `routes/admin.php`, and Blade namespaces before
   the base is considered clone-ready. This produces the actual "sanitized
   base" every product clones from.
3. **Document (or fix) runtime config file mutation.** `GeneralSettingController@generalUpdate`
   mutates physical config files at runtime via
   `file_put_contents(config_path('timezone.php'), ...)`. On some shared
   hosting environments the application's file-write permissions to its own
   `config/` directory are restricted. Document this as a known limitation
   requiring writable `config/` permissions at install time in every
   product's deployment guide (see
   [`../templates/deployment-template.md`](../templates/deployment-template.md)),
   or replace it with a DB-driven config value read at boot — track the
   decision as an ADR if changed, since it affects every product cloned from
   the base afterward.
4. **Document the sitemap/robots file-write requirement.** The
   sitemap/robots admin editor overwrites `sitemap.xml`/`robots.xml`
   directly on disk rather than rendering them dynamically. This is
   acceptable for the shared/VPS hosting target audience but requires the
   web server document root to be writable by the PHP process — document
   this explicitly in the deployment checklist so it is not discovered as a
   support ticket after a buyer's first SEO configuration attempt.

## Layering a product's domain modules onto the sanitized base

Once a product's clone has been genericized (tables stripped, identity
renamed, Plan pattern available), that product's own domain modules —
ZodiMed's patient records, ZodiBank's re-added loan/DPS/FDR modules,
ZodiHotel's room inventory — are added as new, clearly bounded modules per
[`../development/coding-standards-php-laravel.md`](../development/coding-standards-php-laravel.md)
and [`../templates/module-template.md`](../templates/module-template.md),
built against the product's own
[`docs/products/<product>/SPEC.md`](../products/) data model. These new
modules consume the inherited engine's services (wallet, notifications,
RBAC, settings) the same way any other application code does — they never
fork or duplicate an inherited controller/model to add domain behavior.

## Mandatory isolation before touching any live reference codebase

**This rule is universal and non-negotiable, for every product, present or
future — not limited to qfsfountains.** Whenever a product is built
starting from an existing **live, running** reference codebase — the
sanitized qfsfountains base, Pay Secure (ZodiBank), Ultimate POS
(ZodiCore), novavest (ZodiCapital/ZodiYield), or any future live site used
as a foundation or reference — the live reference site itself is **never**
edited, migrated against, or logged into for anything beyond read-only
inspection. This was added after a real incident: an early build pass on
ZodiCapital/ZodiYield ran migrations and file edits directly against the
live `novavest.org` codebase and database before this rule existed — see
`BUILD_STATE.md`'s Flagged Items for the full accounting. Rule 6 in
`BUILD_STATE.md` already required this for `Live — Extend Only` products
(ZodiTrack); this section generalizes it to *any* live reference codebase,
including internal ones like novavest that aren't formally `Live — Extend
Only` in the catalog sense.

The mandatory first step, before any modification:

1. **Copy first.** Copy the reference codebase into the product's own
   isolated subfolder at `/home/script/public_html/<product-slug>/`
   (excluding `vendor/`/`node_modules/`/`.git` — reinstall those fresh in
   the copy rather than carrying them over). The original reference site's
   folder is read-only from this point forward.
2. **Isolated database, always.** Create a new, separate MySQL database
   for the product's copy, granted to the `script` DB user. Export the
   reference codebase's schema and non-live reference/config data (i18n
   strings, gateway/plan configuration, seed lookup tables) — never live
   user accounts, financial transactions, or other real business data —
   and import that clean structure into the new database.
3. **Point the copy's runtime `.env` at its own isolated database**,
   clearing out any connection values copied from the reference site.
   Some inherited codebases load their real `.env` from a non-obvious path
   (e.g. novavest's `bootstrap/app.php` calls
   `$app->loadEnvironmentFrom('vendor/psr/log/.env')` rather than the
   project-root `.env`) — locate and update the actual file the
   application reads, not just the conventional root `.env`, and verify by
   booting the app rather than assuming the root file is authoritative.
4. **Rebuild feature work as fresh migrations/seeders inside the isolated
   copy**, never by copying live data across, and create a fresh
   admin/owner account independently rather than reusing the reference
   site's live credentials.
5. **Confirm the isolated copy boots and its core admin flow works live**,
   inside its own folder and database, before any further feature work
   proceeds. If the reference codebase turns out to be a licensed
   commercial script with an activation/license-check mechanism (as
   novavest's ViserLab/HYIPLAB base was found to have — see
   `BUILD_STATE.md`), do **not** attempt to bypass, patch around, or copy
   activation state from the licensed reference installation to make the
   isolated copy pass that check — stop and flag it as a licensing/
   provenance concern requiring a human decision, the same as any other
   piracy/licensing finding in this handbook.

## Related standards

- [`overview.md`](./overview.md)
- [`single-tenant-deployment-model.md`](./single-tenant-deployment-model.md)
- [`frontend-backend-bridge.md`](./frontend-backend-bridge.md)
- [`product-genericization-checklist.md`](./product-genericization-checklist.md)
- [`../standards/admin-configuration-baseline.md`](../standards/admin-configuration-baseline.md)
- [`../standards/payment-gateways.md`](../standards/payment-gateways.md)
- [`../standards/wallet-system.md`](../standards/wallet-system.md)

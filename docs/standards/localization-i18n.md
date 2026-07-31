# Localization, Multi-Currency, and Multi-Company Scoping Standards

Zodize products are sold as self-hosted source code into markets with
different languages, writing directions, and currencies, to buyers who may
themselves operate multiple companies, branches, or locations under one
deployed instance. This document defines the multi-language standard
(inherited from the base codebase's existing i18n engine — see
[`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)),
the multi-currency standard, and the multi-company/multi-branch data-scoping
standard every product MUST implement within its own single deployment. This
is not multi-tenancy — see
[`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md);
every product here is one buyer's one deployment.

## Multi-language standard: inherited i18n engine

Every product inherits the base codebase's existing, working i18n engine —
this is not a system to design from scratch:

- **`Admin/LanguageController.php`** and **`Models/Language.php`** provide
  the admin-facing language management screen: adding a language, setting
  it active/inactive, setting the default language, and editing translation
  strings through the admin panel (not a code editor).
- **Translation storage**: `core/lang/{code}.json` — one flat JSON file per
  language code (e.g. `en.json`, `fr.json`, `ar.json`), key-value pairs of
  the original English string to its translation. This is Laravel's
  JSON-based translation file convention, not a nested/namespaced key
  structure — new strings are added by wrapping UI text in `__('Original
  English String')` (or the Blade `@lang()` equivalent) in the source
  templates, which auto-populates the key in `en.json`; the admin
  translation screen then lets a non-technical buyer supply the translated
  value for each other active language directly from the panel.
- **`LanguageMiddleware`** resolves the active language per request (from
  the user's session/cookie preference, defaulting to the admin-configured
  default language) and loads the corresponding `core/lang/{code}.json`
  file for that request — this is the mechanism every product uses; it is
  not replaced or reimplemented per product.
- Multi-language is **mandatory, not optional**, for every product: every
  product ships with English as the base locale plus the base codebase's
  already-integrated language management screen enabled and visible in the
  admin panel, so a buyer can add their market's language(s) — and edit
  every string — without ever touching code. A product's own
  [`SPEC.md`](../products/) documents which languages ship pre-translated
  in its `DemoSeeder`/`core/lang/` files, not whether the language system
  itself exists.
- New UI strings introduced by a product's own domain modules (see
  [`../architecture/product-genericization-checklist.md`](../architecture/product-genericization-checklist.md))
  MUST use the same `__()`/`@lang()` convention as the inherited engine —
  no hardcoded UI strings in new Blade views or controller response
  messages, so every product-specific string is translatable through the
  same admin screen as the inherited engine's own strings.

### RTL support

- Every product MUST support right-to-left (RTL) layout for RTL locales
  (Arabic, Hebrew) an admin activates via the language management screen.
  RTL is driven by the `dir="rtl"` attribute on the document root when an
  RTL locale is active; the layout system — sidebar position, breadcrumb
  chevron direction, form field alignment, directional icon mirroring — MUST
  use logical CSS properties (`margin-inline-start`, not `margin-left`)
  rather than physical properties, so the layout flips correctly without
  component-specific RTL overrides. See
  [`../design-system/responsive-standards.md`](../design-system/responsive-standards.md).

### Date and number formatting

- Dates and numbers are formatted according to the active language's
  locale convention (resolved by `LanguageMiddleware`, above) — a global
  per-request setting in this single-tenant model, not a per-user stored
  preference layered over a shared tenant default. A buyer's staff using
  the admin panel see dates/numbers formatted per the deployment's currently
  active language.
- Underlying data is always stored in locale-neutral form: dates as
  standard `timestamp`/`date` database columns, numbers as raw numeric
  types, currency amounts per the minor-units rule below — locale-specific
  formatting happens at render time only.

## Multi-currency standard

### What the base codebase actually provides today

The inherited base codebase, as audited, is **single-base-currency**: the
operating currency (`cur_text`, `cur_sym`) is a single value stored in the
`general_settings` row, configured once by the buyer at setup. Multi
-currency handling exists only at the payment-gateway layer, via the
`gateway_currencies` table, which stores a per-gateway exchange rate used
to convert an incoming payment in a foreign currency into the deployment's
one base currency at the moment of transaction — it does not give the
product a multi-currency wallet where a user could hold balances in more
than one currency simultaneously. This is a real gap relative to a "true"
multi-currency product, not an already-solved problem — see
[`../standards/wallet-system.md`](../standards/wallet-system.md#multi-currency-gap)
for the explicit decision on whether and how each product closes it.

### Baseline rules that apply regardless of how the gap above is resolved

- All monetary amounts MUST be stored in **minor units** (integer cents, or
  the equivalent smallest unit for the currency — no decimal subdivision for
  currencies like JPY), never as floating-point decimal values.
- Every monetary field MUST be stored alongside its ISO 4217 currency code.
  For products remaining single-base-currency (the default, inherited
  behavior), this is simply the deployment's one configured currency
  recorded per transaction row for historical accuracy even if the base
  currency setting is later changed — never a bare numeric amount with an
  implied, unstated currency.
- **Display formatting**: monetary values are formatted using the active
  language's locale convention (above) combined with the value's currency
  code — symbol, decimal separator, grouping separator, and symbol position
  follow locale convention.
- **Exchange rates** used at the gateway layer (`gateway_currencies`) are
  timestamped per transaction so a historical payment's conversion can be
  reconstructed exactly as it occurred, never recalculated against today's
  rate.
- **Financial products have stricter requirements.** Products handling
  regulated financial transactions — most notably ZodiBank, ZodiTrade,
  ZodiXchange, ZodiCapital, ZodiYield — MUST state explicitly in their own
  `SPEC.md` whether they require a true multi-currency wallet (and therefore
  extend the base wallet engine per
  [`wallet-system.md`](./wallet-system.md#multi-currency-gap)) or operate
  single-base-currency with gateway-layer conversion only. This is not
  assumed either way by this baseline standard.

## Multi-company / multi-branch data scoping

A single deployed product instance belongs to one buyer's business, but that
business may itself operate multiple companies, branches, or locations
(e.g. a hotel group with 12 properties, a retailer with multiple
storefronts). This is scoping **within** one deployment, not tenancy — see
[`../architecture/single-tenant-deployment-model.md`](../architecture/single-tenant-deployment-model.md#what-replaces-multi-company--multi-branch-scoping).

- Every product that supports multi-company/multi-branch operation MUST
  model an explicit scoping entity (e.g. `Company`, `Branch`, `Location`) as
  a first-class record, and every scoped business record (accounts, orders,
  staff assignments, inventory) MUST carry a foreign key to it — scoping
  MUST NOT be inferred implicitly from the current user's session alone.
- A company/branch switcher (a dropdown in the sidebar header or page
  header, per [`navigation-standards.md`](./navigation-standards.md)) lets a
  staff member with access to more than one branch switch their working
  context — there is no tenant switcher, because there is no tenant.
- A user's access to each company/branch is governed by the RBAC model in
  [`../security/rbac-permissions.md`](../security/rbac-permissions.md),
  with the possibility of a user holding different roles at different
  branches (e.g. branch manager at one location, read-only viewer at
  another).
- Reports and dashboards MUST support both a single-branch scoped view
  (default) and, for users with cross-branch permission, an aggregated
  "All branches" view, clearly labeled as such (e.g. "Aggregated across 12
  locations").
- Currency and language settings (above) are deployment-wide (one
  `general_settings` row), not per-branch, given the single-base-currency
  reality documented above; a product whose spec requires per-branch
  currency MUST document that as an extension to the inherited
  `GeneralSetting` model, not assume it already varies per branch.

## Roadmap

- The translation-management workflow for buyers managing many languages at
  scale (bulk export/import of `core/lang/{code}.json`, machine-translation
  assist) is a candidate enhancement to the inherited `LanguageController`,
  not yet scoped per product.
- The multi-currency wallet gap (above) is tracked per financial product in
  that product's own `SPEC.md` roadmap section pending an ADR if the base
  wallet engine itself is extended — see
  [`wallet-system.md`](./wallet-system.md#multi-currency-gap).

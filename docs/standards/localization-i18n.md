# Localization, Multi-Currency, and Multi-Tenancy Scoping Standards

Zodize products are sold into markets with different languages, writing
directions, currencies, and organizational structures. This document
defines the multi-language standard, the multi-currency standard, and the
multi-company/multi-branch data-scoping standard that every product MUST
implement — these are not optional "internationalization polish," they are
baseline requirements for enterprise SaaS sold globally.

## Multi-language standard

- All user-facing strings MUST be externalized as translation keys — no
  hardcoded UI strings in component templates or backend response messages.
  Translation keys follow a namespaced, dot-delimited structure:
  `<domain>.<screen_or_component>.<element>`, e.g.
  `invoices.list_page.empty_state_title`,
  `auth.login_form.password_label`,
  `common.actions.save` (the `common` namespace holds strings reused across
  many screens — buttons, generic labels — to avoid duplicate keys with
  drifting translations).
- Keys MUST NOT be constructed by free-form string concatenation at runtime.
  A pattern like `t('status.' + status)` is acceptable ONLY when every
  possible value of `status` is enumerated and pre-registered as a real key
  in the translation files — dynamically-built keys that bypass the
  translation file's static key set break automated missing-translation
  tooling.
- Pluralization and interpolation use the ICU MessageFormat syntax
  supported by the shared i18n library (per `docs/development/`), e.g.
  `{count, plural, one {# item selected} other {# items selected}}` —
  string concatenation to build plural forms (`count + ' item(s)'`) is
  prohibited because it doesn't generalize to languages with more than two
  plural forms.
- Every product MUST ship English (`en`) as the base locale with 100%
  key coverage; additional locales are enabled per product per the
  product's `SPEC.md` market requirements. A missing translation key falls
  back to the base English string, never to a raw key name, in production.

### RTL support

- Every product MUST support right-to-left (RTL) layout for RTL locales
  (Arabic, Hebrew, and any future RTL locale a product enables). RTL is
  driven by the `dir="rtl"` attribute on the document root when an RTL
  locale is active, and the entire layout system — sidebar position
  (flips to the right), breadcrumb chevron direction, form field alignment,
  icon mirroring for directional icons (back/forward arrows, chevrons) —
  MUST use logical CSS properties (`margin-inline-start`, not
  `margin-left`) rather than physical properties, so the layout flips
  correctly without component-specific RTL overrides. This is defined in
  full at
  [`../design-system/responsive-standards.md`](../design-system/responsive-standards.md);
  this document establishes only that RTL support is a REQUIREMENT, not an
  enhancement, for any product enabling an RTL locale.
- Non-directional content (numbers, currency codes, embedded LTR content
  like email addresses) remains LTR even within an RTL layout, per Unicode
  bidi algorithm defaults — the i18n library MUST wrap such fragments in
  the appropriate bidi-isolation markup automatically rather than requiring
  manual handling per screen.

### Date, number, and currency formatting

- All dates, numbers, and currency values MUST be formatted using the
  **user's own locale preference**, not the server's locale, not the
  tenant's default locale, and not the browser's raw `Accept-Language`
  header alone — the user has an explicit locale preference stored on
  their profile (defaulting to their browser locale at first login, or
  their organization's default locale, whichever the product's onboarding
  flow captures), and every formatted value in the UI MUST derive from that
  stored preference.
- Two users in the same tenant viewing the same record MAY see different
  date/number formatting if their personal locale preferences differ (e.g.
  a US-based user sees `$1,234.56` and `03/14/2026`; their German colleague
  in the same tenant sees `1.234,56 $` and `14.03.2026`) — formatting is a
  presentation-layer concern per user, never a stored/computed value per
  tenant.
- Underlying data is always stored and transmitted (API payloads) in
  locale-neutral form: dates as ISO 8601 (`2026-03-14T00:00:00Z`), numbers
  as raw numeric types, currency amounts per the minor-units rule below —
  locale-specific formatting happens exclusively at render time on the
  client.

## Multi-currency standard

- All monetary amounts MUST be stored in **minor units** (integer cents, or
  the equivalent smallest unit for the currency — e.g. no decimal
  subdivision for currencies like JPY) — never as floating-point decimal
  values, to avoid rounding-error classes of bugs. A `$12.50` amount is
  stored as the integer `1250` alongside a currency code.
- Every monetary field MUST be stored alongside its ISO 4217 currency code
  (e.g. `amount_minor: 1250, currency: "USD"`) — a bare numeric amount with
  no currency code is not a valid Zodize monetary value.
- **Display formatting**: monetary values are formatted for display using
  the user's locale (per the date/number rule above) combined with the
  value's own currency code — the currency symbol, decimal separator,
  grouping separator, and symbol position all follow locale convention
  (e.g. `1250` minor units in USD renders as `$12.50` for an en-US user and
  `12,50 $US` for an fr-FR user viewing the same underlying record).
- **Exchange rates**: for products that support multi-currency accounts,
  conversions, or cross-currency reporting, exchange rates MUST be sourced
  from a single rate-provider integration (defined per product in its
  `SPEC.md`), stored as historical, timestamped rate records (never
  overwritten), so any historical transaction's conversion can be
  reconstructed exactly as it occurred rather than recalculated against
  today's rate. A displayed "converted" value MUST be visually marked as an
  approximation (e.g. "≈ €1,150.00") with the rate and timestamp used
  available on hover/tooltip.
- **Financial products have stricter requirements.** Currency handling
  described here is the company-wide floor. Products handling regulated
  financial transactions — most notably ZodiBank — layer additional
  requirements on top (ledger-grade precision, reconciliation, regulatory
  rounding rules, dual-entry accounting guarantees) that are specified in
  that product's own specification; see
  [`../products/ZodiBank/SPEC.md`](../products/ZodiBank/SPEC.md). Any
  product handling money MUST confirm in its own `SPEC.md` whether it
  inherits only this baseline standard or must additionally comply with
  ZodiBank-grade financial precision requirements.

## Multi-company / multi-branch data scoping

Most Zodize tenants are not a single flat organization — they operate
multiple companies, branches, locations, or cost centers under one tenant
(e.g. a hotel group with 12 properties, a bank with regional branches, a
retailer with multiple storefronts). This is distinct from multi-tenancy
itself (one tenant per customer organization, defined in
[`../architecture/multi-tenancy.md`](../architecture/multi-tenancy.md)) —
this section defines scoping WITHIN a single tenant.

- Every product that supports multi-company/multi-branch operation MUST
  model an explicit scoping entity (e.g. `Company`, `Branch`, `Location`)
  as a first-class record, and every scoped business record (accounts,
  orders, staff assignments, inventory) MUST carry a foreign key to it —
  scoping MUST NOT be inferred implicitly from the current user's session
  alone.
- The top bar's org/tenant switcher (per
  [`navigation-standards.md`](./navigation-standards.md#top-bar)) handles
  switching between TENANTS. Switching between companies/branches WITHIN
  one tenant is a separate, secondary switcher — typically a dropdown in
  the sidebar header or page header — since a user may have access to
  several branches within one tenant without those branches being separate
  tenants.
- A user's access to each company/branch is governed by the RBAC model in
  `docs/security/`, with the possibility of a user holding different roles
  at different branches within the same tenant (e.g. branch manager at one
  location, read-only viewer at another).
- Reports and dashboards MUST support both a single-branch scoped view
  (default, matching the currently selected branch) and, for users with
  cross-branch permission, an aggregated "All branches" view — the
  aggregation MUST clearly label itself as such (e.g. a badge or subtitle
  "Aggregated across 12 branches") so a user does not mistake a rolled-up
  figure for a single branch's number.
- Currency and locale settings (above) MAY be configured per branch (e.g. a
  hotel group operating in both the US and Mexico shows USD at one property
  and MXN at another) independent of the tenant's overall default — the
  scoping entity, not the tenant record, is the source of truth for a given
  business record's operating currency.

## Roadmap

- A shared translation-management workflow (how new keys get added,
  reviewed, and sent to translators) is planned but not yet documented;
  until it exists, product teams add keys directly to the base `en` locale
  file and flag missing non-English coverage in their product's `SPEC.md`
  roadmap section.

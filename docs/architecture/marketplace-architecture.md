# Marketplace Architecture

> Governs how plugins, themes, and integrations built on
> [`plugin-architecture.md`](./plugin-architecture.md) reach the buyers of
> **one specific product**. Each product that ships a plugin system has its
> own marketplace — there is no ZodiCore-hosted catalog shared across every
> Zodize product, because there is no shared ZodiCore platform; see
> [`single-tenant-deployment-model.md`](./single-tenant-deployment-model.md).
> A buyer's admin panel for, say, ZodiCommerce shows only plugins built for
> ZodiCommerce.

Not every product needs this — see the note at the top of
[`plugin-architecture.md`](./plugin-architecture.md). This document applies
to any product whose own `SPEC.md` states it ships a plugin/extension
marketplace.

## Listing model

- A product's marketplace lists three package types: **Plugins**
  (functional extensions, per
  [`plugin-architecture.md`](./plugin-architecture.md)), **Themes** (Blade
  component or Bootstrap SCSS style overrides within the design tokens
  defined in [`../design-system/`](../design-system/) — a theme MUST NOT
  override accessibility-critical tokens such as minimum contrast ratios), and
  **Integrations** (pre-built connectors to third-party services, packaged
  as a specialization of a plugin with `permissions` scoped to `webhooks.*`
  and outbound API credentials).
- Because a plugin is built for one product's own codebase (its manifest's
  `min_product_version`/`max_product_version` target that product
  specifically, per
  [`plugin-architecture.md`](./plugin-architecture.md#plugin-manifest-format)),
  there is no cross-product listing filter to maintain — every listing in a
  product's marketplace is, by construction, for that product.
- A listing page MUST show: vendor identity (verified, per submission
  vetting below), version history with changelogs, the full permission list
  the buyer's admin will be asked to consent to at install, pricing, and a
  support contact. Listings MUST NOT omit the permission list from the
  public page — it is visible before install, not only at install time.

## Review model

- Every published plugin version carries an aggregate star rating (1-5) and
  written reviews from buyers' admins who have the plugin installed (review
  eligibility is enforced server-side against install records — no
  unverified reviews).
- Vendors MAY respond publicly to a review once; review text is subject to
  the same content-moderation policy as any other user-generated content and
  is audit-logged if removed by a marketplace moderator, per
  [`../security/audit-logging.md`](../security/audit-logging.md).
- A plugin whose average rating falls below 2.5 across at least 10 reviews,
  or that accumulates 3 or more unresolved security reports, is
  automatically flagged for moderator re-review and hidden from new
  installs (existing installs are not force-removed, but installing buyers
  are notified) until the flag is cleared.

## Revenue share model

- The default revenue share is **80/20**: the publishing vendor retains 80%
  of net revenue from a paid listing (purchase price or subscription fee,
  net of payment-processor fees), and Zodize retains 20% as the marketplace
  operator's share.
- This split is configurable per vendor agreement (e.g. a strategic
  partnership negotiated at a different rate), but 80/20 is the standing
  default applied to any vendor who has not negotiated a bespoke agreement,
  and MUST be stated as such — never left unspecified — in the vendor
  onboarding terms.
- Free plugins carry no revenue share by definition; Zodize MAY still apply
  the standard submission/vetting pipeline fee-free.
- Payouts are processed monthly, net-30 from the close of the revenue month,
  through the payment-gateway integrations documented in
  [`../standards/payment-gateways.md`](../standards/payment-gateways.md) —
  the marketplace's own payout mechanism is a Zodize-operated service
  (since the marketplace itself, unlike the products it sells extensions
  for, IS a service Zodize runs), distinct from any individual buyer's
  self-hosted product instance.

## Submission, vetting, and security review pipeline

A plugin MUST pass every stage below before it appears in a product's
public marketplace catalog:

1. **Manifest validation** (automated): `zodize-plugin.json` is
   schema-validated — required fields present, `slug` uniqueness checked
   within that product's marketplace, `permissions` entries all resolve to
   real, cataloged permissions on that product.
2. **Static security scan** (automated): dependency audit (per
   [`../security/security-standards.md`](../security/security-standards.md#dependency-scanning))
   against the plugin's own `composer.json`/`package.json`, static analysis
   for common vulnerability patterns (raw SQL, unescaped output, use of
   `eval`/dynamic includes), and a check that declared migrations only touch
   `plugin_{slug}_`-prefixed tables per
   [`plugin-architecture.md`](./plugin-architecture.md#migrations-routes-and-views).
   A failure here blocks submission outright; there is no manual override.
3. **Manual security review** (human): a Zodize marketplace reviewer
   installs the plugin against a clean instance of that product, exercises
   its declared hooks and routes, and confirms the requested `permissions`
   list matches observed behavior — a plugin requesting `orders.view` but
   also writing to an unrelated table is rejected. Plugins for
   financial-grade products (ZodiBank, ZodiTrade, ZodiXchange, ZodiCapital,
   ZodiYield) undergo an additional review pass against the OWASP mapping
   in
   [`../security/security-standards.md`](../security/security-standards.md#owasp-top-10-mapping).
4. **Vendor identity verification**: a business or individual developer
   identity check (comparable to a KYC-lite check) before a vendor can
   publish any paid listing, to establish accountability for the revenue
   share and support obligations above.
5. **Publish**: on passing all stages, the version becomes installable by
   any buyer of that product. Subsequent versions of an already-approved
   plugin re-enter at stage 2 automatically; stage 3 (manual review) is
   re-triggered only when the diff touches `permissions`, `hooks`, or
   migrations, to keep routine patch releases fast.

A plugin that fails manual review receives a structured rejection reason and
may resubmit after addressing it; there is no cap on resubmission attempts,
but each resubmission re-enters at stage 1.

## Licensing enforcement

- Paid plugins are license-gated per installed instance: install issues a
  signed license token (a per-install identifier, plugin slug, plan tier,
  expiry) that the plugin runtime validates on each enable and on a
  recurring background check (daily); an expired or revoked license
  auto-disables the plugin (data retained, per the disable behavior in
  [`plugin-architecture.md`](./plugin-architecture.md#plugin-lifecycle))
  rather than deleting anything. This mirrors how the base codebase's own
  product-level license/purchase-code validation works (see
  [`../architecture/single-tenant-deployment-model.md`](./single-tenant-deployment-model.md#licensing-and-update-model)),
  applied one layer down at the plugin level.
- License validation happens server-side, inside the product's own plugin
  runtime — never trusting a client-supplied "licensed: true" flag from the
  plugin's own frontend bundle.
- Refunds/chargebacks on a plugin purchase immediately revoke the
  corresponding license token; the vendor's payout for that transaction is
  reversed in the next payout cycle.

## Related standards

- [`plugin-architecture.md`](./plugin-architecture.md)
- [`overview.md`](./overview.md)
- [`../standards/payment-gateways.md`](../standards/payment-gateways.md)
- [`../security/security-standards.md`](../security/security-standards.md)
- [`../security/audit-logging.md`](../security/audit-logging.md)

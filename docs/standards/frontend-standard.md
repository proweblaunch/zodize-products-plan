# Frontend Standard: One Shared Design System, Twenty Products

> Which codebase every product's public-facing pages are built from, and
> what's actually confirmed to exist in it today. Companion to
> [`../architecture/frontend-backend-bridge.md`](../architecture/frontend-backend-bridge.md)
> (the CMS-wiring gap) and [`../design-system/`](../design-system/) (the
> token/visual-language spec).

## The rule

Every product's public-facing marketing pages — homepage, pricing,
features, about, contact, legal pages, per
[`../templates/marketing-website-template.md`](../templates/marketing-website-template.md) —
use **one shared frontend shell**, not a per-product reimplementation. That
shell is the codebase at `/home/zodize/public_html` on the build server: a
Laravel + Blade application carrying the design tokens
([`../design-system/design-tokens.md`](../design-system/design-tokens.md))
mapped onto **Bootstrap 5** (SCSS variable overrides/a shared theme
stylesheet, per
[`coding-standards-frontend.md`](../development/coding-standards-frontend.md))
plus a reusable `x-zodize.*` Blade component library for markup reuse — the
component library is a thin Blade-partial convenience layer on top of
Bootstrap, it does not replace Bootstrap's grid/utility classes or pull in
Tailwind. **Correction**: an earlier pass of this document described this
shell as Tailwind-based; that was wrong for the standard going forward —
treat every `x-zodize.*` component as Bootstrap-5-markup-plus-tokens, and
re-confirm the literal CSS framework already compiled into
`/home/zodize/public_html` on disk next time the build VPS is reachable,
correcting this note if the live shell still needs to be migrated off
Tailwind. A product's own build clones this shell alongside the sanitized
qfsfountains base (or, for products on an alternate base per
[`base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)'s
documented exceptions — Pay Secure for ZodiBank, novavest for
ZodiCapital/ZodiYield — the shell is still layered on top the same way),
swapping in that product's own branding/content via the admin CMS per
[`frontend-backend-bridge.md`](../architecture/frontend-backend-bridge.md) —
never by forking the Blade templates per product.

## What's confirmed to exist today

A direct filesystem audit of `/home/zodize/public_html/resources/views/components/zodize/`
confirms exactly **eight** components:

| Component | File | Notes |
|---|---|---|
| `button` | `button.blade.php` | |
| `badge` | `badge.blade.php` | |
| `card` | `card.blade.php` | |
| `input` | `input.blade.php` | |
| `textarea` | `textarea.blade.php` | |
| `container` | `container.blade.php` | |
| `section` | `section.blade.php` | |
| `nav.header` | `nav/header.blade.php` | 26KB — substantial, likely covers responsive/mobile nav already |

The codebase also carries `ZODIZE_DESIGN_SYSTEM_SPEC.md` and
`ZODIZE_FRONTEND_AUDIT.md` at its root, documenting the token set and a
prior audit of the frontend's own state — a follow-up session should read
both in full before building new components, so new work matches the
system's own documented intent rather than reverse-engineering it from the
Blade files alone.

**No other component currently exists in this directory** — see the
correction in
[`frontend-backend-bridge.md`](../architecture/frontend-backend-bridge.md#the-gap-stated-plainly)
for the specific list of components an earlier, unaudited draft of this
handbook incorrectly claimed were already built (`nav.footer`, `hero`,
`feature-grid`, `testimonial`, `pricing-table`, `faq`, `cta-band`,
`stat-block`, `logo-cloud`, `breadcrumbs`, `empty-state`, `404`). Building
any marketing page that needs one of these MUST start by building the
missing component against the confirmed eight components' conventions —
not by assuming it already exists.

## How a product adopts this shell

1. Clone `/home/zodize/public_html`'s `resources/`, `public/`, `routes/`
   (the marketing-facing ones), and `config/` (Bootstrap SCSS/Vite build
   config) alongside that product's own backend base (qfsfountains-derived,
   Pay Secure, Ultimate POS, or novavest, per that product's own
   [`SPEC.md`](../products/)).
2. Wire the CMS bridge per
   [`frontend-backend-bridge.md`](../architecture/frontend-backend-bridge.md)
   so the product's own admin panel can edit page content through the
   shared components rather than requiring a Blade edit per product.
3. Set the product's own accent color/logo/icon mark within the shared
   token system, per
   [`../design-system/brand-standards.md`](../design-system/brand-standards.md)'s
   per-product accent derivation — never a parallel palette.
4. Build any product-specific marketing section (e.g. a pricing table, if
   that component doesn't exist yet) once, in the shared component library
   convention, so every other product benefits from it too — a new
   component built for one product's marketing site is contributed back to
   the shared `x-zodize.*` library, not duplicated per product.

## Applies to every product being built or touched

This standard is not limited to new-from-scratch products. Per the current
build order, **ZodiCore's and ZodiTrack's public front-facing pages** are
explicitly in scope for this treatment as part of their own current work
(ZodiCore's public pages currently 404 and must be replaced with this
shell; ZodiTrack's live public pages — `air-freight.php`,
`ocean-freight.php`, etc. — are a candidate for this shell as an additive,
non-destructive enhancement per its `Live — Extend Only` status, not a
requirement to rebuild them immediately — see
[`BUILD_STATE.md`](../../BUILD_STATE.md)).

## Related standards

- [`../architecture/frontend-backend-bridge.md`](../architecture/frontend-backend-bridge.md)
- [`../design-system/`](../design-system/)
- [`../templates/marketing-website-template.md`](../templates/marketing-website-template.md)
- [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)

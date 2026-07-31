# Brand Standards

## Source of Truth Note

This document, together with the rest of `docs/design-system/`, is the
**canonical Zodize design specification**. The brief for this handbook
references a live marketing site at `/home/zodize/public_html` as the
company's design system of record; that site is not available in this
environment and is not assumed to exist for the purposes of building
products. Until the two are formally reconciled, **this document governs**.
If a future reconciliation surfaces a conflict between this handbook and the
live site, the conflict MUST be resolved via an Architecture Decision Record
in `docs/decisions/` per [`CONTRIBUTING.md`](../../CONTRIBUTING.md) — never
by silently adopting whichever version was seen last. No engineer or agent
may treat the marketing site as authoritative over this file.

## Brand Principles

Zodize builds the operating infrastructure for regulated, high-stakes
industries — banking, trading, healthcare, government services, legal
practice. The brand MUST read as **enterprise software, not a startup demo**.
Every product, regardless of its target industry, is built on four
principles:

1. **Precision over decoration.** Every visual element earns its place by
   communicating information or hierarchy. Zodize does not use ornamental
   gradients, illustrations, or motion that exist only to look "modern."
2. **Confidence without noise.** A near-black, low-chroma dark theme is the
   default surface for every product (see [Dark Theme](./dark-theme.md)).
   Color is spent deliberately — on the single accent hue, on semantic
   states, and nowhere else.
3. **One system, twenty products.** A user who has used ZodiBank must feel
   immediately competent in ZodiMed. Typography, spacing, motion, and
   component behavior are identical across every product. Only the accent
   hue and the product's icon mark change.
4. **Density with clarity.** Zodize products are used by professionals for
   hours a day, not browsed for minutes. The system favors information
   density (see [Spacing & Layout](./spacing-layout.md)) over the generous
   whitespace of consumer marketing sites, while never sacrificing legibility
   or touch/click target size (see [Accessibility](./accessibility.md)).

## Master Brand Palette

Zodize's brand color is an indigo-violet, chosen for its association with
trust and technical precision without the coldness of pure blue or the
volatility connotations of pure violet. This is the **one color every
Zodize product shares**, used as the platform-level brand mark (ZodiCore,
the handbook, internal tooling, auth screens) and as the default accent for
any product that has not yet declared its own in `docs/products/<product>/SPEC.md`.

| Token | Hex | HSL | Usage |
|---|---|---|---|
| `--zdz-brand-primary` | `#6366F1` | `hsl(239, 84%, 67%)` | Platform brand mark, default accent, primary buttons on unbranded surfaces |
| `--zdz-brand-primary-hover` | `#818CF8` | `hsl(234, 89%, 74%)` | Hover state of the above |
| `--zdz-brand-primary-active` | `#4F46E5` | `hsl(243, 75%, 59%)` | Pressed/active state |
| `--zdz-brand-ink` | `#0B0D12` | `hsl(226, 20%, 6%)` | Master near-black background |
| `--zdz-brand-paper` | `#F5F6FA` | `hsl(228, 27%, 97%)` | Master near-white surface (light theme) |

Full neutral, semantic, and surface-elevation palettes are defined in
[Color System](./color-system.md) and are shared by every product without
exception. Only the accent hue varies per product.

## Logo Usage

The Zodize wordmark and the per-product icon marks are vector assets
maintained outside this handbook (in the shared component library). Rules
that apply regardless of asset source:

- The logo MUST always render at full opacity. Never apply a tint, drop
  shadow, outline, or gradient overlay to the logo itself.
- Minimum clear space around the logo is **1x the height of the logo mark**
  on all sides. No other UI element, text, or graphic may enter that space.
- Minimum digital size is **20px mark height**. Below that, use the
  icon-only mark, never a scaled-down wordmark.
- On dark surfaces (the default), use the light/white logo variant. On
  light surfaces, use the ink variant. Never place the light variant on a
  surface lighter than `--zdz-surface-2` (see [Color System](./color-system.md)).
- The logo MUST NOT be rotated, skewed, stretched to a non-native aspect
  ratio, or recolored to a product's accent hue. The Zodize wordmark stays
  brand-indigo or neutral-ink/white in every context.
- Per-product icon marks (used in navigation, favicons, and app switchers)
  MAY use the product's accent color. The Zodize wordmark itself MAY NOT.

## Voice & Tone

Zodize product copy — UI strings, empty states, error messages,
notifications, onboarding — follows one voice across all 20 products:

- **Professional, not playful.** No exclamation points in system-generated
  copy. No emoji in product UI. No jokes in error states.
- **Confident, not hedging.** Write "This action cannot be undone," not
  "This action might not be able to be undone, just so you know."
- **Precise, not vague.** Error messages MUST state what happened and, where
  possible, what to do next: "Transfer failed: daily limit of $10,000
  exceeded." not "Something went wrong."
- **Respectful of expertise.** Zodize users are professionals — bank
  operators, clinicians, brokers, property managers. Copy does not
  over-explain domain concepts they already know, but MUST explain Zodize-
  specific concepts (permissions, plugin marketplace, tenancy) clearly on
  first encounter.

### Do

- Use active voice and second person for instructions: "Select a payment
  method to continue."
- State consequences of destructive actions before they happen, not after.
- Keep button labels as verbs: "Create Invoice," not "Invoice" or "OK."
- Use sentence case for all UI copy, including headings and buttons.

### Don't

- Don't use "Oops!", "Uh oh," or similarly casual phrasing anywhere in
  product copy.
- Don't use title case in headings or buttons ("Create New Invoice" is
  wrong; "Create new invoice" is correct).
- Don't stack more than one exclamation point-worthy tone shift per screen —
  in practice, don't use exclamation points in system copy at all.
- Don't use humor, puns, or cultural references that don't translate across
  the regulated industries Zodize serves.

## "Unmistakably Zodize" Across 20 Products

Every product MUST be immediately recognizable as a Zodize product while
remaining visually distinct enough that a user never confuses which product
they are in. This is achieved through a strict split between what is fixed
and what varies:

**Fixed across every product (never varies):**
- Typography system ([Typography](./typography.md))
- Spacing scale and grid ([Spacing & Layout](./spacing-layout.md))
- Component shapes, states, and interaction patterns ([Components](./components.md))
- Motion timing and easing ([Motion & Animation](./motion-animation.md))
- Dark-theme-first posture and elevation model ([Dark Theme](./dark-theme.md))
- Neutral/semantic color palette ([Color System](./color-system.md))

**Varies per product (the only variation permitted):**
- Accent hue (used for primary actions, active nav state, focus rings,
  selected chart series, and the product's icon mark)
- The product's icon mark itself
- Domain-specific iconography and illustration subject matter (see
  [Icons & Illustrations](./icons-illustrations.md)), while the illustration
  *style* stays fixed

A product MUST NOT introduce a second accent hue, a different type scale, a
different border radius scale, or a different motion timing table. Any such
deviation is a breaking change to the design system and requires an ADR per
[`CONTRIBUTING.md`](../../CONTRIBUTING.md).

## Per-Product Accent Colors

Each product is assigned one accent hue, rotated around the color wheel from
the master brand indigo so that no two products in the initial catalog sit
adjacent in hue. The table below is the handbook's default assignment.
**`docs/products/<product>/SPEC.md` is the authoritative source for a
product's final accent color** — if a product's SPEC.md declares a
different accent, the SPEC.md wins, and this table MUST be updated to match
in the same PR (see [`CONTRIBUTING.md`](../../CONTRIBUTING.md) on additive
changes).

| Product | Accent Hue | Rationale |
|---|---|---|
| ZodiCore | Brand Indigo (`#6366F1`) | Platform-level product; uses the master brand color directly |
| ZodiBank | Deep Blue | Trust, stability, traditional banking association |
| ZodiTrade | Cyan | Speed, live markets, clarity of data |
| ZodiXchange | Electric Blue | Adjacent to ZodiTrade but higher saturation for market infrastructure |
| ZodiEstate | Slate Teal | Grounded, physical-asset, long-term stewardship |
| ZodiMed | Teal | Clinical, calm, associated with healthcare without cliché red-cross imagery |
| ZodiCampus | Sky Blue | Approachable but institutional |
| ZodiCommerce | Coral | Retail energy, differentiated from financial products |
| ZodiBusiness | Amber | Warm, general-purpose, SMB-friendly |
| ZodiTrack | Orange | Logistics, motion, visibility |
| ZodiCapital | Deep Violet | Distinguishes fund management from retail banking blue |
| ZodiYield | Gold | Value, yield, credit |
| ZodiReach | Magenta | Marketing energy, distinct from operational products |
| ZodiPOS | Red-Orange | Urgency and speed appropriate to point-of-sale |
| ZodiLaw | Charcoal Blue | Gravity, formality, legal authority |
| ZodiHotel | Rose | Hospitality warmth |
| ZodiAgro | Green | Direct agricultural association |
| ZodiFleet | Steel Blue | Industrial, transport-oriented |
| ZodiBuild | Brick Orange | Construction association without literal skeuomorphism |
| ZodiGov | Navy | Institutional, civic authority |

Exact hex values for each accent (base, hover, active, and the
`--zdz-accent-*` token family) MUST be defined in the product's own
`SPEC.md`, following the token pattern in [Design Tokens](./design-tokens.md).
Two products MUST NOT share the same accent hex value; a new product's
SPEC.md author is responsible for checking this table before finalizing a
color.

## Open Questions

- Whether Zodize will introduce a formal secondary brand mark (a monogram
  distinct from the wordmark) for compact contexts smaller than 20px is not
  yet decided. Track in a future ADR if raised.

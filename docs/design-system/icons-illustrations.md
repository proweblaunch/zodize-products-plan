# Icons & Illustrations

## Icon System

Zodize uses a single, consistent line-icon style across all 20 products.
Icons are functional UI elements first — they communicate meaning at a
glance in dense, professional interfaces — and MUST NOT be treated as
decoration.

### Standard

| Property | Value |
|---|---|
| Grid | 24×24px artboard |
| Stroke width | 1.5px (default), 2px (only for icon sizes ≤16px, where 1.5px under-renders) |
| Style | Line/outline only — no filled icons in default state |
| Corner style | Rounded caps and joins (matches the rounded, non-sharp geometry of the type and component radius scale) |
| Color | Single color, inherits `currentColor`; icons carrying semantic meaning use the relevant semantic token (see [Color System](./color-system.md)) |
| Set | Lucide-compatible icon set (Lucide or a fork/superset maintaining the same 24px/1.5px-stroke convention) |

Lucide is the reference icon set because its 24px grid and 1.5px default
stroke match this specification without modification, it is MIT-licensed
(no per-product licensing overhead across 20 products), and it covers the
breadth of general UI icons Zodize needs (navigation, actions, status,
files, communication). Domain-specific icons not present in Lucide (e.g. a
banking-specific "wire transfer" glyph) MUST be custom-drawn to the same
24px/1.5px-stroke/rounded-join specification before being added to a
product's icon set — never sourced from a mismatched external set, which
immediately reads as visually inconsistent.

### Sizes

| Token | Size | Usage |
|---|---|---|
| `--zdz-icon-xs` | 14px | Inline with `--zdz-text-small` / caption text |
| `--zdz-icon-sm` | 16px | Inline with `--zdz-text-body-sm`, table cell icons |
| `--zdz-icon-md` (default) | 18px | Buttons, form fields, default UI |
| `--zdz-icon-lg` | 20px | Nav items, section headers |
| `--zdz-icon-xl` | 24px | Empty states (small), page-level accents |

### Usage Rules

- An icon accompanying a text label is decorative (`aria-hidden="true"`);
  an icon used alone as an interactive control (icon-only button) MUST
  carry an `aria-label` per [Accessibility](./accessibility.md) and
  [Components](./components.md).
- Never mix filled and outline icon styles within the same screen. A
  "filled" state (e.g. a favorited/starred icon) is achieved by filling
  that specific icon's interior while retaining the same stroke, not by
  swapping to a different icon family.
- Status icons (success/warning/danger/info) use the semantic color tokens
  from [Color System](./color-system.md) and MUST always be paired with a
  text label or accessible name — color/shape alone never carries meaning
  (see [Accessibility](./accessibility.md)).
- Product icon marks (the per-product logo/app-switcher glyph referenced in
  [Brand Standards](./brand-standards.md)) are the one exception to the
  line-only rule: they MAY be a solid geometric mark, since they function as
  a logo, not a UI icon.

## Illustration Style

Illustrations appear in exactly three contexts across Zodize products:
**empty states**, **onboarding flows**, and **error pages** (404, 500,
offline). Illustrations are never used as generic marketing decoration
inside application UI, and a product screen MUST NOT ship a stock or
AI-generated illustration that doesn't match the style below.

### Style Definition

| Property | Requirement |
|---|---|
| Construction | Geometric line art built from the same stroke language as the icon system (consistent stroke width, rounded joins) — illustrations read as "large, composed icons," not a different visual language |
| Color | Monochrome or duotone using the product's accent hue (see [Brand Standards](./brand-standards.md)) plus neutral surface tones — never full-color, photorealistic, or gradient-mesh illustration |
| Subject matter | Abstracted representations of the relevant domain concept (an empty inbox, a magnifying glass over a document, a disconnected plug) — never a literal photo-style scene, never people/characters (avoids localization, representation, and maintenance burden across 20 industry verticals) |
| Complexity | Simple enough to render clearly at 120–200px; illustrations are supporting context, not a focal artwork |
| Motion | Illustrations are static by default; a subtle, purposeful animation (e.g. a single element pulsing) is permitted only per the "what should animate" guidance in [Motion & Animation](./motion-animation.md), never a busy or playful animation loop |

### Explicitly Prohibited

- Generic stock illustration libraries (undraw-style flat character
  illustrations, generic "team celebrating" or "person at desk" scenes).
- Photography of any kind inside product UI (photography is permitted only
  on marketing pages, governed separately and out of scope for this
  document).
- Illustrations that introduce a color outside the product's accent hue and
  the shared neutral palette.
- Cartoonish, rounded-mascot, or "friendly onboarding buddy" character
  illustrations — these read as consumer-app patterns and directly
  contradict the "enterprise software, not startup demo" brand principle
  in [Brand Standards](./brand-standards.md).

### Context-Specific Guidance

- **Empty states**: illustration size 96–160px, paired per the structure
  defined in [Components](./components.md#empty-states).
- **Onboarding**: illustration size up to 240px, may occupy up to 40% of a
  step's layout width; still monochrome/duotone, still geometric.
- **Error pages** (404/500/offline): illustration size up to 200px,
  centered, paired with the standard error copy voice defined in
  [Brand Standards](./brand-standards.md) — precise and non-alarmist
  ("Page not found," not "Whoops, we lost it!").

## Open Questions

- A shared, versioned illustration asset library (SVG source files usable
  across all 20 products) is not yet built. Until it exists, each product
  team draws illustrations to this specification independently; this MUST
  be prioritized as shared tooling once three or more products need the
  same illustration concept (e.g. every product needs a "no results"
  illustration).

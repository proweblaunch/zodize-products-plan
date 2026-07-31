# Color System

This document is the single authoritative definition of every color token
used across Zodize products. All 20 products consume this palette
unmodified except for the accent hue, which is assigned per product in
[Brand Standards](./brand-standards.md). No product may introduce a color
outside this system without an ADR per
[`CONTRIBUTING.md`](../../CONTRIBUTING.md).

Dark theme is the primary, default theme for every Zodize product; light
theme is a fully supported secondary mode. Both palettes are defined below.
See [Dark Theme](./dark-theme.md) for the elevation model and dark-mode-
specific rules (charts, imagery) that build on this palette.

## Accessibility Baseline

Every color pairing used for text, icons, or meaningful UI state MUST meet
**WCAG 2.1 AA**, non-negotiably (see [Accessibility](./accessibility.md)):

- Normal text (<24px, or <19px bold): contrast ratio **≥ 4.5:1** against its
  background.
- Large text (≥24px, or ≥19px bold): contrast ratio **≥ 3:1**.
- UI components and graphical objects (borders, icons carrying meaning,
  focus indicators, chart data marks): contrast ratio **≥ 3:1** against
  adjacent colors.
- Every token pair listed in this document as a "text on surface" pairing
  has been selected to meet these minimums; do not substitute a token from
  a different step of the scale without re-verifying contrast.

## Neutral Scale — Dark Theme (primary)

Dark theme never uses pure black (`#000000`). The base ink is a near-black
with a slight blue undertone, matching `--zdz-brand-ink` from
[Brand Standards](./brand-standards.md), which reduces eye strain and OLED
smearing while keeping perceived contrast high.

| Token | Hex | Usage |
|---|---|---|
| `--color-neutral-0` (dark bg) | `#0B0D12` | App background (`--zdz-brand-ink`) |
| `--color-neutral-50` | `#12141C` | Sunken/inset surfaces (code blocks, wells) |
| `--color-neutral-100` | `#1A1D27` | Card / panel surface |
| `--color-neutral-200` | `#242733` | Raised surface, table row hover |
| `--color-neutral-300` | `#2E323F` | Border, divider (subtle) |
| `--color-neutral-400` | `#3D4150` | Border (default), input border |
| `--color-neutral-500` | `#565B6D` | Disabled text, placeholder |
| `--color-neutral-600` | `#767C90` | Tertiary text, icon default |
| `--color-neutral-700` | `#9CA1B5` | Secondary text |
| `--color-neutral-800` | `#C7CAD9` | Primary text (non-heading) |
| `--color-neutral-900` | `#F1F2F7` | Heading / highest-emphasis text |

## Neutral Scale — Light Theme (secondary)

Light theme never uses pure white (`#FFFFFF`) as the app background, for the
same reduced-eye-strain and edge-softening reasons.

| Token | Hex | Usage |
|---|---|---|
| `--color-neutral-0-light` | `#F5F6FA` | App background (`--zdz-brand-paper`) |
| `--color-neutral-50-light` | `#FFFFFF` | Card / panel surface (pure white permitted only for raised cards atop the off-white background) |
| `--color-neutral-100-light` | `#ECEEF4` | Sunken/inset surfaces |
| `--color-neutral-200-light` | `#E1E4ED` | Table row hover, secondary surface |
| `--color-neutral-300-light` | `#CBCFDC` | Border (subtle) |
| `--color-neutral-400-light` | `#AEB3C4` | Border (default), input border |
| `--color-neutral-500-light` | `#8A8FA3` | Disabled text, placeholder |
| `--color-neutral-600-light` | `#666B80` | Tertiary text, icon default |
| `--color-neutral-700-light` | `#4A4E61` | Secondary text |
| `--color-neutral-800-light` | `#2B2E3D` | Primary text |
| `--color-neutral-900-light` | `#12141C` | Heading / highest-emphasis text |

## Surface Elevation (Dark Theme)

Zodize uses a five-level flat elevation model — no drop shadows create
elevation in dark theme (a subtle 1px border does); shadows are reserved for
true overlays (modals, popovers) per [Motion & Animation](./motion-animation.md)
and [Dark Theme](./dark-theme.md).

| Token | Hex | Level | Usage |
|---|---|---|---|
| `--color-surface-0` | `#0B0D12` | Base | App background, page canvas |
| `--color-surface-1` | `#12141C` | +1 | Sidebar, top nav bar, sunken wells |
| `--color-surface-2` | `#1A1D27` | +2 | Cards, panels, table containers |
| `--color-surface-3` | `#242733` | +3 | Dropdowns, popovers, nested cards |
| `--color-surface-4` | `#2E323F` | +4 | Modals, dialogs, the highest-elevation overlay |

Each step up MUST increase lightness — never introduce elevation via shadow
alone in dark theme, and never skip a level (a modal opened from a card
goes from `surface-2` to `surface-4`, passing conceptually through `surface-3`
for any intermediate popover it might contain).

## Primary / Accent

| Token | Hex | Usage |
|---|---|---|
| `--color-accent-default` | Product-specific (defaults to `#6366F1`, see [Brand Standards](./brand-standards.md)) | Primary buttons, active nav item, focus rings, links, selected states |
| `--color-accent-hover` | Product accent, +8% lightness | Hover state |
| `--color-accent-active` | Product accent, −8% lightness | Pressed state |
| `--color-accent-subtle` | Product accent at 12% opacity over `--color-surface-2` | Selected row background, active tab underline background |
| `--color-accent-text` | Product accent, lightness-adjusted to meet 4.5:1 on `--color-surface-0`/`-1`/`-2` | Accent-colored text/links (never the raw accent hex, to preserve contrast) |

## Semantic Colors

Semantic colors are **fixed across every product** — they never rotate with
the product accent, so "danger" reads as danger in ZodiBank exactly as it
does in ZodiHotel.

| State | Token | Dark theme hex | Light theme hex | Usage |
|---|---|---|---|---|
| Success | `--color-success` | `#22C55E` | `#16A34A` | Confirmations, positive deltas, completed states |
| Success (subtle bg) | `--color-success-subtle` | `#14532D` (16% mix) | `#DCFCE7` | Success badge/toast background |
| Warning | `--color-warning` | `#F59E0B` | `#D97706` | Caution states, pending review, expiring items |
| Warning (subtle bg) | `--color-warning-subtle` | `#78350F` (16% mix) | `#FEF3C7` | Warning badge/toast background |
| Danger | `--color-danger` | `#EF4444` | `#DC2626` | Destructive actions, errors, negative deltas |
| Danger (subtle bg) | `--color-danger-subtle` | `#7F1D1D` (16% mix) | `#FEE2E2` | Error badge/toast background |
| Info | `--color-info` | `#3B82F6` | `#2563EB` | Informational banners, neutral system messages |
| Info (subtle bg) | `--color-info-subtle` | `#1E3A8A` (16% mix) | `#DBEAFE` | Info badge/toast background |

Every semantic foreground color listed above meets 4.5:1 contrast against
`--color-surface-0` through `--color-surface-2` in dark theme, and against
`--color-neutral-0-light`/`-50-light` in light theme. Semantic colors used
as plain text on a subtle background MUST use the base semantic token (not
a lighter/darker manual adjustment) to preserve this guarantee.

## Text Tokens

| Token | Dark theme | Light theme | Usage |
|---|---|---|---|
| `--color-text-primary` | `--color-neutral-900` | `--color-neutral-900-light` | Headings, high-emphasis body text |
| `--color-text-secondary` | `--color-neutral-800` | `--color-neutral-800-light` | Default body text |
| `--color-text-tertiary` | `--color-neutral-700` | `--color-neutral-700-light` | Captions, table headers, helper text |
| `--color-text-disabled` | `--color-neutral-500` | `--color-neutral-500-light` | Disabled control text |
| `--color-text-inverse` | `--color-neutral-0-light` | `--color-neutral-0` | Text on filled accent/semantic backgrounds |

## Border Tokens

| Token | Dark theme | Light theme | Usage |
|---|---|---|---|
| `--color-border-subtle` | `--color-neutral-300` | `--color-neutral-300-light` | Dividers, table row separators |
| `--color-border-default` | `--color-neutral-400` | `--color-neutral-400-light` | Input borders, card outlines |
| `--color-border-focus` | `--color-accent-default` | `--color-accent-default` | Focus ring (see [Accessibility](./accessibility.md) for focus ring spec) |

## Chart / Data Visualization Colors

Chart series colors are a fixed, colorblind-safe categorical sequence, not
derived from the product accent, so multi-series charts remain distinguishable
regardless of which product they render in:

| Series | Hex |
|---|---|
| 1 | `#6366F1` |
| 2 | `#22C55E` |
| 3 | `#F59E0B` |
| 4 | `#EC4899` |
| 5 | `#06B6D4` |
| 6 | `#A855F7` |
| 7 | `#84CC16` |
| 8 | `#F97316` |

Dark-theme-specific chart adaptation (gridlines, axis colors, tooltip
surfaces) is documented in [Dark Theme](./dark-theme.md).

## Token Naming Convention

All tokens use the `--color-*` prefix inside this document for readability;
the fully-qualified, implementation-facing name carries the `--zdz-` product
prefix (e.g. `--zdz-color-surface-2`). See
[Design Tokens](./design-tokens.md) for the complete naming convention and
the CSS-variable/Tailwind mapping used in the Laravel+Vue stack.

# Dark Theme

Dark theme is the **default and primary theme for every Zodize product**.
Every screen MUST be designed dark-first; light theme (documented as a
fully supported secondary mode below) is derived from the dark design, not
the other way around. This is a deliberate brand decision (see
[Brand Standards](./brand-standards.md)): Zodize products are used in
professional, often low-light or multi-monitor operational environments
(trading desks, NOCs, back offices), and a dark, low-chroma surface reduces
eye strain during extended use while reinforcing the "precision instrument"
brand character.

## Default Behavior

- New users see dark theme on first login with no theme selection prompt.
- A theme toggle MUST be available in account/workspace settings (never
  buried more than one navigation level deep), switching immediately without
  a page reload, and persisting per-user via a stored preference.
- The toggle sets `data-theme="dark"` or `data-theme="light"` on the root
  `<html>` element; the token blocks defined in
  [Color System](./color-system.md) key off this attribute, per the
  mapping mechanism in [Design Tokens](./design-tokens.md).
- If no stored preference exists, the product MUST respect the OS-level
  `prefers-color-scheme` media query for the *initial* render, then fall
  back to dark if the OS signal is unavailable — but the explicit in-app
  toggle always takes precedence once set.

## Elevation & Surface System

Dark theme uses lightness, not shadow, to communicate elevation (full token
values in [Color System](./color-system.md)):

| Level | Token | Hex | Distance from base |
|---|---|---|---|
| Base | `--color-surface-0` | `#0B0D12` | — |
| +1 | `--color-surface-1` | `#12141C` | Sidebar, top bar |
| +2 | `--color-surface-2` | `#1A1D27` | Cards, panels |
| +3 | `--color-surface-3` | `#242733` | Dropdowns, popovers |
| +4 | `--color-surface-4` | `#2E323F` | Modals, highest overlay |

Shadows (`--zdz-shadow-sm/md/lg`, see [Design Tokens](./design-tokens.md))
are layered on top of the elevation step for true floating overlays
(popovers, modals) to separate them from the content behind, but are never
the *only* signal of elevation — the lightness step is the primary cue and
must be present even where a shadow is also used.

## Never Pure Black, Never Pure White

- The dark theme background is `#0B0D12`, not `#000000`. Pure black against
  bright text creates excessive contrast that causes visual vibration and
  fatigue during long sessions, and produces unwanted OLED smearing on
  fast content changes (e.g. live trading tickers).
- The light theme background is `#F5F6FA`, not `#FFFFFF`. Pure white is
  permitted only as a card surface directly atop the off-white background
  (`--color-neutral-50-light`, see [Color System](./color-system.md)),
  never as the outermost app canvas.
- Text on dark surfaces caps at `#F1F2F7` (`--color-neutral-900`), not pure
  white, for the same reduced-vibration reasoning.

## Image & Illustration Adaptation

- Line-style illustrations (see [Icons & Illustrations](./icons-illustrations.md))
  are authored as single-color or duotone SVG using `currentColor` or
  token-referenced fills, so they automatically adapt to either theme
  without a separate dark/light asset.
- Product screenshots, diagrams, or any raster image that was authored for
  a light background MUST be avoided in dark-theme UI; where an external
  image cannot be avoided (e.g. a partner logo in a directory), it is
  placed on an explicit `--color-surface-1` or white "logo well" card
  rather than directly on the dark canvas, to guarantee legibility.
- Uploaded user-supplied content (e.g. a business's own logo in ZodiCommerce
  storefront branding) is exempt from the token system by nature, but the
  UI chrome around it MUST still provide adequate surrounding contrast per
  [Accessibility](./accessibility.md).

## Chart Color Adaptation

Charts use the same categorical series palette in both themes (see
[Color System](./color-system.md)), since those hues were selected to
already meet contrast against both `--color-surface-0/1/2` (dark) and
`--color-neutral-0-light/50-light` (light). Adaptation is required only for
the *supporting* chart chrome:

| Element | Dark theme | Light theme |
|---|---|---|
| Gridlines | `--color-neutral-200` at 60% opacity | `--color-neutral-300-light` at 80% opacity |
| Axis labels | `--color-text-tertiary` | `--color-text-tertiary` (light mapping) |
| Zero-baseline | `--color-neutral-400` | `--color-neutral-400-light` |
| Tooltip surface | `--color-surface-3` + `--zdz-shadow-md` | `--color-neutral-50-light` + `--zdz-shadow-md` |
| Area/bar fill opacity | Series color at 24% fill under a solid stroke | Series color at 16% fill (light backgrounds need less fill opacity to avoid muddying) |

Charts MUST NOT hardcode a light-mode-only "white chart background" card —
the chart canvas is always `--color-surface-2`/`--color-neutral-50-light`
(the standard card surface token for the active theme), never a fixed
white.

## Light Theme as Secondary Mode

Light theme is not a lesser-maintained fallback — it MUST meet the same
WCAG 2.1 AA contrast requirements (see [Accessibility](./accessibility.md))
and implement every component state defined in
[Components](./components.md). The light palette is fully specified in
[Color System](./color-system.md). Product teams MUST QA both themes before
shipping any new screen; a screen that has only been visually verified in
dark theme is not done.

## Testing Requirement

Every new component or screen MUST be verified in both `data-theme="dark"`
and `data-theme="light"` as part of the Definition of Done (see
[`../quality/`](../quality)), including a contrast check against the
minimums in [Accessibility](./accessibility.md) for each theme
independently — colors that pass contrast in one theme do not automatically
pass in the other.

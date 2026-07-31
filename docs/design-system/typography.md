# Typography

Typography is the single most load-bearing element of the Zodize visual
system: with color spent sparingly (see [Color System](./color-system.md)),
hierarchy is carried almost entirely by type. Every Zodize product MUST use
the type system defined here without modification.

## Font Families

| Role | Family | Fallback stack | Source |
|---|---|---|---|
| UI / body / headings | **Inter** | `Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif` | Self-hosted variable font, `font-display: swap` |
| Monospace (code, IDs, data tables with numeric alignment) | **JetBrains Mono** | `"JetBrains Mono", ui-monospace, SFMono-Regular, Menlo, monospace` | Self-hosted variable font |

Inter is chosen for its neutral, high-legibility grotesk forms at both small
UI sizes and large display sizes, and for its tabular-figure OpenType
feature, which MUST be enabled (`font-variant-numeric: tabular-nums`) on any
numeric column in a data table or financial figure, so that digits align
vertically. JetBrains Mono is used wherever exact character alignment
matters: code blocks, API keys/tokens, account numbers, transaction IDs,
and monospaced columns in data-dense tables (e.g. ZodiTrade order books).

Fonts MUST be self-hosted (bundled with the application) rather than loaded
from a third-party CDN, for both performance and data-residency reasons
relevant to regulated Zodize customers.

## Type Scale

The scale uses a 1.25 (major third-adjacent) ratio for display and heading
sizes, tightening at body sizes for density. All sizes are defined in `rem`
with a `1rem = 16px` base; px values are given for reference only.

| Token | Size (rem / px) | Line height | Letter spacing | Weight | Usage |
|---|---|---|---|---|---|
| `--zdz-text-display` | 3rem / 48px | 1.1 (52.8px) | -0.02em | 700 | Marketing hero headlines only |
| `--zdz-text-h1` | 2.25rem / 36px | 1.15 (41.4px) | -0.015em | 700 | Page-level title, one per screen |
| `--zdz-text-h2` | 1.75rem / 28px | 1.2 (33.6px) | -0.01em | 600 | Section heading |
| `--zdz-text-h3` | 1.375rem / 22px | 1.25 (27.5px) | -0.005em | 600 | Subsection heading, card group title |
| `--zdz-text-h4` | 1.125rem / 18px | 1.3 (23.4px) | 0 | 600 | Card title, modal title |
| `--zdz-text-h5` | 1rem / 16px | 1.4 (22.4px) | 0 | 600 | Inline section label, table group header |
| `--zdz-text-h6` | 0.875rem / 14px | 1.4 (19.6px) | 0.01em | 600 | Smallest heading; used for dense dashboard widget titles |
| `--zdz-text-body` | 1rem / 16px | 1.5 (24px) | 0 | 400 | Default paragraph and form text |
| `--zdz-text-body-sm` | 0.875rem / 14px | 1.5 (21px) | 0 | 400 | Default UI text: table cells, list items, secondary content |
| `--zdz-text-small` | 0.8125rem / 13px | 1.45 (18.85px) | 0.01em | 400 | Helper text, form hints, timestamps |
| `--zdz-text-caption` | 0.75rem / 12px | 1.4 (16.8px) | 0.02em | 500 | Labels, badges, table column headers (uppercase) |
| `--zdz-text-code` | 0.875rem / 14px | 1.6 (22.4px) | 0 | 400 | Inline and block code, monospace data |

Weights used across the scale are restricted to **400 (regular), 500
(medium), 600 (semibold), and 700 (bold)**. No other weight may be used;
Inter's 100–300 weights are visually too light for the density and
information-criticality of Zodize interfaces, and 800/900 are reserved for
marketing display use only, not product UI.

## Usage Rules by Context

### Dashboards and operational UI (the default context)

- Page title uses `--zdz-text-h1`. Only one `h1` per screen.
- Widget/card titles use `--zdz-text-h5` or `--zdz-text-h6` depending on
  card density — dense multi-widget dashboards (e.g. ZodiTrade market
  overview) use `h6`; single-focus cards use `h5`.
- Body copy in operational UI defaults to `--zdz-text-body-sm` (14px), not
  16px. Zodize dashboards prioritize information density over marketing-
  style readability; see [Spacing & Layout](./spacing-layout.md) for the
  corresponding density rules.
- Never use `--zdz-text-display` inside application chrome. It is reserved
  for marketing surfaces.

### Marketing and public-facing pages

- Marketing pages (product landing pages, the Zodize corporate site) MAY use
  `--zdz-text-display` for hero headlines and MAY use the full 1.5 body line
  height (`--zdz-text-body` at 16px, not the compact 14px dashboard body) for
  improved readability in long-form, low-density contexts.
- Marketing headings MAY use tighter, more expressive letter-spacing than
  the values above only for hero-level display type; all other type sizes
  keep the values in the scale table verbatim.

### Data tables

- Column headers use `--zdz-text-caption` in uppercase with
  `letter-spacing: 0.02em`, colored `--color-text-tertiary` (see
  [Color System](./color-system.md)).
- Cell content uses `--zdz-text-body-sm`. Numeric cells MUST use
  `--zdz-text-code` or apply `font-variant-numeric: tabular-nums` on Inter,
  right-aligned.
- Row-level primary identifiers (e.g. a transaction ID, an account number)
  use `--zdz-text-code` regardless of the surrounding row's type.
- Full table interaction and structure standards live in
  [`../standards/table-standards.md`](../standards/table-standards.md); this
  document governs type choice only.

### Forms

- Field labels use `--zdz-text-body-sm` at weight 500.
- Input text uses `--zdz-text-body-sm` at weight 400.
- Helper/validation text uses `--zdz-text-small`.
- Full form structure and validation-state standards live in
  [`../standards/form-standards.md`](../standards/form-standards.md).

## Responsive Scaling Rules

Zodize dashboards and admin surfaces are **desktop-first** (see
[Responsive Standards](./responsive-standards.md) for the full policy), so
the type scale above is the desktop baseline and scales *down*, not up, at
smaller breakpoints:

| Breakpoint | Display | H1 | H2 | Body defaults |
|---|---|---|---|---|
| `xl` / `2xl` (≥1280px) | 3rem | 2.25rem | 1.75rem | As specified above |
| `lg` (1024–1279px) | 2.5rem | 2rem | 1.625rem | Unchanged |
| `md` (768–1023px) | 2.25rem | 1.75rem | 1.5rem | Unchanged |
| `sm` (<768px) | 2rem | 1.5rem | 1.375rem | `--zdz-text-body` drops to `--zdz-text-body-sm` as the default paragraph size |

Marketing pages, which are mobile-first, scale in the opposite direction:
the `sm` values above are the marketing baseline, scaling *up* through the
table as viewport width increases, capping at the `xl`/`2xl` values.
Heading tokens themselves (`--zdz-h1` etc.) never change value — only which
breakpoint's row is applied to a given heading's rendered size changes, via
the responsive type utilities documented in
[Design Tokens](./design-tokens.md).

Body, small, caption, and code sizes do not scale responsively — they stay
fixed at their table value across all breakpoints, since further reduction
below 12–14px harms legibility and violates the minimum text size guidance
in [Accessibility](./accessibility.md).

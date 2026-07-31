# Motion & Animation

## Principles

Motion in Zodize products exists to communicate state change and preserve
spatial continuity — never to entertain. This follows directly from the
brand principles in [Brand Standards](./brand-standards.md): enterprise
software used by professionals for hours a day cannot afford bouncy,
playful, or attention-seeking animation.

1. **Purposeful.** Every animation MUST answer "what is this telling the
   user?" (an element appeared, a state changed, a relationship between two
   views is being shown). If an animation has no informational purpose, it
   is removed.
2. **Fast.** Zodize motion is quick relative to consumer-app norms.
   Professionals notice latency; nothing in the interaction path should
   feel like it is waiting on an animation to finish.
3. **No bounce, no overshoot, no elastic easing.** Playful physics-based
   easing (spring overshoot, bounce) is prohibited in product UI. All
   easing is calm, monotonic deceleration/acceleration (see the easing
   table below). Overshoot/bounce curves are reserved exclusively for
   marketing pages, and even there used sparingly.
4. **Consistent.** The same interaction (e.g. a dropdown opening) animates
   identically in every product, using the same duration and easing token
   — never re-tuned per product.

## Duration & Easing Table

All values are delivered as design tokens (`--zdz-duration-*`,
`--zdz-ease-*`) per the naming convention in
[Design Tokens](./design-tokens.md).

| Category | Token | Duration | Named easing | Easing curve | Usage |
|---|---|---|---|---|---|
| Micro | `--zdz-duration-micro` | 100–150ms | `--zdz-ease-out` | `cubic-bezier(0.4, 0, 0.2, 1)` | Hover/active state changes, checkbox/toggle flip, icon state swap |
| Standard | `--zdz-duration-standard` | 200–300ms | `--zdz-ease-standard` | `cubic-bezier(0.4, 0, 0.2, 1)` | Dropdown open/close, tab switch, tooltip appear, accordion expand |
| Page/overlay | `--zdz-duration-page` | 300–400ms | `--zdz-ease-emphasized` | `cubic-bezier(0.2, 0, 0, 1)` | Modal/drawer open/close, page transition, panel slide-in |
| Exit | `--zdz-duration-exit` | 150–200ms (faster than entrance) | `--zdz-ease-in` | `cubic-bezier(0.4, 0, 1, 1)` | Any dismiss/close animation — exits are always faster than the corresponding entrance |

Rules for applying the table:

- An element's **entrance** uses `ease-out` (fast start, gentle stop) so it
  feels responsive; its **exit** uses `ease-in` (gentle start, fast finish)
  and a shorter duration, so dismissing something never feels slower than
  summoning it.
- Nothing in product UI exceeds 400ms. If a transition needs to communicate
  something more elaborate than 400ms allows, it is very likely
  communicating too much at once — split it into a state change, not an
  animation.
- Loading spinners and skeleton shimmer sweeps use a continuous linear
  loop, not the eased durations above, but MUST still respect the
  reduced-motion rule below.

## What Should Animate

- Opacity and transform (translate, scale) only, for GPU-cheap, jank-free
  motion. Animating `width`, `height`, `top`/`left`, or `box-shadow`
  directly is prohibited except where no transform-based alternative exists
  (e.g. an accordion's height, which SHOULD use a measured-height transition
  or the CSS `grid-template-rows` trick rather than JS-measured `height`
  animation where feasible).
- State transitions that have a clear before/after: opening/closing
  (modals, drawers, dropdowns, tooltips, accordions), appearing/disappearing
  (toasts, empty-state-to-loaded-state, skeleton-to-content), and
  navigation between related views (tab panel switch).
- Data changes that benefit from continuity: a KPI number ticking to a new
  value, a chart re-rendering with new data (a brief 200–300ms transition
  between data states, not a redraw flash).
- Focus and selection indicators (focus ring, selected row/tab underline)
  MAY use the micro duration for a subtle, non-distracting shift.

## What Should Not Animate

- Page load itself — the initial render of a screen MUST NOT stagger-fade
  its contents in ("entrance choreography"). Content appears; only
  *subsequent* state changes animate.
- Hover states on non-interactive elements.
- Any decorative, idle, or ambient animation (looping background motion,
  particle effects, auto-playing illustrations) — prohibited outright in
  product UI per [Icons & Illustrations](./icons-illustrations.md).
- Scale/bounce feedback on button press (a Zodize button changes background
  color on press, per [Components](./components.md); it does not scale
  down).
- Chained/staggered list-item entrance animations for data tables — a table
  of 50 rows does not cascade-fade in; it renders immediately once data is
  available (skeleton loaders handle the wait, per
  [Components](./components.md)).

## Reduced Motion

Every animation defined in this document MUST respect the user's
`prefers-reduced-motion: reduce` operating-system setting, as a hard
accessibility requirement (see [Accessibility](./accessibility.md)):

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.001ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.001ms !important;
    scroll-behavior: auto !important;
  }
}
```

This is implemented once, globally, in the shared component library — no
individual product or component should need to special-case reduced motion
itself. Components that convey information *through* motion alone (rare,
and generally a design defect per the "What Should Animate" section) MUST
provide a non-motion equivalent (e.g. an instant state swap plus a text or
icon change) when reduced motion is active, not merely a faster version of
the same animation.

## Marketing Pages Exception

Marketing pages (out of scope for the application-shell rules above) MAY
use scroll-triggered reveals, parallax, and slightly more expressive easing
(including a restrained overshoot on hero elements) to support storytelling,
per [Brand Standards](./brand-standards.md)'s distinction between marketing
and product contexts. Even on marketing pages: durations stay under 600ms,
`prefers-reduced-motion` MUST still be respected, and no animation may block
or delay access to page content or primary CTAs.

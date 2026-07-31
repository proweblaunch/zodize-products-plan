# Accessibility

**WCAG 2.1 Level AA is the non-negotiable baseline for every Zodize
product, every screen, every component, with no exceptions.** This is a
compliance requirement (many Zodize customers are regulated banks,
healthcare providers, and government agencies with statutory accessibility
obligations) and a product-quality requirement. A screen that fails WCAG
2.1 AA is not production ready, regardless of how complete its functionality
is — accessibility failures are treated as defects with the same severity
as functional bugs, tracked in [`../checklists/`](../checklists).

This document sets requirements that apply across every component and
pattern; component-specific accessibility notes also appear inline in
[Components](./components.md), [Color System](./color-system.md), and
[Motion & Animation](./motion-animation.md) — this document is the baseline
those specifics build on.

## Color Contrast

Restated from [Color System](./color-system.md), because it is
non-negotiable:

- Normal text: **≥ 4.5:1** contrast against its background.
- Large text (≥24px, or ≥19px bold): **≥ 3:1**.
- UI components/graphical objects carrying meaning (borders, meaningful
  icons, focus indicators, chart marks): **≥ 3:1** against adjacent colors.
- Color/hue MUST NOT be the only means of conveying information. Every
  semantic state (success/warning/danger/info) pairs color with an icon
  and/or text label — never a colored dot alone.
- Minimum body text size is 14px (`--zdz-text-body-sm`); nothing below
  `--zdz-text-caption` (12px) is used for content the user must read to
  operate the product, only for supplementary labels.

## Keyboard Navigation

- Every interactive element MUST be reachable and operable via keyboard
  alone — no mouse-only interaction pattern is permitted anywhere in a
  Zodize product.
- Tab order MUST follow visual/logical reading order. Custom `tabindex`
  values greater than 0 are prohibited; use source order and `tabindex="-1"`
  (programmatic focus only) / `tabindex="0"` (standard focus) exclusively.
- Standard key conventions are mandatory and consistent across all
  products: `Tab`/`Shift+Tab` moves focus, `Enter`/`Space` activates
  buttons and toggles, `Escape` closes the topmost overlay (dropdown,
  modal, drawer) without side effects, arrow keys navigate within a
  composite widget (tabs, listbox/select, menu, radio group) per the
  WAI-ARIA Authoring Practices pattern for that widget.
- Focus MUST NOT become trapped anywhere except an intentional focus trap
  inside an open modal/dialog (where it is required — see Focus Management
  below).
- A "skip to main content" link MUST be the first focusable element on
  every authenticated application shell, visually hidden until focused.

## Focus Management

- Every focusable element MUST render a visible focus indicator: a 2px
  `--color-border-focus` outline with 2px offset, using `:focus-visible`
  (not `:focus`) so the ring appears for keyboard navigation but not
  incidental mouse clicks. `outline: none` without a replacement focus
  style is prohibited outright — grep for this pattern is part of code
  review.
- Opening a modal, drawer, or dialog MUST move focus to the first
  focusable element inside it (or the dialog's heading if it has no
  interactive content before the close action), and MUST trap focus within
  it (`Tab` cycles inside, does not escape to the page behind) for the
  duration it is open.
- Closing a modal, drawer, or dialog MUST return focus to the element that
  triggered it.
- Client-side route changes (SPA navigation within the Vue app) MUST move
  focus to the new page's `h1` or a designated landmark, so screen reader
  and keyboard users are not left focused on a now-removed element.
- Asynchronous content updates that are not user-initiated (e.g. a live
  ticker updating) MUST NOT steal focus from whatever the user is currently
  interacting with.

## Screen Reader Requirements

- All non-text content (icons used as controls, images conveying
  information) MUST have an accessible name via `aria-label`,
  `aria-labelledby`, or equivalent alt text; purely decorative icons/images
  MUST be `aria-hidden="true"` so they are not announced redundantly (see
  [Icons & Illustrations](./icons-illustrations.md)).
- Every form input MUST have a programmatically associated `<label>`
  (via `for`/`id` or wrapping) — placeholder text is never a substitute
  for a label. Full form accessibility requirements live in
  [`../standards/form-standards.md`](../standards/form-standards.md).
- Custom interactive components (selects, tabs, tooltips, modals, toasts)
  MUST implement the correct ARIA role and state attributes per the
  WAI-ARIA Authoring Practices pattern for that widget, as specified
  per-component in [Components](./components.md).
- Dynamic, non-focus-stealing updates that the user should be told about
  (form validation errors appearing, a toast notification, an async save
  completing) MUST use an `aria-live` region (`polite` for most cases,
  `assertive` only for errors that block progress) so screen reader users
  are notified without needing to be focused on that element.
- Landmark regions (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`)
  MUST be used correctly and uniquely labeled where more than one of the
  same landmark type exists on a page (e.g. a primary nav and a secondary
  in-page nav both need distinct `aria-label`s).
- Heading structure MUST be hierarchical with no skipped levels (an `h3`
  never appears without a preceding `h2` in that section), so screen reader
  users can navigate by heading outline.

## Form Accessibility

In addition to the general keyboard, focus, and screen reader requirements
above, forms carry these specific requirements (full layout/validation
behavior specified in [`../standards/form-standards.md`](../standards/form-standards.md)):

- Required fields are indicated both visually (see
  [Components](./components.md)) and programmatically (`aria-required="true"`
  or the native `required` attribute).
- Validation errors are associated with their field via
  `aria-describedby` and the field is marked `aria-invalid="true"`; the
  error text itself uses language describing the problem and, where
  applicable, how to fix it (see the voice/tone rules in
  [Brand Standards](./brand-standards.md)).
- On submit failure, focus MUST move to the first invalid field or to an
  error summary at the top of the form, and the error summary (if used)
  MUST be an `aria-live="assertive"` region so it is announced immediately.
- Multi-step forms MUST announce the current step and total steps
  programmatically (not only visually via a progress bar).

## Motion & Reduced Motion

Every animation defined in [Motion & Animation](./motion-animation.md)
MUST respect `prefers-reduced-motion: reduce`, implemented globally. No
component may define an animation that ignores this system-level setting.
Content MUST NOT be revealed exclusively through motion in a way that
depends on the animation completing to be legible.

## Touch Targets

Every interactive element MUST provide a minimum **44×44px** touch/click
target on breakpoints below `md` (per
[Responsive Standards](./responsive-standards.md)), and a minimum
**24×24px** target at all breakpoints per WCAG 2.1 AA's target size
guidance, using padding/hit-area expansion when the visible control is
smaller (e.g. a 16px icon button still resolves a 40×40px hit area, per
[Components](./components.md)).

## Testing Requirements

Accessibility is verified automatically and manually, both required:

- **Automated**: `axe-core` MUST run in CI against every product's UI test
  suite, on every pull request, and a build MUST fail on any new axe
  "serious" or "critical" violation. This is a hard CI gate, not an
  advisory report. The full CI pipeline configuration and gating rules are
  defined in [`../quality/ci-cd-standards.md`](../quality/ci-cd-standards.md).
- **Manual keyboard walkthrough**: every new screen MUST be manually
  operated end-to-end using keyboard only (no mouse) before it is
  considered done, as part of the checklist in
  [`../checklists/`](../checklists).
- **Screen reader spot-check**: any new composite/custom component
  (anything beyond a standard form input) MUST be tested with at least one
  screen reader (VoiceOver or NVDA) before shipping.
- **Contrast audit**: any new color usage outside the tokens defined in
  [Color System](./color-system.md) requires an explicit contrast check
  against both dark and light theme backgrounds before merge — this is why
  arbitrary color values outside the token system are prohibited in the
  first place.

Automated tooling (axe-core) catches roughly a third of real-world WCAG
issues; it is a floor, not a substitute for the manual keyboard and screen
reader checks above. A product that only passes automated scans has not met
the bar defined in this document.

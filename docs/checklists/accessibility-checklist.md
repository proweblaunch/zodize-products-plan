# Accessibility Checklist

This is the pre-release accessibility checklist every Zodize product MUST
pass, targeting WCAG 2.1 Level AA conformance. It is a companion to
[`../design-system/accessibility.md`](../design-system/accessibility.md),
which defines the underlying design and interaction rules; every item below
MUST be verified with real assistive technology, not inferred from markup
alone. This checklist is a mandatory sub-gate of
[`../checklists/production-readiness-checklist.md`](./production-readiness-checklist.md).

## Perceivable

- [ ] All non-decorative images have descriptive `alt` text; decorative
      images are marked `alt=""` or `aria-hidden="true"`.
- [ ] Color is never the sole means of conveying information (e.g. status
      indicators pair color with an icon or text label).
- [ ] Text and interactive-element contrast ratios meet WCAG 2.1 AA minimums
      (4.5:1 for normal text, 3:1 for large text and UI components) per the
      color tokens in [`../design-system/`](../design-system).
- [ ] All video/audio content (if any) has captions or a transcript.
- [ ] Content reflows correctly at 400% browser zoom without loss of
      functionality or horizontal scrolling of the page body.

## Operable

- [ ] Every interactive element is reachable and operable via keyboard
      alone, in a logical tab order.
- [ ] No keyboard trap exists in any modal, dropdown, or widget.
- [ ] Visible focus indicators are present on every focusable element and
      meet the contrast requirement above.
- [ ] Skip-to-content link is present on every page with a persistent
      navigation header.
- [ ] No content flashes more than three times per second.
- [ ] Time limits (e.g. session timeout, MFA challenge countdown per
      [`../templates/authentication-template.md`](../templates/authentication-template.md))
      are either adjustable, extendable, or clearly warned before expiry.
- [ ] Drag-and-drop interactions (e.g. dashboard widget reordering per
      [`../templates/dashboard-template.md`](../templates/dashboard-template.md))
      have a keyboard-operable equivalent.

## Understandable

- [ ] Every page has a descriptive, unique `<title>`.
- [ ] Form fields have programmatically associated labels; placeholder text
      is never used as the sole label.
- [ ] Validation errors are announced to assistive technology, identify the
      specific field, and describe how to fix the error, per the standard
      error format in [`../templates/api-template.md`](../templates/api-template.md)
      rendered accessibly on the client.
- [ ] Navigation and interaction patterns are consistent across the product,
      per [`../design-system/`](../design-system) and
      [`../standards/`](../standards).
- [ ] Language of the page is set programmatically (`lang` attribute); any
      in-page language change is marked.

## Robust

- [ ] All interactive components use correct semantic HTML or, where a
      custom component is necessary, correct ARIA roles/states/properties
      per [`../design-system/`](../design-system) component specifications.
- [ ] All custom components (dropdowns, modals, tabs, comboboxes) implement
      the appropriate WAI-ARIA authoring pattern.
- [ ] Dynamic content updates (toasts, live-updating widgets) use
      `aria-live` regions appropriately.
- [ ] Markup validates without critical accessibility-tree errors from an
      automated scanner (e.g. axe-core) run in CI.

## Verification

- [ ] Automated accessibility scan run in CI on every build, with zero
      unresolved critical/serious findings.
- [ ] Manual screen reader walkthrough completed (at minimum: VoiceOver or
      NVDA) for the top three critical flows in the product, per
      [`../checklists/production-readiness-checklist.md`](./production-readiness-checklist.md).
- [ ] Manual keyboard-only walkthrough completed for the same critical flows.
- [ ] Findings from manual review are resolved or explicitly accepted as a
      documented risk with a remediation timeline, not silently deferred.

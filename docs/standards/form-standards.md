# Form Standards

Forms are where Zodize products capture the data that every other pattern
in this handbook displays. This document defines field layout, validation
timing, error message standards, the multi-step wizard pattern, autosave
behavior, and the company-wide policy on when a submit button becomes
enabled. It applies inside modals/drawers (see
[`modal-standards.md`](./modal-standards.md)) and on dedicated form pages
(see
[`page-layout-standards.md`](./page-layout-standards.md#form-page-template))
alike.

## Field layout

- **Label placement**: labels sit ABOVE their input, left-aligned, never
  inline-left of the input and never as placeholder-only text. Placeholder
  text, when used, is example content ("e.g. jane@acme.com"), never a
  substitute for a label — a field with no visible label is a defect.
- **Column layout**: forms use a single column by default. A two-column
  layout is permitted only for logically paired short fields (e.g.
  City/State, First name/Last name) on `lg`/`xl` modals or form pages with
  the 960px width allowance in
  [`page-layout-standards.md`](./page-layout-standards.md#content-width-rules).
  Long-text fields (textareas, address lines) always span the full form
  width regardless of column layout.
- **Grouping**: related fields are grouped under a section heading with
  8px spacing between fields in a group and 32px spacing between groups,
  per [`../design-system/spacing-layout.md`](../design-system/spacing-layout.md).
- **Field width**: an input's width MUST hint at its expected content length
  (a 2-digit "Unit number" field is not full-width; a "Description"
  textarea is) — this is a scannability cue, not just aesthetics.
- **Help text**: optional, single-line, rendered below the input in muted
  color, used for format hints ("Format: XX-XXXXXXX") — distinct from error
  text (below), which replaces help text when a field is invalid.

## Required-field marking

- Required fields are marked with a red asterisk (`*`) immediately after
  the label text. Optional fields are unmarked BY DEFAULT — but if a form
  mixes mostly-required with a few optional fields, optional fields MUST
  instead be marked with a muted "(optional)" suffix, and the asterisk
  convention dropped, whichever convention results in fewer marks on the
  page. A form MUST pick one convention and apply it consistently to itself
  (never mix "asterisk-for-required" and "(optional)-for-optional" in the
  same form).
- A legend ("* Required") appears once, near the top of the form, only when
  the asterisk convention is used.

## Inline validation

Zodize uses a hybrid validation timing model, applied per field type:

- **On-blur validation**: format and simple presence checks (email format,
  required-field presence, min/max length, numeric range) validate when the
  user leaves the field (`blur` event), not on every keystroke — validating
  on every keystroke while the user is still typing a valid value produces
  premature, distracting errors. EXCEPTION: once a field has been
  validated and shown an error, subsequent keystrokes in that field
  re-validate live (on `input`) so the error clears the moment it's fixed,
  rather than requiring another blur.
- **On-submit validation**: checks that require server round-trips or
  cross-field context (uniqueness checks like "email already in use",
  cross-field checks like "end date must be after start date") validate on
  form submission. Async on-blur checks (e.g. username availability) MAY
  also run on blur with a debounce of 400ms and a small inline spinner next
  to the field while checking, showing a result (checkmark or error) within
  the field's border/icon.
- Submitting a form with existing on-blur errors immediately re-focuses and
  scrolls to the first invalid field rather than re-running all validations
  silently.

## Error message standards

- Error messages render directly below their field (never in a summary
  block at the top only — a top summary MAY additionally exist for forms
  with 10+ fields, but is not a substitute for inline messages), in the
  design system's danger color, with an error icon.
- The invalid field's border also switches to the danger color and gets a
  danger-colored focus ring on next focus, so the error is visible even
  without reading the text.
- Every error message MUST be specific and actionable — state what's wrong
  and, where possible, how to fix it. "Invalid input" and "This field is
  required" (with no field context) are prohibited when a more specific
  message is derivable. Compare:
  - Prohibited: "Invalid value."
  - Required: "Enter a valid email address, like jane@acme.com."
  - Prohibited: "Error."
  - Required: "Account number must be 10 digits. You entered 8."
- Server-side validation errors returned on submit MUST be mapped back to
  their specific field when the API response identifies a field (per the
  API error format in `docs/development/`); only truly form-level errors
  (e.g. "This tenant has reached its account limit") render in a banner at
  the top of the form.

## Multi-step form / wizard pattern

Used for long, sequential data-collection flows (e.g. onboarding a new
tenant, opening a new account) that benefit from being broken into
digestible stages rather than one long scroll.

- A horizontal step indicator (numbered circles connected by a line) sits
  above the form body, showing all steps by name, with the current step
  highlighted, completed steps shown with a checkmark, and future steps
  shown muted.
- Steps are NOT freely jumpable forward — a user may click back to any
  completed step to revisit it, but MUST NOT jump ahead to an unvisited
  step by clicking the indicator; forward progress only happens via each
  step's "Next" action, which validates the current step's fields before
  advancing.
- Each step's field state persists when navigating back and forth within
  the wizard (no data loss moving between steps).
- The final step shows a "Review" summary of all entered data across every
  step, with an "Edit" link next to each section that jumps directly back
  to the relevant step, before the terminal "Submit"/"Create" action.
- Wizards presented in a modal/drawer are capped at `lg` size; wizards with
  more than 4 steps SHOULD be a full form page instead, per the full-page
  threshold in
  [`modal-standards.md`](./modal-standards.md#modal-vs-drawer-vs-full-page).

## Autosave and drafts

- Any form estimated to take longer than 90 seconds to complete (multi-step
  wizards, forms with 8+ fields, long-text/rich-text content) MUST autosave
  a draft, so a closed tab, browser crash, or session timeout does not
  destroy the user's work.
- Autosave triggers on a 3-second debounce after the last field change (not
  on every keystroke) and on every successful step transition in a wizard.
  A small "Saving draft…" / "Draft saved" indicator shows near the form's
  primary action, transitioning states per the save lifecycle.
- Drafts are stored server-side (not only in browser local storage) keyed
  to the user and form context, so a draft is recoverable from a different
  device/browser session. Returning to a form with an existing draft
  prompts "Resume your draft from <relative time>?" with options to resume
  or start fresh (discarding the draft).
- Drafts expire and are purged after 30 days of inactivity; this MUST be
  stated to the user in the draft-resume prompt when a draft is older than
  7 days ("This draft is 12 days old and will expire in 18 days.").

## Submit policy: disabled-until-valid vs. submit-then-validate

**Zodize's standard policy is submit-then-validate, not disabled-until-valid.**
The submit button is always visibly enabled (never grayed out while the
user is still filling the form), and clicking it triggers full validation,
scrolling to and focusing the first error if any exist.

Justification: a permanently disabled submit button gives the user no
signal about WHY they cannot proceed, forcing them to hunt for the invalid
field themselves — this violates Principle 1 (clarity) in
[`ux-principles.md`](./ux-principles.md). Combined with the on-blur inline
validation defined above, users get error feedback progressively as they
work through the form, and a click on submit always produces either success
or a clear, focused list of what remains to fix — never a mysteriously
inert button.

**Exception**: Tier 3 destructive-action confirmation dialogs (per
[`modal-standards.md`](./modal-standards.md#destructive-action-tiers)) use
disabled-until-valid deliberately — the typed-confirmation requirement is a
DELIBERATE friction mechanism, not a data-entry form, and keeping the button
inert until the exact phrase matches is the intended safety behavior, not a
clarity failure (the requirement itself is stated in the dialog text).

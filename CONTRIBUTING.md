# Contributing to the Zodize Engineering Handbook

This repository governs how every Zodize product is built. Changing it changes
behavior across the entire company's future codebase, so it is held to a
higher bar than a normal docs repo.

## What belongs here vs. what doesn't

- **Belongs here**: any standard, convention, or architectural decision that
  should apply beyond a single product, plus each product's own specification.
- **Does not belong here**: product source code, generated API docs, secrets,
  environment files, or anything that isn't documentation/specification.

## Proposing a change to a global standard

1. Identify the file(s) under `docs/architecture/`, `docs/design-system/`,
   `docs/development/`, `docs/security/`, `docs/quality/`, or `docs/standards/`
   that the change affects.
2. If the change is a **breaking change to an existing standard** (e.g.
   changing the primary color token, changing the required PHP version,
   changing the RBAC model), write an ADR in `docs/decisions/` first, using
   `docs/decisions/adr-template.md`. Link the ADR from the standard you are
   changing.
3. If the change is **additive** (a new standard, a clarification, closing a
   documented gap), you may edit the target file directly in your PR.
4. Update `CHANGELOG.md` under `[Unreleased]`.
5. If the change affects a template in `docs/templates/`, note in the PR
   description which existing product specs (if any) now diverge from the
   template and need a follow-up.

## Proposing a change to a product specification

1. Edit `docs/products/<product>/SPEC.md` (and any sibling files in that
   product's directory).
2. A product spec may **not** contradict a global standard. If it needs to,
   that is a signal the global standard is incomplete — fix the global
   standard first, then the product spec.
3. Update the product's status in `PRODUCT_CATALOG.md` if the change moves it
   between Foundation / Deep / Locked.

## Writing standards for this repository

- No placeholders, no "TBD" left in prose. If something is genuinely not yet
  decided, say so explicitly in a `## Open Questions` or `## Roadmap` section
  at the bottom of the document — never mid-document.
- Write in the imperative/declarative voice of a standard ("Services MUST use
  constructor injection"), not a tutorial voice.
- Every document should be usable by an engineer, or an AI coding agent, with
  zero additional context beyond the rest of this repository.
- Cross-reference related documents with relative markdown links rather than
  restating their content.
- Keep one document focused on one concern. If a document grows past the
  point where its table of contents no longer fits on one screen, split it
  and update the index pages that link to it.

## Review bar

A PR against this repository should be reviewed as if it were a PR against
production infrastructure — because every product downstream of it is
production infrastructure. Treat ambiguity as a defect: if a reviewer has to
ask "but what happens when...", the document is not done.

## Definition of Done for a handbook PR

- [ ] The change is internally consistent with every other document it
      touches or references.
- [ ] Cross-references are valid relative links, not prose descriptions of
      where something "should" live.
- [ ] `CHANGELOG.md` updated.
- [ ] No placeholder text, no lorem ipsum, no "coming soon" outside of an
      explicit `## Roadmap` / `## Open Questions` section.
- [ ] If a product spec was touched, `PRODUCT_CATALOG.md` status is accurate.

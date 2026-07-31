# ADR-0001: Record Architecture Decisions and Establish the Repository Layout

## Status

Accepted

## Context

The Zodize Engineering Handbook exists to be the single source of truth for
how roughly twenty enterprise SaaS products are built on a shared Laravel +
Vue stack, on top of a shared platform (ZodiCore) providing identity,
billing, tenancy, notifications, plugins, and permissions, per
[`../../README.md`](../../README.md). Two related problems needed to be
solved before any global standard or product specification could be written
in good faith:

1. **How do we record a decision that changes an existing standard, rather
   than just overwrite it?** A handbook whose standards can change with no
   trace of why creates two failure modes: engineers and AI coding agents
   cannot tell whether a divergence from a standard in an existing product
   is a bug or a deliberate, superseded-but-documented choice; and future
   contributors re-litigate settled decisions because the reasoning behind
   them was never written down. [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md)
   already assumes ADRs exist as a mechanism ("write an ADR in
   `docs/decisions/`... using `docs/decisions/adr-template.md`") without
   this ADR having established the process it assumes.

2. **What is the permanent top-level shape of this repository?** Twenty
   products, a shared platform, and a shared design system generate a large
   volume of documentation. Without a fixed, enterprise-grade directory
   layout established before content is written, every contributor
   independently guesses where architecture decisions, security policy,
   product specs, and reusable scaffolds belong, producing an
   inconsistent, unnavigable handbook exactly at the scale where
   navigability matters most. [`../../ROADMAP.md`](../../ROADMAP.md) Phase 1
   states this layout is `Stable`, but that status needed a decision record
   backing it, per the same self-referential requirement this ADR
   establishes for every future structural change.

## Decision

Zodize adopts Architecture Decision Records for every breaking change to an
existing global standard, using the process and template defined in
[`adr-template.md`](./adr-template.md):

- Any breaking change to an existing standard under `docs/architecture/`,
  `docs/design-system/`, `docs/development/`, `docs/security/`,
  `docs/quality/`, or `docs/standards/` MUST be preceded by an ADR in
  `docs/decisions/`, per [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md).
- ADRs are numbered sequentially (`adr-XXXX-title-slug.md`), are
  append-only once `Accepted` (superseding decisions are recorded as new
  ADRs, not edits to old ones), and follow the four-section structure
  (Context, Decision, Consequences, Alternatives Considered) fixed by
  [`adr-template.md`](./adr-template.md).

Zodize further adopts the following permanent top-level repository layout,
ratifying [`../../ROADMAP.md`](../../ROADMAP.md) Phase 1:

```
docs/
  architecture/   System architecture, multi-tenancy, plugin/marketplace architecture.
  design-system/  Brand, typography, color, spacing, components, motion, dark theme.
  development/    Coding standards, API standards, database standards, git workflow.
  security/       AuthN/AuthZ, RBAC, audit logging, data protection, DR.
  quality/        Definition of Done, Definition of Production Ready, CI/CD, performance.
  standards/      UX patterns: navigation, layout, dashboards, tables, forms, notifications.
  templates/      Reusable product scaffolds every product inherits from.
  checklists/     Go/no-go checklists for security, accessibility, production readiness.
  decisions/      Architecture Decision Records (this directory).
  products/       Full specification for each Zodize product.
  automation/     CI/CD pipeline definitions and automation standards.
```

This layout is permanent. Introducing a new top-level category under `docs/`
(a thirteenth directory) MUST itself be preceded by an ADR, exactly as any
other breaking structural change would be — the layout is not exempt from
the process it establishes.

## Consequences

**Positive:**

- Every future breaking change to a global standard now has a mandatory,
  discoverable paper trail, making it possible for an engineer or AI coding
  agent to distinguish "deliberate, superseded decision" from "undocumented
  drift" when a product's implementation appears to diverge from a current
  standard.
- The repository layout is now unambiguous and citable — every other
  document in this handbook can link to a fixed set of sibling directories
  with confidence those paths will not move.
- [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md)'s existing reference to
  the ADR process is now backed by an actual accepted decision rather than
  an assumption.

**Negative:**

- Every breaking change to a global standard now carries process overhead:
  an ADR must be written, reviewed, and accepted before the standard itself
  can change. This is a deliberate tradeoff — the cost of writing an ADR is
  small relative to the cost of an undocumented breaking change propagating
  silently across twenty products.
- Renumbering is permanently prohibited even for a rejected or quickly
  reversed decision (its ADR moves to `Superseded`, its number is not
  reclaimed), which means the ADR sequence will contain gaps in practical
  relevance over time, though never gaps in numbering.

No existing product spec required migration as a result of this decision,
since it is the founding structural decision of the handbook itself.

## Alternatives Considered

- **No formal decision record, rely on commit messages and PR descriptions.**
  Rejected: commit history is not organized for "why does this standard say
  X," it is organized chronologically by unrelated change, making it
  effectively unsearchable as a decision log at handbook scale, and it is
  not linkable from the standard itself the way an ADR file is.
- **A single running `DECISIONS.md` log instead of one file per decision.**
  Rejected: a single growing file cannot be linked to at stable, specific
  granularity from the standards it affects, violates the
  one-document-one-concern principle in
  [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md), and produces merge
  conflicts across concurrent PRs touching unrelated decisions.
- **A flatter repository layout (e.g. all standards in one `docs/standards/`
  directory, no separate `security/`, `quality/`, `architecture/`
  directories).** Rejected: a shared platform serving twenty products across
  domains as varied as banking and healthcare requires enough depth per
  concern (security alone spans authN/authZ, RBAC, audit logging, data
  protection, and disaster recovery) that a flat layout would force any one
  directory to become unnavigable well before the handbook's content is
  complete.
- **Organize by product instead of by concern at the top level.** Rejected:
  the explicit goal of this handbook is that standards are shared and
  inherited across products, per [`../../README.md`](../../README.md);
  organizing by product first would encourage per-product reinvention of
  standards that are supposed to be common, which is the exact failure mode
  the templates in `docs/templates/` exist to prevent.

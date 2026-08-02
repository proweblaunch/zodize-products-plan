# Zodize Engineering Handbook

> The single source of truth for every Zodize product, present and future.

This repository is not a wiki page and not a collection of notes. It is the **operating
system of the Zodize engineering department** — the complete set of standards,
architecture decisions, design language, and product blueprints that every Zodize
product must inherit and comply with.

If a decision about how Zodize builds software is not written down here, it is not
yet a Zodize standard. Engineers, designers, and AI coding agents (including Claude
Code) should be able to build any Zodize product to production quality using only
what is documented in this repository, the linked design system at
`/home/zodize/public_html`, and the product-specific specification in
`docs/products/<product>/`.

## How this repository is organized

| Path | Purpose |
|---|---|
| [`ROADMAP.md`](./ROADMAP.md) | Sequencing of handbook phases and the product build order. |
| [`PRODUCT_CATALOG.md`](./PRODUCT_CATALOG.md) | Every Zodize product, one-line pitch, market, and status. |
| [`CHANGELOG.md`](./CHANGELOG.md) | Notable changes to the handbook itself (not to any product). |
| [`CONTRIBUTING.md`](./CONTRIBUTING.md) | How to propose a change to a standard. |
| [`docs/architecture/`](./docs/architecture) | System architecture, the single-tenant deployment model, base codebase strategy, plugin/marketplace architecture. |
| [`docs/design-system/`](./docs/design-system) | Brand, typography, color, spacing, components, motion, dark theme. |
| [`docs/development/`](./docs/development) | Coding standards, API standards, database standards, git workflow. |
| [`docs/security/`](./docs/security) | AuthN/AuthZ, RBAC, audit logging, data protection, DR. |
| [`docs/quality/`](./docs/quality) | Definition of Done, Definition of Production Ready, CI/CD, performance. |
| [`docs/standards/`](./docs/standards) | UX patterns: navigation, layout, dashboards, tables, forms, notifications. |
| [`docs/templates/`](./docs/templates) | Reusable product scaffolds every product inherits from. |
| [`docs/checklists/`](./docs/checklists) | Go/no-go checklists for security, accessibility, production readiness. |
| [`docs/decisions/`](./docs/decisions) | Architecture Decision Records (ADRs). |
| [`docs/products/`](./docs/products) | Full specification for each Zodize product. |
| [`docs/automation/`](./docs/automation) | CI/CD pipeline definitions and automation standards. |

## Reading order for a new engineer (human or AI)

1. `docs/development/engineering-principles.md` — how Zodize thinks about software.
2. `docs/design-system/` — the visual and interaction language, in full.
3. `docs/development/` — the technical standards (Laravel, Blade/Bootstrap, API, DB, testing, git).
4. `docs/security/` and `docs/quality/` — the non-negotiables.
5. `docs/templates/` — the scaffolds every product starts from.
6. `docs/products/<product>/SPEC.md` — the product you are actually building.

## Golden rule

**No implementation assumption is left undocumented.** If you find yourself
guessing how something should work, the gap belongs in this handbook, not in your
head and not in a Slack thread. Open a PR against the relevant `docs/` file first.

## Status

This handbook is under active construction. See `ROADMAP.md` for what is complete,
what is in progress, and what is queued next. Documents in this repository are
production-quality as written — nothing here is a stub — but the handbook's
*coverage* expands over time following the roadmap.

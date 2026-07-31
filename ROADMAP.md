# Zodize Handbook & Product Roadmap

This roadmap tracks two things that must never be conflated: the maturity of the
**handbook** (the standards) and the maturity of the **products** (the software
built from those standards). Products do not start until the handbook sections
they depend on are marked `Stable`.

## Phase status legend

- `Stable` — ratified, in force, breaking changes require an ADR.
- `Drafted` — written to production quality, open for review/refinement.
- `Queued` — scoped in this roadmap, not yet written.

## Phase 1 — Repository Structure

**Status: Stable**

The enterprise directory layout (`docs/architecture`, `docs/design-system`,
`docs/development`, `docs/security`, `docs/products`, `docs/templates`,
`docs/decisions`, `docs/standards`, `docs/checklists`, `docs/automation`,
`docs/quality`) is established and is the permanent top-level shape of this
repository. New categories require an ADR (see `docs/decisions/`).

## Phase 2 — Global Engineering Handbook

**Status: Drafted, expanding**

| Area | Status |
|---|---|
| Engineering Principles, Product Philosophy | Drafted |
| Design System (brand, typography, color, spacing, tokens, components, icons, motion, dark theme, responsive, accessibility) | Drafted |
| UX Standards (navigation, layout, dashboards, tables, forms, modals, notifications, email/SMS) | Drafted |
| Coding Standards (Laravel, Vue), API/REST/Webhook/SDK Standards | Drafted |
| Database, Migration, Seeder, Repository Pattern | Drafted |
| Security, AuthN/AuthZ, RBAC, Audit Logs, Data Protection, DR | Drafted |
| Multi-tenancy, Plugin & Marketplace Architecture | Drafted |
| CI/CD, Testing, Performance, Monitoring, Definition of Done/Production Ready | Drafted |
| Git Workflow, PR Standards, Code Review, AI Coding Standards | Drafted |
| Localization, Feature Flags, Environment & Release Standards | Drafted |
| Deep sub-specs (per-component design specs, full API schema style guide with OpenAPI examples, exhaustive chart/report catalog) | Queued — expand incrementally as products surface real requirements |

## Phase 3 — Product Templates

**Status: Drafted**

All twelve templates listed in the mission brief exist in `docs/templates/` as
implementation-ready specifications: Marketing Website, Documentation,
Authentication, Dashboard, Admin, Database, Module, API, Permission, Testing,
Deployment, Release. Every future product inherits these by reference rather
than copy-pasting — a product spec says "uses the Dashboard Template" plus its
deltas, not a re-derivation.

## Phase 4 — Product Specifications

**Status: In progress — ZodiCore complete as reference; remaining 19 drafted at
foundation depth, queued for deep expansion**

`ZodiCore` is the reference-depth product specification: every section in
"Product Specification Requirements" (vision through production checklist) is
written out in full, because ZodiCore is the platform every other product is
built on top of (identity, billing, tenancy, notifications, audit, plugin
runtime). Every other product's spec currently covers vision, personas, market,
architecture, modules, core data model, key workflows, integrations, and
acceptance criteria in full — with database ER detail, exhaustive endpoint
listings, and full report/chart catalogs queued as the next expansion pass,
tracked per-product in each `docs/products/<product>/SPEC.md#roadmap` section.

### Product build order (dependency-driven, not alphabetical)

1. **ZodiCore** — identity, tenancy, billing, notifications, plugin runtime. Every other product depends on this.
2. **ZodiBusiness** — general SMB ERP (CRM, inventory, invoicing) — proves the module system.
3. **ZodiCommerce** — storefront + order management — proves the marketplace/plugin system under load.
4. **ZodiPOS** — point of sale — proves offline-first and hardware integration patterns.
5. **ZodiTrack** — logistics/fleet-adjacent inventory tracking.
6. **ZodiFleet** — fleet management.
7. **ZodiEstate** — real estate management.
8. **ZodiHotel** — hospitality/PMS.
9. **ZodiMed** — healthcare/clinic management (highest compliance bar — HIPAA-equivalent).
10. **ZodiCampus** — education/campus management.
11. **ZodiLaw** — legal practice management.
12. **ZodiBuild** — construction/project management.
13. **ZodiAgro** — agriculture management.
14. **ZodiGov** — government/public sector.
15. **ZodiBank** — core banking (highest security bar).
16. **ZodiTrade** — brokerage/trading.
17. **ZodiXchange** — exchange infrastructure.
18. **ZodiCapital** — investment/fund management.
19. **ZodiYield** — yield/lending products.
20. **ZodiReach** — marketing/outreach/CRM-adjacent.

Financial products (15–19) are sequenced last because they inherit the deepest
security, audit, and compliance requirements from ZodiBank's spec, which itself
extends every standard in `docs/security/`.

## Non-goals for this phase

- No product source code, no Laravel projects, no scaffolding commands are run
  from this repository. This repository is documentation and specification only.
- No product begins implementation until its spec passes the
  `docs/checklists/production-readiness-checklist.md` "spec complete" gate.

## How to propose a roadmap change

Open a PR modifying this file plus the relevant ADR in `docs/decisions/`. See
`CONTRIBUTING.md`.

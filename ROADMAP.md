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
| Base Codebase Strategy, Single-Tenant Deployment Model, Frontend–Backend Bridge, Product Genericization Checklist | Drafted |
| Payment Gateways, Wallet/Ledger System, Admin Configuration Baseline | Drafted |
| Plugin & Marketplace Architecture (per-product extension marketplace, not a cross-product platform) | Drafted |
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

**Status: Drafted — every product's spec revised to the standalone,
self-hosted architecture (see `docs/architecture/overview.md`); all 20
complete at foundation depth, queued for deep expansion**

Every product's spec covers vision, personas, market, architecture (how it
is built from the sanitized base codebase plus its own domain modules —
see `docs/architecture/base-codebase-strategy.md`), modules, core data
model, key workflows, integrations, and acceptance criteria in full — with
database ER detail, exhaustive endpoint listings, and full report/chart
catalogs queued as the next expansion pass, tracked per-product in each
`docs/products/<product>/SPEC.md#roadmap` section. No product depends on
another product or on a shared platform — each is fully standalone, per
`docs/architecture/single-tenant-deployment-model.md`.

### Product build order (complexity/risk-driven, not dependency-driven)

There is no cross-product dependency to sequence around — every product is
independent. The build order below exists purely to manage complexity and
risk, and its first entry is deliberately chosen to validate the shared
**build pipeline** every subsequent product reuses: clone the sanitized base
codebase → run the
[genericization checklist](./docs/architecture/product-genericization-checklist.md) →
wire the [frontend–backend bridge](./docs/architecture/frontend-backend-bridge.md) →
layer on the product's own domain modules. Getting this pipeline right once,
on the simplest product, is far cheaper than discovering a pipeline defect
on the fourth or fifth product.

1. **ZodiBusiness** — general SMB ERP (CRM, inventory, invoicing). Closest
   fit to the base codebase's existing shape (users, wallet, plans), so it
   validates the clone → genericize → bridge → extend pipeline with the
   least new domain surface area, and is the reference example other
   products follow for their own genericization pass.
2. **ZodiCommerce** — storefront + order management — proves the plugin/
   marketplace extension system (per-product, not cross-product) under a
   catalog/inventory-heavy domain.
3. **ZodiPOS** — point of sale — proves offline-first and hardware
   integration patterns on top of the same base.
4. **ZodiTrack** — logistics/fleet-adjacent inventory tracking.
5. **ZodiFleet** — fleet management.
6. **ZodiEstate** — real estate management.
7. **ZodiHotel** — hospitality/PMS.
8. **ZodiReach** — marketing/outreach/CRM-adjacent.
9. **ZodiCore** — general-purpose ERP/back-office starter product (no longer
   a shared platform — just the product whose domain surface is closest to
   the base codebase as-is).
10. **ZodiMed** — healthcare/clinic management (highest compliance bar —
    HIPAA-equivalent).
11. **ZodiCampus** — education/campus management.
12. **ZodiLaw** — legal practice management.
13. **ZodiBuild** — construction/project management.
14. **ZodiAgro** — agriculture management.
15. **ZodiGov** — government/public sector.
16. **ZodiBank** — core banking (highest security bar; re-adds the
    loan/DPS/FDR domain modules the genericization checklist strips from
    every other product).
17. **ZodiTrade** — brokerage/trading.
18. **ZodiXchange** — exchange infrastructure.
19. **ZodiCapital** — investment/fund management.
20. **ZodiYield** — yield/lending products.

Financial products (16–20) are sequenced last because they carry the deepest
security, audit, and compliance requirements, and because several of them
(ZodiXchange, ZodiTrade, ZodiBank) require the multi-currency wallet
extension documented in `docs/decisions/0002-single-currency-wallet-by-default.md` —
best tackled once the base pipeline is well-proven on simpler products.

## Non-goals for this phase

- No product source code, no Laravel projects, no scaffolding commands are run
  from this repository. This repository is documentation and specification only.
- No product begins implementation until its spec passes the
  `docs/checklists/production-readiness-checklist.md` "spec complete" gate.

## How to propose a roadmap change

Open a PR modifying this file plus the relevant ADR in `docs/decisions/`. See
`CONTRIBUTING.md`.

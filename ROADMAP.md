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

**This Foundation depth is sufficient to begin implementation.** Full ER
diagrams and an exhaustive endpoint catalog (the depth ZodiCore alone
currently has) are not a prerequisite for starting a product's build — they
are a **GA gate** requirement (before that product is sold to real buyers),
per `docs/checklists/production-readiness-checklist.md`. Deep artifacts are
written just-in-time, one module at a time, as that module is actually
implemented — not as a blanket prerequisite that blocks every other
product from starting until ZodiCore-level depth is reached everywhere.

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

> **ZodiTrack is excluded from this build-from-scratch queue.** It already
> exists as a complete, working, currently-resold product — see
> `PRODUCT_CATALOG.md`'s `Live — Extend Only` status and `BUILD_STATE.md`
> for its audit-driven gap-fill queue. Nothing below applies to it: it is
> never cloned, re-scaffolded, or built fresh.

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
4. **ZodiFleet** — fleet management.
5. **ZodiEstate** — real estate management.
6. **ZodiHotel** — hospitality/PMS.
7. **ZodiReach** — marketing/outreach/CRM-adjacent.
8. **ZodiCore** — general-purpose ERP/back-office starter product (no longer
   a shared platform — just the product whose domain surface is closest to
   the base codebase as-is).
9. **ZodiMed** — healthcare/clinic management (highest compliance bar —
    HIPAA-equivalent).
10. **ZodiCampus** — education/campus management.
11. **ZodiLaw** — legal practice management.
12. **ZodiBuild** — construction/project management.
13. **ZodiAgro** — agriculture management.
14. **ZodiGov** — government/public sector.
15. **ZodiBank** — core banking (highest security bar; re-adds the
    loan/DPS/FDR domain modules the genericization checklist strips from
    every other product).
16. **ZodiTrade** — brokerage/trading.
17. **ZodiXchange** — exchange infrastructure.
18. **ZodiCapital** — investment/fund management.
19. **ZodiYield** — yield/lending products.

Financial products (15–19) are sequenced last because they carry the deepest
security, audit, and compliance requirements, and because several of them
(ZodiXchange, ZodiTrade, ZodiBank) require the multi-currency wallet
extension documented in `docs/decisions/0002-single-currency-wallet-by-default.md` —
best tackled once the base pipeline is well-proven on simpler products.

## Non-goals for this phase

- This repository stays documentation and specification only — no product
  source code, no Laravel projects, and no scaffolding commands are ever
  run *from* this repository. It is, however, the authoritative blueprint
  Claude Code (or any engineer) executes *against* when building each
  product's actual codebase at the working-directory convention in
  `docs/architecture/deployment-paths.md`
  (`/home/script/public_html/<product-slug>/`), tracked via
  `BUILD_STATE.md`. The repository documents; the build happens elsewhere,
  guided entirely by what's documented here.
- No product begins implementation until its spec passes the "Spec
  Complete" gate in `docs/checklists/production-readiness-checklist.md` —
  satisfiable at **Foundation** depth (see Phase 4 below); full ER
  diagrams and exhaustive endpoint catalogs are a **GA gate** requirement,
  not an implementation-start requirement.

## How to propose a roadmap change

Open a PR modifying this file plus the relevant ADR in `docs/decisions/`. See
`CONTRIBUTING.md`.

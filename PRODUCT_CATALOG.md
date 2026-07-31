# Zodize Product Catalog

Every Zodize product, its one-line pitch, primary market, and spec status.
Full specification lives at `docs/products/<name>/SPEC.md`. Every product is
sold as **standalone, self-hosted source code** — see
[`docs/architecture/overview.md`](./docs/architecture/overview.md) — not as
a hosted service, and each is fully independent (no product depends on
another being deployed or reachable).

| Product | Pitch | Primary Industry | Spec Status |
|---|---|---|---|
| [ZodiCore](./docs/products/ZodiCore/SPEC.md) | General-purpose back-office/ERP starter product | ERP / General Business | Foundation |
| [ZodiBank](./docs/products/ZodiBank/SPEC.md) | Core banking platform for digital-first banks and credit unions | Banking | Foundation |
| [ZodiTrade](./docs/products/ZodiTrade/SPEC.md) | Multi-asset brokerage and trading platform | Capital Markets | Foundation |
| [ZodiXchange](./docs/products/ZodiXchange/SPEC.md) | Exchange infrastructure for spot and derivatives markets | Capital Markets | Foundation |
| [ZodiEstate](./docs/products/ZodiEstate/SPEC.md) | Property management and real estate operations platform | Real Estate | Foundation |
| [ZodiMed](./docs/products/ZodiMed/SPEC.md) | Clinic and hospital management with patient records | Healthcare | Foundation |
| [ZodiCampus](./docs/products/ZodiCampus/SPEC.md) | Campus and student information system | Education | Foundation |
| [ZodiCommerce](./docs/products/ZodiCommerce/SPEC.md) | Enterprise storefront and order management | Retail / E-commerce | Foundation |
| [ZodiBusiness](./docs/products/ZodiBusiness/SPEC.md) | SMB ERP: CRM, inventory, invoicing, accounting | SMB / General | Foundation |
| [ZodiTrack](./docs/products/ZodiTrack/SPEC.md) | Asset and inventory tracking across locations | Logistics | Live — Extend Only |
| [ZodiCapital](./docs/products/ZodiCapital/SPEC.md) | Fund and investment portfolio management | Asset Management | Foundation |
| [ZodiYield](./docs/products/ZodiYield/SPEC.md) | Lending, credit, and yield product management | Fintech Lending | Foundation |
| [ZodiReach](./docs/products/ZodiReach/SPEC.md) | Marketing automation and omnichannel outreach | Marketing | Foundation |
| [ZodiPOS](./docs/products/ZodiPOS/SPEC.md) | Point of sale for retail and hospitality | Retail | Foundation |
| [ZodiLaw](./docs/products/ZodiLaw/SPEC.md) | Legal practice and matter management | Legal | Foundation |
| [ZodiHotel](./docs/products/ZodiHotel/SPEC.md) | Property management system for hotels | Hospitality | Foundation |
| [ZodiAgro](./docs/products/ZodiAgro/SPEC.md) | Farm and agribusiness operations management | Agriculture | Foundation |
| [ZodiFleet](./docs/products/ZodiFleet/SPEC.md) | Fleet management and vehicle telematics | Logistics / Transport | Foundation |
| [ZodiBuild](./docs/products/ZodiBuild/SPEC.md) | Construction project and site management | Construction | Foundation |
| [ZodiGov](./docs/products/ZodiGov/SPEC.md) | Government service delivery and citizen records | Public Sector | Foundation |

## Spec status definitions

- **Reference-depth**: every section of the product specification requirements
  (vision through production checklist) is written to full implementation
  detail, including complete ER diagrams and endpoint catalogs.
- **Foundation**: vision, market, personas, architecture (how the product is
  built from the sanitized base codebase per
  [`docs/architecture/base-codebase-strategy.md`](./docs/architecture/base-codebase-strategy.md)
  plus its own domain modules), modules, core data model, key workflows,
  integrations, permissions model, and acceptance criteria are complete and
  implementation-usable. Deep artifacts (full ER diagrams, exhaustive
  endpoint listings, full report catalogs) are queued and tracked in the
  product's own roadmap section.
- **Deep**: reference-depth detail plus full ER diagrams and endpoint
  catalogs, without yet being frozen for a release.
- **Locked**: reviewed and frozen for a release; changes require an ADR.
- **Live — Extend Only**: the product already exists as a complete, working
  codebase currently being resold to real buyers, independent of this
  handbook's build pipeline. It is never cloned, re-scaffolded, or built
  from the sanitized base codebase, and never subject to destructive setup
  (fresh migrations, dependency reinstall-from-scratch). The only allowed
  workflow is auditing the live codebase against its `SPEC.md`, producing a
  gap list, and adding purely additive features via normal, rollback-safe
  migrations — see [`BUILD_STATE.md`](./BUILD_STATE.md)'s protocol for
  `Live — Extend Only` products.

## Every product is independent

Every product listed above is its own standalone, single-tenant Laravel
application (see
[`docs/architecture/single-tenant-deployment-model.md`](./docs/architecture/single-tenant-deployment-model.md)),
cloned and genericized from one shared, audited base codebase (see
[`docs/architecture/base-codebase-strategy.md`](./docs/architecture/base-codebase-strategy.md))
and the shared Zodize frontend design system, then extended with its own
domain modules per
[`docs/architecture/product-genericization-checklist.md`](./docs/architecture/product-genericization-checklist.md).
No product depends on another product, or on any Zodize-operated central
service, being deployed or reachable at runtime. `ZodiCore` is not a shared
platform the other nineteen depend on — it is its own sellable product (a
general-purpose ERP/back-office starter), listed here on equal footing with
every other product. `ZodiTrack` is the one exception to the
clone-from-base build pipeline described above — it is already live and
independently built; see its `Live — Extend Only` status.

# Zodize Product Catalog

Every Zodize product, its one-line pitch, primary market, and spec status.
Full specification lives at `docs/products/<name>/SPEC.md`.

| Product | Pitch | Primary Industry | Spec Status |
|---|---|---|---|
| [ZodiCore](./docs/products/ZodiCore/SPEC.md) | The identity, tenancy, billing, and plugin runtime every Zodize product is built on | Platform / Internal | Reference-depth |
| [ZodiBank](./docs/products/ZodiBank/SPEC.md) | Core banking platform for digital-first banks and credit unions | Banking | Foundation |
| [ZodiTrade](./docs/products/ZodiTrade/SPEC.md) | Multi-asset brokerage and trading platform | Capital Markets | Foundation |
| [ZodiXchange](./docs/products/ZodiXchange/SPEC.md) | Exchange infrastructure for spot and derivatives markets | Capital Markets | Foundation |
| [ZodiEstate](./docs/products/ZodiEstate/SPEC.md) | Property management and real estate operations platform | Real Estate | Foundation |
| [ZodiMed](./docs/products/ZodiMed/SPEC.md) | Clinic and hospital management with patient records | Healthcare | Foundation |
| [ZodiCampus](./docs/products/ZodiCampus/SPEC.md) | Campus and student information system | Education | Foundation |
| [ZodiCommerce](./docs/products/ZodiCommerce/SPEC.md) | Enterprise storefront and order management | Retail / E-commerce | Foundation |
| [ZodiBusiness](./docs/products/ZodiBusiness/SPEC.md) | SMB ERP: CRM, inventory, invoicing, accounting | SMB / General | Foundation |
| [ZodiTrack](./docs/products/ZodiTrack/SPEC.md) | Asset and inventory tracking across locations | Logistics | Foundation |
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
  detail, including complete ER diagrams and endpoint catalogs. Used as the
  template other specs are measured against.
- **Foundation**: vision, market, personas, architecture, modules, core data
  model, key workflows, integrations, permissions model, and acceptance
  criteria are complete and implementation-usable. Deep artifacts (full ER
  diagrams, exhaustive endpoint listings, full report catalogs) are queued and
  tracked in the product's own roadmap section.
- **Deep**: matches ZodiCore's level of detail.
- **Locked**: reviewed and frozen for a release; changes require an ADR.

## Cross-cutting platform

Every product listed above is a tenant application running on **ZodiCore**.
None of them re-implement authentication, billing, notifications, or the
plugin runtime — they consume ZodiCore's services and extend them with
domain-specific modules. See `docs/architecture/multi-tenancy.md` and
`docs/architecture/plugin-architecture.md`.

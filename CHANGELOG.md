# Changelog

All notable changes to the **handbook itself** are documented here. This file
does not track product releases — each product tracks its own release notes
under `docs/products/<product>/CHANGELOG.md` once implementation begins.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Changed — architecture correction: standalone self-hosted products, not multi-tenant SaaS

- Corrected the handbook's foundational architecture assumption. Every
  Zodize product is sold as standalone, self-hosted source code to a
  non-technical buyer who deploys it on their own shared/VPS hosting — not
  as a hosted SaaS service. Removed the "ZodiCore as shared platform every
  product is a tenant of" model entirely.
- Replaced `docs/architecture/multi-tenancy.md` with
  `docs/architecture/single-tenant-deployment-model.md`: no `tenant_id`
  scoping, no tenant provisioning lifecycle, no subdomain-per-tenant
  routing — each product is one buyer's one database.
- Rewrote `docs/architecture/overview.md` around the real deployment target
  (shared/VPS hosting, one codebase, no load balancer or CDN assumed by
  default) and the real base-codebase strategy.
- Added `docs/architecture/base-codebase-strategy.md`, documenting the
  audited base codebase (derived from `qfsfountains`'s Laravel 11/PHP 8.3
  application): its inherited admin engine (settings, wallet/ledger,
  payment gateways, withdrawal methods, referrals, plans, KYC, i18n, cron,
  extensions, CMS/page builder), which banking-specific tables must be
  stripped per non-banking product, and the one-time cleanup required
  before the base is clone-ready.
- Added `docs/architecture/frontend-backend-bridge.md`, scoping the real,
  unsolved gap between the static Zodize marketing frontend shell (derived
  from `zodize`'s design-system codebase) and the base codebase's working
  CMS — this bridge does not exist yet and must be built.
- Added `docs/architecture/product-genericization-checklist.md`, the exact
  per-product steps to clone and adapt the sanitized base.
- Added `docs/standards/payment-gateways.md` cataloging the confirmed
  gateway inventory (Authorize.Net, BTCPay, CoinGate, Mollie, Razorpay,
  Stripe, manual/offline) and flagging Flutterwave/Paystack verification as
  an open action item for the African/Nigerian market.
- Added `docs/standards/wallet-system.md` documenting the existing
  `Transaction`/`BalanceTransfer` double-entry ledger engine, and explicitly
  scoping the single-currency-by-default gap via
  `docs/decisions/0002-single-currency-wallet-by-default.md`.
- Added `docs/standards/admin-configuration-baseline.md`, enumerating every
  setting a buyer configures with zero code access.
- Updated `docs/standards/localization-i18n.md` to document the actual
  inherited i18n engine (`LanguageController`, `core/lang/{code}.json`,
  `LanguageMiddleware`) and made multi-language mandatory.
- Rewrote `docs/templates/admin-template.md` around the real
  qfsfountains-derived back office (guards, custom RBAC, ~40 admin
  controllers) instead of a SaaS tenant-management console.
- Updated `ROADMAP.md` and `PRODUCT_CATALOG.md`: removed the ZodiCore-first
  dependency-driven build order; every product is independent, and the
  build order is now complexity/risk-driven, validating the clone →
  genericize → bridge pipeline on the simplest product first.
- Swept every other handbook document referencing `ZodiCore` as an infra
  dependency or `tenant`/multi-tenant scoping and corrected or removed each
  instance, including all 20 product specifications.

## [0.1.0] - 2026-07-31

### Added

- Enterprise repository structure: `docs/architecture`, `docs/design-system`,
  `docs/development`, `docs/security`, `docs/products`, `docs/templates`,
  `docs/decisions`, `docs/standards`, `docs/checklists`, `docs/automation`,
  `docs/quality`.
- Root governance documents: `README.md`, `ROADMAP.md`, `PRODUCT_CATALOG.md`,
  `CONTRIBUTING.md`.
- Full first pass of the Global Engineering Handbook (Phase 2): engineering
  principles, product philosophy, complete design system, UX standards,
  coding/API/database standards, security standards, multi-tenancy and plugin
  architecture, CI/CD and quality gates, git workflow and AI coding standards.
- All twelve Phase 3 product templates.
- Reference-depth specification for **ZodiCore**.
- Foundation-depth specifications for the remaining 19 products.
- Architecture Decision Record process and ADR-0001.

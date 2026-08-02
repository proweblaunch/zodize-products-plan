# Changelog

All notable changes to the **handbook itself** are documented here. This file
does not track product releases — each product tracks its own release notes
under `docs/products/<product>/CHANGELOG.md` once implementation begins.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added — Zodize MCP Server build-VPS access; ZodiCore/ZodiTrack/ZodiBank verified-fact corrections

- Resolved the previously-flagged environment blocker: the Zodize MCP
  Server now provides real filesystem access to the build VPS
  (`/home/script/public_html/`, `/home/qfsfountains/`, `/home/zodize/`,
  `/home/novavest/`, `/home/dash/`, `/home/web3chainlink/`, etc.). Updated
  `BUILD_STATE.md` to record this resolution and to add per-product rows
  for ZodiBank, ZodiCore, ZodiCapital, ZodiYield, and ZodiTrack reflecting
  direct filesystem audit findings, plus a new protocol rule (7) covering
  alternate-base products.
- **Corrected `docs/products/ZodiCore/SPEC.md`'s Architecture, Technology,
  and Modules sections**: a direct audit found ZodiCore's real codebase is
  built on "Ultimate POS," not the sanitized qfsfountains base, with **22
  addon modules already installed and active** (confirmed via
  `modules_statuses.json` and Laravel's own `bootstrap/cache/*_module.php`
  service-provider caches) — correcting the prior task premise that these
  were only staged as zip files awaiting installation. Recorded the 3
  known bugs (cheque due-date field, language-specific spacing, customer/
  supplier creation error), the confirmed-404 public front pages, and
  flagged the reported license/piracy-marker concern as **unconfirmed** in
  this pass (not found by text search, which is not the same as
  disproven) — tracked as a parallel, non-blocking security task in
  `BUILD_STATE.md`'s Flagged Items.
- **Corrected `docs/products/ZodiTrack/SPEC.md`**: a direct audit of the
  live codebase found it is a freight/shipment-tracking and logistics-
  brokerage website (public tracking-number lookup, freight service pages,
  a 33-file admin back office covering shipments/branches/staff/customers/
  vendors/invoices/reports) — **not** the ITAM/enterprise-asset-tracking
  tool the existing spec (§1–§7) describes. Added a prominent correction
  notice (new §0) rather than fabricating a full rewrite without a deeper
  audit; flagged for a follow-up session to complete. Updated
  `PRODUCT_CATALOG.md`'s ZodiTrack pitch to reflect this.
- Added `docs/standards/frontend-standard.md`: the shared-frontend-shell
  rule (one design system, twenty-plus products), grounded in a direct
  audit of `/home/zodize/public_html`'s actual component library —
  correcting `docs/architecture/frontend-backend-bridge.md`'s prior,
  unaudited claim of 20 components down to the **8 actually confirmed to
  exist** (`button`, `badge`, `card`, `input`, `textarea`, `container`,
  `section`, `nav.header`).
- Updated `ROADMAP.md`'s build order: removed ZodiCore, ZodiBank,
  ZodiCapital, and ZodiYield from the from-scratch build queue (all four
  are alternate-base audit/extend products, not qfsfountains clones), added
  ZodiChain to the queue, and renumbered accordingly.

### Added — ZodiChain product spec; dual trading/swap-execution architecture for ZodiTrade, ZodiXchange, ZodiChain

- Added `docs/products/ZodiChain/SPEC.md`: a new, Foundation-depth product
  spec covering multi-chain custodial/non-custodial wallet management,
  WalletConnect integration, an NFT marketplace, crypto-to-crypto and
  crypto-to-fiat swap functionality, and a multi-level crypto-denominated
  affiliate/referral system layered on the inherited base engine's referral
  program. Promoted from a previously-unwritten "future expansion" idea to
  an active product on direct confirmation of two reference codebases on
  the build server: `dash` (confirmed via `package.json` to be "Bicrypto"
  v6.3.0, a Node.js/TypeScript pnpm monorepo — feature/UX reference only,
  never ported) and `web3chainlink` (an ordinary Laravel application with a
  `Modules/`-pattern directory, not yet fully audited — see the spec's
  `## Open Questions`).
- Corrected `docs/products/ZodiTrade/SPEC.md` and
  `docs/products/ZodiXchange/SPEC.md`'s Architecture sections to explicitly
  document that both products are fresh Laravel builds on the sanitized
  qfsfountains base, and that `dash`/Bicrypto and `web3chainlink` are
  feature/UX references only, never a base either product clones or ports
  code from.
- Added a dual trading/swap-execution-mode architecture decision to
  ZodiTrade (external clearing-broker API vs. an internal admin-configured
  pricing engine), ZodiXchange (external liquidity API vs. the internal
  matching engine), and ZodiChain (external swap-liquidity API vs. an
  internal admin-priced or AMM-style swap engine) — each following the
  same pluggable-gateway abstraction pattern established in
  `docs/standards/payment-gateways.md`, admin-selectable per
  `docs/standards/admin-configuration-baseline.md`.
- Updated `PRODUCT_CATALOG.md`: added the ZodiChain row (bringing the
  catalog to 21 products) and noted the promotion rationale and the
  `web3chainlink` open-verification item in the "Every product is
  independent" section.

### Changed — ZodiBank architecture correction: built on Pay Secure, not the qfsfountains base

- Corrected `docs/products/ZodiBank/SPEC.md`'s Architecture (§11), Modules &
  Submodules (§13), and Core Data Model (§14) sections. A direct filesystem
  audit of the actual build server confirmed ZodiBank's real codebase
  (`/home/script/public_html/zodibank/`) is **not** cloned from the
  sanitized qfsfountains-sourced base codebase — it is built on "Pay
  Secure," a separate, already-feature-complete commercial Laravel banking
  product using `nwidart/laravel-modules`, with `Modules/Agent` and
  `Modules/Merchant` already present, and Authorize.Net, Flutterwave,
  CoinGate, and CinetPay already integrated.
- Documented that Fixed Deposit Receipt (FDR), Deposit Pension Scheme
  (DPS), account-number generation, and staff/branch management do not
  exist anywhere in Pay Secure's codebase (confirmed by direct code
  search) and must be built fresh as new `Modules/*` packages, modeled on
  — but not inherited from — the equivalent qfsfountains base-codebase
  tables documented in `docs/architecture/base-codebase-strategy.md`.
- Added `docs/products/ZodiBank/FINCRA_INTEGRATION.md`: a new
  admin-configurable Fincra integration module spec (Payins, Payouts,
  Virtual Accounts, Identity Management), following the
  `docs/standards/payment-gateways.md` and
  `docs/standards/admin-configuration-baseline.md` patterns, with exact
  Fincra API header/field names explicitly flagged as unverified pending
  a live-docs check at implementation time.

### Added — build execution tracking and process gates

- Added `docs/architecture/deployment-paths.md`: documents the
  `/home/script/public_html/<product-slug>/` build working-directory
  convention, distinct from a buyer's own sale-time deployment.
- Added `BUILD_STATE.md`: the root-level resumability ledger tracking every
  product's build status, with the session-continuity protocol (verify
  on-disk state before trusting the ledger, commit small, never re-run
  destructive setup on in-progress work, stop and flag rather than guess).
- Marked `ZodiTrack` as `Live — Extend Only` in `PRODUCT_CATALOG.md` (new
  status definition) — it already exists as a complete, working, resold
  product and is removed from `ROADMAP.md`'s build-from-scratch queue
  entirely. Documented the audit-only, additive-extension-only workflow for
  any product in this status.
- Fixed the "Spec Complete" gate in
  `docs/checklists/production-readiness-checklist.md`: it no longer
  requires full ER diagrams/exhaustive endpoint catalogs before
  implementation begins (only ZodiCore had these). That depth is now a
  **GA gate** requirement, written just-in-time per module as it's
  implemented. Updated `ROADMAP.md` Phase 4 and its Non-goals section to
  match, and to state plainly that this repository, while still
  documentation-only, is the authoritative blueprint Claude Code executes
  against when building each product's real codebase.

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

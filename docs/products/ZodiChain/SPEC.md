# ZodiChain — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth) at the bottom of
> this document. See [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for
> spec status definitions.

ZodiChain is a standalone, self-hosted Laravel application built by cloning
the sanitized [base codebase](../../architecture/base-codebase-strategy.md),
running the
[genericization checklist](../../architecture/product-genericization-checklist.md)
to strip the base engine's banking-specific loan/DPS/FDR/branch tables, and
layering crypto-wallet, NFT-marketplace, swap, and multi-level-affiliate
domain modules on top. It does not depend on any other Zodize product or on
a central "ZodiCore" platform for identity, billing, notifications, or
tenancy — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
`ZodiCore` is itself just another standalone product in the catalog (a
general-purpose back-office/ERP starter), not a platform ZodiChain runs on.

ZodiChain was promoted from a previously-unwritten "future expansion" idea
to an active, specified product on direct confirmation that both a
feature/UX reference codebase (`dash`/Bicrypto) and a Laravel-based
crypto-adjacent commercial reference codebase (`web3chainlink`) exist on the
build server — see [§11.1](#111-reference-codebases-featureux-study-only-never-ported)
and [Open Questions](#open-questions).

## 1. Vision

ZodiChain is a self-hosted crypto wallet, NFT marketplace, and swap platform
for operators launching a digital-asset consumer product — real multi-chain
wallet custody (both custodial and non-custodial), a real NFT marketplace,
real crypto-to-crypto and crypto-to-fiat swap execution, and a real
multi-level affiliate/commission program — not a thin wrapper around a
single custodian's SDK. It gives an operator the wallet, marketplace, swap,
and referral infrastructure to run a credible consumer crypto product from
day one, distinct from ZodiTrade's brokerage focus and ZodiXchange's
exchange-infrastructure focus: ZodiChain is the consumer-facing wallet/NFT/
swap product, not an order-matching venue or a brokerage OMS.

## 2. Purpose

Launching a consumer crypto wallet product today means integrating
multi-chain node/RPC access, a custodial key-management solution, a
non-custodial wallet-connect flow, an NFT marketplace contract layer, a
swap/liquidity execution path, and a referral/affiliate commission engine
separately, then bolting compliance and audit on top. ZodiChain exists to
give that integration surface as one coherent, self-hosted platform built on
a base codebase whose RBAC, KYC, wallet ledger, and referral engine already
work, so an operator's engineering team builds the product experience, not
the plumbing.

## 3. Target Market

Digital wallet operators, NFT marketplace launchers, crypto-adjacent fintech
apps building a consumer swap product, and affiliate-driven crypto platforms
(referral/MLM-style crypto earning apps). Buyers are typically a crypto
product founder, a Head of Product at a digital-asset startup, or a fintech
CTO evaluating a build-vs-buy decision against an in-house wallet/NFT stack.

## 4. Industries

Cryptocurrency / digital assets, consumer fintech, digital collectibles
(NFTs).

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Multi-chain wallet platform (custodial + non-custodial) | Trust Wallet, Coinbase Wallet | Self-hosted per operator brand/deployment — each buyer runs their own independent instance with no shared infrastructure or cross-brand data exposure |
| Consumer crypto exchange/wallet suite | Binance (consumer app tier), Coinbase (consumer app tier) | Ships with RBAC/audit/KYC already built into the inherited base codebase, faster to stand up a compliant consumer wallet product |
| NFT marketplace | OpenSea, Rarible | Marketplace listing/escrow logic shares one audit trail with the wallet ledger, not a reconciled downstream copy |
| WalletConnect-based external wallet linking | MetaMask's WalletConnect ecosystem, Trust Wallet's dApp browser | Standardized `WalletConnectionContract` shared with ZodiXchange's custody abstraction pattern |
| Crypto/UX feature reference (category reference, not a competitor) | Bicrypto (`dash`, feature/UX reference only, per §11.1) | Reimplemented as a fresh PHP/Laravel codebase (§11.1), not ported Node.js code, so the deployment still runs on ordinary shared/VPS hosting |
| Multi-level affiliate/referral crypto commissions | Various MLM-crypto platforms (category, not a single named product) | Layers on top of the inherited base engine's existing referral program (`ref_by` tree) rather than building a parallel referral system, per [wallet-system.md](../../standards/wallet-system.md) |

## 6. Personas

- **Wallet Operations Manager** — oversees custodial wallet float, hot/cold
  balance policy, and withdrawal exception handling.
- **Compliance Officer** — reviews KYC submissions, AML flags on swap and
  withdrawal activity, and sanctioned-address screening results.
- **Marketplace Curator** — manages NFT collection approval, featured
  listings, and marketplace fee configuration.
- **End User (Wallet Holder)** — holds custodial and/or linked
  non-custodial balances, executes swaps, buys/sells/mints NFTs.
- **Affiliate/Referral Partner** — refers new users and earns multi-level,
  crypto-denominated commissions per the admin-configured referral program.
- **Buyer's own IT/support staff** — the only support layer this deployment
  has; there is no Zodize-operated support console, since each deployment is
  the buyer's own standalone codebase (see
  [admin-template.md](../../templates/admin-template.md)).

## 7. User Journeys

1. **Wallet creation and funding**: a new user registers → the platform
   provisions a custodial wallet address per supported chain automatically,
   and the user may separately link an external non-custodial wallet via
   WalletConnect → user deposits crypto to their custodial address or funds
   via a configured payment gateway on-ramp → balance reflects in the
   in-app wallet dashboard.
2. **Crypto-to-crypto swap**: user selects a source and destination asset
   and an amount → the swap engine (internal or external, per
   [§11.2](#112-dual-swap-execution-mode-external-liquidityswap-api-vs-internal-engine))
   quotes a rate → user confirms → the swap executes, debiting the source
   asset balance and crediting the destination asset balance, both recorded
   as ledger entries.
3. **NFT mint, list, and purchase**: a creator mints an NFT into their
   custodial or connected wallet → lists it on the marketplace with a fixed
   price or auction → a buyer purchases it → the marketplace escrow
   releases the NFT to the buyer and the sale proceeds (minus marketplace
   fee) to the seller's wallet balance, fully audit-logged.
4. **WalletConnect external wallet linking**: a user scans or clicks a
   WalletConnect session request from the ZodiChain UI → approves the
   connection from their external wallet app → ZodiChain reads the linked
   wallet's balance/address for display and can request the user's
   signature for a marketplace purchase or swap without ever holding that
   wallet's private key.
5. **Multi-level affiliate commission flow**: an existing user shares their
   referral link → a new user signs up and later executes their first swap
   or NFT purchase → a multi-level, crypto-denominated commission is
   credited up the referral chain per the admin-configured commission
   percentage and level depth, mirroring the inherited base engine's
   referral trigger pattern.
6. **Withdrawal and compliance review**: a user requests a withdrawal of
   custodial wallet funds to an external address → the platform screens the
   destination address against a configured sanctioned-address list → an
   Admin (or auto-approval below a configured threshold) approves the
   withdrawal → the payout executes and the balance debits.

## 8. Business Goals

- Let an operator launch a multi-chain wallet, NFT marketplace, and swap
  product without building custody, marketplace escrow, or swap execution
  infrastructure from scratch.
- Keep every wallet balance change, swap, NFT transaction, and referral
  commission traceable end-to-end for compliance review and user dispute
  resolution.
- Give a buyer a credible affiliate-driven growth mechanism (multi-level,
  crypto-denominated commissions) built on the same trusted ledger as every
  other balance change, not a bolt-on tracking spreadsheet.

## 9. Functional Requirements

- Multi-chain wallet management: custodial wallet provisioning per
  supported chain/asset, plus non-custodial external wallet linking via
  WalletConnect.
- NFT marketplace: minting, fixed-price listing, auction listing, escrowed
  purchase/settlement, and creator royalty distribution on resale.
- Crypto-to-crypto and crypto-to-fiat swap: quote generation, execution,
  and settlement across both the internal and external execution modes
  (§11.2).
- Multi-level affiliate/referral program: layered on the inherited base
  engine's referral tree (`ref_by` on `users`), with crypto-denominated
  (not only base-currency) commission payouts per level, per
  [wallet-system.md](../../standards/wallet-system.md) and
  [admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md#referral-program).
- Sanctioned-address / AML screening on withdrawal and swap destination
  addresses, with a configurable screening provider.
- Deposit/withdrawal for each supported chain/asset, reusing the inherited
  wallet engine's deposit/withdrawal approval flow extended for
  multi-asset balances (§14).
- Full second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (large withdrawal review, NFT collection approval), rule
  engine (AML/sanctioned-address screening rules), saved filters and global
  search over wallets/transactions/NFT listings, custom fields on NFT
  collection records, full audit history, soft delete with restore on
  non-ledger entities, mass actions (bulk NFT listing moderation),
  import/export wizards for wallet and NFT collection data, command
  palette, scheduled/report-builder reporting, system health dashboard,
  white-labeling of the wallet/marketplace UI.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiChain-specific additions:

- Swap quote generation targets p95 < 400ms from request to quoted rate,
  whether sourced from the internal engine or an external liquidity API.
- WalletConnect session establishment targets p95 < 3s from QR scan/deep
  link to an active, signable session.
- 99.9% uptime target for wallet balance read and swap-quote services.

## 11. Architecture

ZodiChain is built by cloning the sanitized
[base codebase](../../architecture/base-codebase-strategy.md) — a single,
independent Laravel application the buyer deploys entirely on their own
shared/VPS hosting, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
Building ZodiChain means running the full
[genericization checklist](../../architecture/product-genericization-checklist.md):
the inherited `loans`/`dps`/`fdr`/`branches`/`other_banks` tables and
controllers are stripped (they do not apply to a consumer crypto wallet
product), and ZodiChain keeps and builds on top of the base engine's
wallet/ledger, payment gateways, RBAC/auth (`Role`/`Permission` models),
KYC, referral program, i18n, and admin configuration surface (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#inherited-as-is-the-admin-engine-every-product-keeps)).

On that foundation, ZodiChain adds its own domain modules — Wallet
(Custodial & Non-Custodial), WalletConnect Integration, NFT Marketplace,
Swap Engine, and Affiliate/Referral Extension — as new, clearly bounded
modules per [module-template.md](../../templates/module-template.md),
registering into the inherited audit log and RBAC policy registry rather
than building parallel systems. Because ZodiChain genuinely needs a user to
hold balances in multiple crypto assets simultaneously, it extends the
inherited wallet engine per §14 rather than using `User.balance` directly
for crypto balances. Chain node/RPC access, swap execution, and NFT
marketplace settlement run as dedicated queued workers (per
[caching-queues-events.md](../../architecture/caching-queues-events.md))
isolated from the request path, with a `ChainNodeContract`,
`SwapExecutionContract`, and `WalletConnectionContract` abstraction so
chain/vendor integrations are swappable without touching wallet or
marketplace core logic. ZodiChain has no runtime dependency on any other
Zodize product or on a Zodize-operated central service; the only external
dependencies are the third-party chain nodes, swap liquidity/custody
providers, and KYC/AML screening integrations the buyer's own deployment
configures (§22).

### 11.1 Reference codebases: feature/UX study only, never ported

A direct filesystem audit of the build server identified two existing
crypto/trading codebases the engineering team may be tempted to reuse:
`dash` (confirmed via its own `package.json` to be "Bicrypto" v6.3.0, at
`/home/dash/public_html`) and `web3chainlink` (at
`/home/web3chainlink/public_html/project/`). **Neither is a base ZodiChain
clones, forks, or ports code from.** `dash`/Bicrypto is a Node.js/TypeScript
pnpm monorepo (separate `frontend/`/`backend/` directories, PM2 process
management via `ecosystem.config.js`, a 91MB `bicryptoV6.3.zip` archive of
the full source, install via a large custom `installer.sh`) — a
fundamentally different runtime from the PHP/Laravel, shared/VPS-hosting-
deployable architecture every Zodize product commits to (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
and [overview.md](../../architecture/overview.md)). Its Node
process-manager deployment model is incompatible with a buyer running the
product on ordinary shared/VPS hosting with zero DevOps involvement, so its
code is never ported. `web3chainlink`, by contrast, is confirmed to be an
ordinary Laravel application (has `app/`, `artisan`, `composer.json`, a
`Modules/`-pattern directory, and a `licence.php` file suggesting
license-gating logic) — but its exact functional coverage relative to
ZodiChain's wallet/NFT/swap scope has not been fully audited; see
[Open Questions](#open-questions) below.

Both codebases are used exclusively as **feature and UX references**:
ZodiChain's multi-chain custodial/non-custodial wallet management,
WalletConnect integration, NFT marketplace, swap functionality, and
MLM/affiliate referral systems are fresh PHP/Laravel implementations, built
against this spec's own data model (§14) and endpoint catalog (§15), that
aim for equivalent buyer-facing capability to what `dash`/Bicrypto
demonstrates (multi-chain custodial + non-custodial wallets, WalletConnect,
an NFT marketplace, futures/copy trading, P2P trading with escrow,
forex/AI investment plans, and MLM/affiliate systems) — never a
line-for-line or structural port of either reference codebase's source.
ZodiChain's own scope is deliberately narrower than the full Bicrypto
feature set: it covers the wallet/NFT/swap/affiliate slice, not Bicrypto's
spot/binary trading, market making, or forex/AI investment-plan surface,
which is ZodiTrade's and ZodiXchange's domain instead.

### 11.2 Dual swap-execution mode: external liquidity/swap API vs. internal engine

ZodiChain's swap functionality supports two execution modes, selected per
deployment (and, where the admin configures it, per asset pair) from the
admin panel — never hard-coded to one or the other:

- **External API mode**: the buyer's admin panel accepts credentials for a
  third-party liquidity provider, swap aggregator, or exchange API (§22),
  and the `SwapExecutionContract` routes swap quotes and execution through
  that external API. This is the mode an operator with an existing
  liquidity relationship uses.
- **Internal/native swap engine mode**: no third-party liquidity or swap
  API is required. ZodiChain implements its own swap pricing internally —
  either a simple admin-priced internal engine (the operator sets and
  updates conversion rates per asset pair from the admin panel) or, where
  the deployment enables it, an AMM-style (automated market maker)
  constant-product pricing model against an operator-funded internal
  liquidity pool. This mode lets a buyer launch with zero third-party
  swap-liquidity dependency and start offering swaps immediately after
  install.

Both modes implement the same `SwapExecutionContract` interface, following
the same pluggable-gateway pattern documented in
[payment-gateways.md](../../standards/payment-gateways.md) for payment
gateways: the wallet/swap core (balance ledger, audit log, AML screening)
is written once against the contract and is unaware which implementation is
active. A buyer can switch a given asset pair from the internal engine to
an external API (or vice versa) from the admin panel — per
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md)'s
pattern of zero-code-change configuration — without any change to
application code elsewhere. `ExternalSwapApiProvider` and
`InternalSwapEngineProvider` (itself supporting either admin-priced or
AMM-style internal pricing) are the two concrete implementations shipped by
default; a third-party or custom provider can be added following the same
contract.

## 12. Technology

Laravel (PHP) + Blade/Bootstrap/jQuery per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-frontend.md](../../development/coding-standards-frontend.md);
PostgreSQL for wallet/NFT/swap records + Redis for swap-quote caching and
WalletConnect session state per
[database-standards.md](../../development/database-standards.md); chain
node/RPC connectivity (self-hosted or third-party node provider, per chain)
behind a `ChainNodeContract`; WalletConnect v2 protocol support for
non-custodial external wallet linking; WebSocket channels (via Laravel
Reverb/Echo conventions) for streaming swap quote updates and NFT
marketplace bid/listing activity to the client.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Wallet | Custodial Wallet Provisioning, Multi-Asset Balances, Deposits, Withdrawals |
| WalletConnect Integration | Session Handshake, Signature Requests, Linked-Wallet Balance Read |
| NFT Marketplace | Minting, Fixed-Price Listings, Auctions, Escrowed Settlement, Creator Royalties |
| Swap Engine | Quote Generation, Internal Pricing/AMM, External Liquidity Routing, Settlement |
| Affiliate/Referral Extension | Multi-Level Commission Rules, Crypto-Denominated Payouts, Referral Tree (inherited `ref_by`) |
| Compliance | KYC Review (inherited), AML/Sanctioned-Address Screening, Withdrawal Review |

## 14. Core Data Model

The entities below are the load-bearing core; full ER diagram is queued
(see [Roadmap (spec depth)](#roadmap-spec-depth)). There is no `tenant_id`
column anywhere in this model — each deployed instance belongs to exactly
one operator, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md#what-single-tenant-changes-in-the-data-model).

| Entity | Key columns |
|---|---|
| `custodial_wallets` | id, user_id, chain, asset, address, key_management_ref, created_at |
| `linked_wallets` | id, user_id, chain, address, walletconnect_session_id, linked_at, revoked_at |
| `crypto_balances` | id, user_id, asset, chain, available_balance, held_balance, updated_at |
| `crypto_transactions` | id, crypto_balance_id, amount (signed), asset, trigger (deposit/withdrawal/swap/nft_sale/nft_purchase/referral_commission), post_balance_snapshot, reference_id, created_at |
| `nft_collections` | id, creator_id, name, chain, contract_ref, status, created_at |
| `nfts` | id, collection_id, owner_id, token_ref, metadata_json, minted_at |
| `nft_listings` | id, nft_id, seller_id, listing_type (fixed/auction), price, status, expires_at |
| `nft_sales` | id, listing_id, buyer_id, sale_price, marketplace_fee, royalty_paid, settled_at |
| `swaps` | id, user_id, source_asset, destination_asset, source_amount, destination_amount, execution_mode (internal/external), rate, executed_at |
| `swap_liquidity_pools` | id, asset_pair, pool_balance_a, pool_balance_b, updated_at |
| `referral_commissions` | id, referrer_id, referred_user_id, level, asset, amount, trigger_event, credited_at |
| `sanctioned_address_screens` | id, address, chain, checked_at, result, provider |

**Multi-asset crypto balances**: ZodiChain genuinely needs a user to hold
balances in multiple crypto assets across multiple chains simultaneously —
this is the product's fundamental reason for existing. Per
[ADR-0002](../../decisions/0002-single-currency-wallet-by-default.md), the
inherited single-currency `User.balance`/`Transaction` engine is not
extended globally; ZodiChain instead extends the pattern in its own domain
module following
[wallet-system.md's Multi-currency gap](../../standards/wallet-system.md#multi-currency-gap):
`crypto_balances` is ZodiChain's own multi-asset balances table (`asset`
plus `chain` together are the `currency`-equivalent dimension), scoped per
`user_id`, and `crypto_transactions` is the corresponding append-only,
post-balance-snapshot ledger, mirroring the inherited `Transaction` model's
invariants rather than the shared base engine's schema itself. Referral
commissions credited in a crypto asset (rather than the deployment's base
currency) post through `crypto_transactions` with trigger
`referral_commission`, keeping the multi-level affiliate payout on the same
self-verifying ledger as every other balance change, per
[admin-configuration-baseline.md's referral section](../../standards/admin-configuration-baseline.md#referral-program).

## 15. Key API Endpoints

The endpoints below are the primary implementation surface; the full
catalog is queued (see [Roadmap (spec depth)](#roadmap-spec-depth)). All
conform to [api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md).

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/wallets` | Provision a custodial wallet for a chain/asset |
| GET | `/api/v1/wallets/balances` | Fetch a user's multi-asset balances |
| POST | `/api/v1/wallets/walletconnect/session` | Initiate a WalletConnect linking session |
| DELETE | `/api/v1/wallets/walletconnect/{id}` | Revoke a linked external wallet |
| POST | `/api/v1/wallets/deposit-address` | Fetch/generate a deposit address for an asset |
| POST | `/api/v1/withdrawals` | Request a withdrawal to an external address |
| POST | `/api/v1/withdrawals/{id}/approve` | Approve a pending withdrawal (admin/ops) |
| POST | `/api/v1/swaps/quote` | Get a swap quote for an asset pair/amount |
| POST | `/api/v1/swaps` | Execute a swap at a quoted rate |
| GET | `/api/v1/swaps/{id}` | Fetch swap execution/settlement status |
| POST | `/api/v1/nft/collections` | Create an NFT collection |
| POST | `/api/v1/nft/mint` | Mint a new NFT into a collection |
| POST | `/api/v1/nft/listings` | List an NFT for fixed-price sale or auction |
| POST | `/api/v1/nft/listings/{id}/purchase` | Purchase a fixed-price NFT listing |
| POST | `/api/v1/nft/listings/{id}/bid` | Place a bid on an NFT auction listing |
| GET | `/api/v1/referrals/tree` | Fetch a user's referral tree and commission history |
| GET | `/api/v1/compliance/sanctioned-address-check` | Screen an address before withdrawal/swap |
| GET | `/api/v1/reports/wallet-activity` | Generate a wallet activity report |
| GET | `/api/v1/reports/marketplace-volume` | Generate an NFT marketplace volume report |

## 16. Events

Domain events registered on the inherited event bus (see
[caching-queues-events.md](../../architecture/caching-queues-events.md)):
`wallet.provisioned`, `wallet.linked` (WalletConnect), `wallet.link_revoked`,
`deposit.completed`, `withdrawal.requested`, `withdrawal.approved`,
`withdrawal.rejected`, `swap.quoted`, `swap.executed`, `swap.failed`,
`nft.minted`, `nft.listed`, `nft.sold`, `nft.bid_placed`,
`referral.commission_earned`, `sanctioned_address.flagged`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `deposit.completed` | ✔ | ✔ | — | ✔ |
| `withdrawal.approved` | ✔ | ✔ | — | ✔ |
| `withdrawal.rejected` | ✔ | ✔ | — | — |
| `swap.executed` | ✔ | — | — | ✔ |
| `nft.sold` (to seller) | ✔ | ✔ | — | ✔ |
| `nft.bid_placed` (to current listing owner) | ✔ | — | — | ✔ |
| `referral.commission_earned` | ✔ | ✔ | — | — |
| `sanctioned_address.flagged` | ✔ (to compliance queue) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Built on the inherited `Role`/`Permission` engine per
[admin-template.md](../../templates/admin-template.md), with
crypto-specific roles registered on top of the
[default system roles](../../security/rbac-permissions.md#default-system-roles):
`Wallet Operations Manager`, `Compliance Officer`, `Marketplace Curator`.
Key permissions: `wallets.manage`, `withdrawals.approve`,
`withdrawals.approve_any` (ops-only override), `swaps.configure_pricing`,
`swaps.approve_external_provider`, `nft.approve_collection`,
`nft.moderate_listing`, `compliance.screen_address`,
`referrals.configure_commission`. Full model per
[rbac-permissions.md](../../security/rbac-permissions.md).

## 19. Workflows & Approval Chains

- **Large withdrawal review**: withdrawals above an admin-configured
  threshold route to a Wallet Operations Manager for approval before the
  external payout executes.
- **NFT collection approval**: a new NFT collection requires Marketplace
  Curator approval before its NFTs can be listed for public sale, with the
  approval decision audit-logged.
- **Sanctioned-address hold**: a withdrawal or swap destination flagged by
  the AML/sanctioned-address screen is held pending Compliance Officer
  review and cannot auto-complete regardless of amount.
- **External swap-provider enablement**: switching an asset pair from
  internal engine mode to external API mode (§11.2) requires Wallet
  Operations Manager confirmation of the configured provider's credentials
  before the switch takes effect.

## 20. Audit Logs

Every wallet provisioning event, deposit, withdrawal decision, swap
execution, NFT mint/listing/sale, referral commission credit, and
sanctioned-address screening result writes an immutable audit entry via the
inherited audit log ([audit-logging.md](../../security/audit-logging.md)),
capturing actor (including "system" for automated swap execution),
timestamp, and before/after state. `crypto_transactions` and `nft_sales`
records are never edited in place — corrections occur via a new offsetting
record referencing the original.

## 21. Reports & Analytics & Dashboards

- Operational dashboard: wallet balance totals by asset, swap volume,
  pending withdrawal queue, NFT marketplace listing/sale activity, referral
  commission payout totals — per
  [dashboard-standards.md](../../standards/dashboard-standards.md).
- Compliance: sanctioned-address screening register, withdrawal approval
  history, KYC status summary.
- Marketplace: NFT collection performance, top sellers, marketplace fee
  revenue.
- Growth: referral tree depth and commission payout summary by level.
- Report builder and scheduled report delivery per the second-layer
  baseline in [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Chain nodes / RPC providers**: multi-chain node connectivity (e.g. an
  Infura/Alchemy-class RPC provider, or a self-hosted node) behind a
  `ChainNodeContract`, one adapter per supported chain.
- **WalletConnect**: WalletConnect v2 protocol integration for linking
  external non-custodial wallets, behind a `WalletConnectionContract`.
- **Swap liquidity / execution APIs** (external API mode, §11.2): a
  third-party liquidity provider or swap aggregator API. Not required
  where an asset pair runs in internal engine mode.
- **KYC/AML and sanctioned-address screening**: account onboarding uses the
  inherited base engine's KYC form-builder and review flow (see
  [admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md#kyc)),
  plus a configurable sanctioned-address screening provider for
  withdrawal/swap destinations.
- **Payment gateways**: fiat on/off-ramp via the inherited gateway system
  (see [payment-gateways.md](../../standards/payment-gateways.md)),
  including its confirmed cryptocurrency gateways (BTCPay Server,
  CoinGate) as an additional funding path alongside direct on-chain
  deposit.
- **NFT storage/metadata**: off-chain metadata and media storage (e.g. an
  IPFS pinning service) for NFT assets minted through the marketplace.

## 23. AI Features

- **NFT listing description assistant**: AI-assisted marketplace listing
  description and tag suggestions from an uploaded NFT's metadata/media,
  always reviewed by the creator before publishing.
- **Swap rate anomaly detection**: layered on top of the inherited audit
  log's anomaly detection, flags internal-engine swap rates that deviate
  materially from a reference external rate for Wallet Operations review
  before the rate is used further.
- **AML/sanctioned-address triage assistant**: plain-language summary of
  why an address was flagged (network, screening provider, match
  confidence) to speed Compliance Officer review — never auto-clears a
  flag.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: hot-wallet/cold-wallet balance reconciliation, sanctioned-
  address list sync, NFT listing expiration sweep (auctions), referral
  commission batch calculation, external swap-provider health check.
- CLI commands (Artisan): `chain:reconcile-wallets`,
  `chain:sync-sanctioned-list`, `chain:expire-nft-listings`,
  `chain:calculate-referral-commissions` — each requires the same
  authorization context as its API equivalent.

## 25. Seed/Demo Data

`ChainDemoSeeder` provisions the demo deployment with custodial wallets
across several chains/assets for a set of synthetic users, a handful of
linked (simulated) WalletConnect sessions, a populated NFT marketplace with
multiple collections and both fixed-price and auction listings, 6 months of
swap history across both internal-engine and external-API execution modes,
and a multi-level referral tree with crypto-denominated commission history
— per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: wallet balance reads reflect the latest completed
deposit/withdrawal/swap within 1 second, and the NFT marketplace listing
feed remains responsive (p95 < 1s) for a marketplace with tens of thousands
of active listings.

## 27. Security Requirements

Financial/crypto products carry Zodize's highest security/compliance bar.
Full baseline from
[security-standards.md](../../security/security-standards.md) applies,
plus:

- **Custodial key management**: private keys backing custodial wallets are
  never stored in plaintext application code or logs, and key-management
  operations are isolated from the general application request path.
- **PCI-DSS-equivalent handling** for any stored fiat funding-instrument
  data (on-ramp for wallet funding), tokenized and never logged in
  plaintext.
- **SOC2-equivalent controls**: change management and access review apply
  to wallet, swap-execution, and NFT-marketplace-settlement modules with
  the same rigor as the inherited base engine.
- **KYC/AML and sanctioned-address screening** required before a user can
  request their first external withdrawal.
- **Immutable audit trails**: wallet, swap, and NFT-sale audit entries are
  append-only, matching §20.
- **MFA is mandatory, not optional**, for every human user role approving
  withdrawals, managing wallet operations, or reviewing compliance flags,
  enforced at the deployment's security policy level per
  [authentication-authorization.md](../../security/authentication-authorization.md).
- Sanctioned-address holds (§19) are a security/compliance control, not a
  workflow convenience, and cannot be disabled by an admin of the
  deployment.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated swap-execution regression suite validating internal-engine and
external-API modes produce consistent settlement amounts for a known rate
fixture set, and an NFT-marketplace-escrow test suite covering every legal
and illegal listing/bid/settlement state transition.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md), onto
the buyer's own shared/VPS hosting per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
Wallet and swap-quote services deploy with a zero-downtime requirement, with
chain-node connectivity changes scheduled to avoid interrupting in-flight
swap or withdrawal processing.

## 30. Acceptance Criteria

- A user can provision a custodial wallet, deposit an asset, and see the
  resulting balance update, entirely through the API.
- A user can link an external wallet via WalletConnect and have its balance
  reflected without ZodiChain ever holding that wallet's private key.
- A swap quote, once accepted, executes and settles correctly in both
  internal-engine and external-API modes, producing matching audit entries.
- An NFT can be minted, listed, purchased, and settled with the correct
  marketplace fee and creator royalty applied.
- A referral commission is credited in the correct crypto asset to the
  correct level of the referral tree on a triggering event.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md)
and [security-checklist.md](../../checklists/security-checklist.md).
ZodiChain additionally requires sign-off from a compliance stakeholder that
KYC/AML screening, sanctioned-address holds, and custodial key-management
practices have been validated against the buyer's actual regulatory
obligations before go-live.

## 32. Future Roadmap

- Additional chain support beyond the initial launch chain set.
- Fractionalized NFT ownership support.
- Cross-chain swap support (swapping an asset on one chain for an asset on
  a different chain) beyond same-chain swaps.

## 33. Known Risks

- Chain node/RPC dependency: wallet balance freshness and swap execution
  depend on the integrated chain node/RPC provider's reliability —
  mitigated by the `ChainNodeContract` abstraction, but remains an external
  dependency in external API mode.
- Custodial key-management risk: any custodial wallet product concentrates
  key-management risk; mitigated by isolating key operations from the
  general request path (§27), but this remains the single highest-severity
  risk category for this product.
- Internal AMM-style pricing risk: an under-funded internal liquidity pool
  can produce poor swap rates or fail to fill a quoted swap — mitigated by
  the admin-priced internal engine alternative (§11.2), but the AMM-style
  option requires the operator to understand pool-funding requirements.

## 34. Future Improvements

- Configurable royalty-split support for multi-creator NFT collaborations.
- Tiered referral commission structures beyond a flat per-level percentage.

## Open Questions

- **`web3chainlink`'s actual functional scope is not yet fully audited.**
  The build-server audit confirmed `web3chainlink` (at
  `/home/web3chainlink/public_html/project/`) is an ordinary Laravel
  application — `app/`, `artisan`, `composer.json` (generic
  `laravel/laravel` package name, so it is a white-labeled commercial
  script, not identifiable by package name alone), a `Modules/` directory
  (`nwidart/laravel-modules` pattern, the same pattern as ZodiBank's Pay
  Secure base), and a `licence.php` file at the root suggesting
  license-gating logic common to CodeCanyon-style commercial scripts.
  Confirmed payment/crypto-adjacent composer dependencies include
  `flutterwavedev/flutterwave-v3`, `anandsiddharth/laravel-paytm-wallet`,
  `bacon/bacon-qr-code`, and `barryvdh/laravel-dompdf`. Its `Modules/`
  directory contents and README.md were **not** inspected during this audit
  pass — a follow-up session MUST open `Modules/` and confirm exactly which
  functional slice (wallet/NFT/swap-relevant vs. something else entirely)
  it actually implements before this spec treats it as a validated
  feature/UX reference for any specific ZodiChain module beyond the
  general observation that it demonstrates a real Laravel-based
  payment/crypto-adjacent commercial script exists. Do not assume
  equivalence to Bicrypto's feature set until that follow-up audit runs.

## Roadmap (spec depth)

This spec was newly written to promote ZodiChain from a previously-
unwritten "future expansion" idea to an active, Foundation-depth product,
following direct confirmation of the `dash`/Bicrypto feature/UX reference
and the `web3chainlink` Laravel-based reference codebase on the build
server. ZodiChain is a fresh Laravel build on the sanitized qfsfountains
base per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
and the
[genericization checklist](../../architecture/product-genericization-checklist.md);
`dash`/Bicrypto and `web3chainlink` are feature/UX references only
(§11.1), and the dual external-API/internal-engine swap-execution
architecture (§11.2) is the documented default for ZodiChain's swap
functionality. This spec is Foundation-depth. Queued for Deep-depth
expansion: a full ER diagram and migration set for the wallet/NFT/swap
schema (companion `DATA_MODEL.md`), a complete endpoint catalog (companion
`API_REFERENCE.md`), and a full report catalog covering additional
marketplace and referral analytics. Changes follow
[CONTRIBUTING.md](../../../CONTRIBUTING.md).

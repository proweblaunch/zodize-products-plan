# ZodiTrade — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth) at the bottom of
> this document. See [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for
> spec status definitions.

ZodiTrade is a standalone, self-hosted Laravel application built by cloning
the sanitized [base codebase](../../architecture/base-codebase-strategy.md),
running the
[genericization checklist](../../architecture/product-genericization-checklist.md)
to strip the base engine's banking-specific loan/DPS/FDR/branch tables, and
layering brokerage-domain modules on top. It does not depend on any other
Zodize product or on a central "ZodiCore" platform for identity, billing,
notifications, or tenancy — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
`ZodiCore` is itself just another standalone product in the catalog (a
general-purpose back-office/ERP starter), not a platform ZodiTrade runs on.

## 1. Vision

ZodiTrade is a multi-asset brokerage and trading platform for broker-
dealers, RIAs building a self-directed offering, and fintech trading apps
that need real order management, real settlement accounting, and real tax
lot tracking — not a market-data widget bolted onto a generic dashboard. It
gives a brokerage operator the order lifecycle, portfolio, margin, and
settlement infrastructure to run a compliant trading product from day one.

## 2. Purpose

Launching a brokerage today means integrating an order management system, a
clearing/settlement relationship, a market data feed, and tax lot
accounting separately, then bolting compliance and audit on top. ZodiTrade
exists to give that integration surface as one coherent, self-hosted
platform built on a base codebase whose RBAC, KYC, and audit engine already
work, so a broker-dealer's engineering team builds the product experience,
not the plumbing.

## 3. Target Market

Introducing broker-dealers, RIAs adding a self-directed trading tier,
fintech trading apps operating under a clearing-broker relationship, and
family offices/wealth platforms that need in-house portfolio and order
tooling. Buyers are typically a Head of Trading Operations, Chief
Compliance Officer, or a fintech CTO evaluating a build-vs-buy decision
against an in-house OMS build.

## 4. Industries

Capital markets, wealth management, retail and institutional brokerage.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Order management system | Charles River IMS, FlexTrade, Bloomberg EMSX | Ships with RBAC/audit already built into the inherited base codebase, faster to stand up a compliant OMS |
| Retail brokerage platform | Interactive Brokers' platform, Robinhood's internal stack, Alpaca | Standalone, self-hosted per brokerage brand/program — each buyer runs their own independent deployment with no shared infrastructure or cross-brand data exposure |
| Portfolio/positions tooling | Addepar, Black Diamond | Positions and tax lots share one audit trail with orders and settlement, not a reconciled downstream copy |
| Market data distribution | Refinitiv Eikon feeds, Polygon.io, IEX Cloud | Feed-agnostic ingestion contract so a buyer can swap vendors without touching the OMS |
| Trade confirmation/compliance | Broadridge, SS&C Advent | Confirmations and audit trail generated directly from the settlement ledger of record |
| Binary/fixed-payout trading UX | Bicrypto-class binary/AI-trading tools (feature/UX reference only, per §11.1) | Reimplemented as a fresh PHP/Laravel internal pricing engine (§11.2), not ported Node.js code, so the deployment still runs on ordinary shared/VPS hosting |

## 6. Personas

- **Trading Operations Manager** — oversees order flow, exceptions, and
  settlement operations.
- **Compliance Officer** — reviews trade surveillance flags, restricted
  lists, and best-execution reporting.
- **Retail/Institutional Trader** — places orders, manages watchlists,
  reviews portfolio and tax lot detail.
- **Margin/Risk Analyst** — monitors margin utilization, maintenance calls,
  and concentration risk.
- **Back-Office/Settlement Analyst** — reconciles trades to settlement and
  produces trade confirmations.
- **Buyer's own IT/support staff** — the only support layer this deployment
  has; there is no Zodize-operated support console, since each deployment is
  the buyer's own standalone codebase (see
  [admin-template.md](../../templates/admin-template.md)).

## 7. User Journeys

1. **Order placement and routing**: trader selects an instrument, chooses
   order type (market/limit/stop) and quantity → pre-trade checks run
   (buying power, restricted list, margin requirement) → order routes to
   the configured broker/venue integration → order status streams back
   (`accepted → partially_filled → filled`/`canceled`) to the trader's
   blotter in real time.
2. **Margin account maintenance call**: a position's value drop pushes an
   account's margin utilization above the maintenance threshold → the
   system generates a maintenance call → account is flagged with a
   configurable cure window → if uncured, the risk engine can trigger a
   forced liquidation of the lowest-priority position, fully audit-logged.
3. **Trade settlement and confirmation**: a filled order enters settlement
   processing on the applicable T+1/T+2 cycle → settlement status updates
   as clearing confirms → on settlement, a trade confirmation document is
   generated and delivered to the account holder, and the position/tax lot
   records update.
4. **Tax lot disposal at sale**: trader sells a partial position → the tax
   lot accounting engine selects lots per the account's configured method
   (FIFO/LIFO/specific-lot) → realized gain/loss is computed per lot →
   disposal detail feeds the year-end tax reporting export.
5. **Watchlist to order flow**: trader builds a watchlist of instruments →
   real-time market data feed streams quotes to the watchlist → trader
   places an order directly from a watchlist row, pre-filling the order
   ticket with the last quoted price for a limit order.

## 8. Business Goals

- Let a broker-dealer or fintech launch multi-asset trading without
  building an OMS, portfolio engine, or tax lot accounting from scratch.
- Keep every order, fill, and settlement traceable end-to-end for
  regulatory examination and client dispute resolution.
- Reduce time-to-compliance-readiness by generating best-execution and
  surveillance reports directly from the order/fill ledger.

## 9. Functional Requirements

- Order management: market, limit, stop, and stop-limit order types, with
  time-in-force (day, GTC, IOC, FOK) support.
- Broker/venue routing: pluggable routing to one or more execution venues
  or broker integrations, with smart-order-routing hooks for future
  expansion.
- Portfolio and positions: real-time position tracking, cost basis, unrealized
  P&L, and account-level portfolio views across asset classes.
- Margin accounts: buying power calculation, maintenance requirement
  monitoring, margin call generation, and configurable forced-liquidation
  policy.
- Trade settlement: T+1/T+2 (asset-class-configurable) settlement tracking
  from trade date through settlement date, with settlement exception
  handling.
- Watchlists: user-created and shared watchlists with real-time quote
  streaming.
- Real-time market data feeds: level 1 (and optionally level 2) quote
  ingestion, normalized across vendor feeds.
- Tax lot accounting: FIFO, LIFO, and specific-lot identification methods,
  configurable per account, with realized/unrealized gain-loss reporting.
- Trade confirmations: generated per filled/settled trade, delivered per
  account holder notification preferences, and archived per retention
  policy.
- Full second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (margin liquidation override, large-order review),
  rule engine (pre-trade compliance checks), saved filters and global
  search over orders/positions, custom fields on account records, full
  audit history, soft delete with restore on non-ledger entities, mass
  actions (bulk order cancel), import/export wizards for account and
  position data, command palette, scheduled/report-builder reporting,
  system health dashboard, white-labeling of the trading UI.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiTrade-specific additions:

- Order submission-to-acknowledgment latency targets p95 < 250ms measured
  from API receipt to venue/broker routing confirmation.
- Market data quote propagation to a subscribed client targets p95 < 500ms
  from feed receipt to UI update.
- 99.95% uptime target for order entry and portfolio-read services during
  each configured market's trading hours.

## 11. Architecture

ZodiTrade is built by cloning the sanitized
[base codebase](../../architecture/base-codebase-strategy.md) — a single,
independent Laravel application the buyer deploys entirely on their own
shared/VPS hosting, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
Building ZodiTrade means running the full
[genericization checklist](../../architecture/product-genericization-checklist.md):
the inherited `loans`/`dps`/`fdr`/`branches`/`other_banks` tables and
controllers are stripped (they do not apply to a brokerage), and ZodiTrade
keeps and builds on top of the base engine's wallet/ledger, payment
gateways, RBAC/auth (`Role`/`Permission` models), KYC, i18n, and admin
configuration surface (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#inherited-as-is-the-admin-engine-every-product-keeps)).

On that foundation, ZodiTrade adds its own domain modules — Order
Management, Routing, Portfolio, Margin, Settlement, Market Data, Tax Lot
Accounting, and Compliance — as new, clearly bounded modules per
[module-template.md](../../templates/module-template.md), registering into
the inherited audit log and RBAC policy registry rather than building
parallel systems. Because ZodiTrade needs brokerage cash balances in
multiple currencies simultaneously per account, it extends the inherited
wallet engine per §14 rather than using `User.balance` directly for
account cash. Market data ingestion and order routing run as dedicated
queued/streaming workers (per
[caching-queues-events.md](../../architecture/caching-queues-events.md))
isolated from the request path, with a `MarketDataFeedContract` and
`BrokerRoutingContract` abstraction so venue/vendor integrations are
swappable without touching OMS core logic. ZodiTrade has no runtime
dependency on any other Zodize product or on a Zodize-operated central
service; the only external dependencies are the third-party clearing
broker, market data, and KYC integrations the buyer's own brokerage
configures (§22).

### 11.1 Reference codebases: feature/UX study only, never ported

A direct filesystem audit of the build server identified two existing
crypto/trading codebases the engineering team may be tempted to reuse:
`dash` (confirmed via its own `package.json` to be "Bicrypto" v6.3.0,
at `/home/dash/public_html`) and `web3chainlink` (at
`/home/web3chainlink/public_html/project/`). **Neither is a base ZodiTrade
clones, forks, or ports code from.** `dash`/Bicrypto is a Node.js/TypeScript
pnpm monorepo (separate `frontend/`/`backend/` directories, PM2 process
management via `ecosystem.config.js`) — a fundamentally different runtime
from the PHP/Laravel, shared/VPS-hosting-deployable architecture every
Zodize product commits to (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
and [overview.md](../../architecture/overview.md)). Its Node process-manager
deployment model is incompatible with a buyer running the product on
ordinary shared/VPS hosting with zero DevOps involvement, so its code is
never ported. `web3chainlink`, by contrast, is confirmed to be an ordinary
Laravel application (has `app/`, `artisan`, `composer.json`, a
`Modules/`-pattern directory) and is a closer structural relative of
ZodiTrade's own target architecture — but its exact functional coverage
relative to ZodiTrade's brokerage/OMS scope has not been fully audited; see
[Open Questions](#open-questions) below.

Both codebases are used exclusively as **feature and UX references**:
ZodiTrade's Order Management, Routing, Portfolio, Margin, and Settlement
modules are fresh PHP/Laravel implementations, built against this spec's
own data model (§14) and endpoint catalog (§15), that aim for equivalent
buyer-facing capability to what these reference codebases demonstrate —
never a line-for-line or structural port of either one's source.

### 11.2 Dual trading-execution mode: external API vs. internal engine

ZodiTrade's order routing supports two execution modes, selected per
deployment (and, where the admin configures it, per instrument or asset
class) from the admin panel — never hard-coded to one or the other:

- **External API mode**: the buyer's admin panel accepts credentials for a
  third-party clearing broker or execution venue (§22), and the
  `BrokerRoutingContract` routes orders through that external API. This is
  the mode a broker-dealer with an existing clearing relationship uses.
- **Internal/native trading engine mode**: no third-party broker/venue API
  is required. ZodiTrade implements its own order execution logic
  internally — for binary-style or fixed-payout instrument types, an
  admin-configured payout-odds/pricing model determines settlement price
  and payout without any external counterparty; for standard equity/ETF
  instrument types in this mode, ZodiTrade fills orders against an
  admin-configured reference price feed rather than routing to a venue.
  This mode lets a buyer launch with zero third-party trading-execution
  dependency and start taking orders immediately after install.

Both modes implement the same `BrokerRoutingContract` interface (already
established as the routing abstraction earlier in this section), following
the same pluggable-gateway pattern documented in
[payment-gateways.md](../../standards/payment-gateways.md) for payment
gateways: the OMS core (order state machine, portfolio, margin, settlement,
audit log) is written once against the contract and is unaware which
implementation is active. A buyer can switch from the internal engine to an
external API (or vice versa) from the admin panel — per
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md)'s
pattern of zero-code-change configuration — without any change to
application code elsewhere. `ExternalBrokerExecutionProvider` and
`InternalPricingEngineProvider` are the two concrete implementations
shipped by default; a third-party or custom provider can be added following
the same contract.

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL for order/position/settlement records + Redis for real-time
quote caching and order-status pub/sub per
[database-standards.md](../../development/database-standards.md); WebSocket
channels (via Laravel Reverb/Echo conventions) for streaming order status
and market data to the client.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Order Management | Order Entry, Order Types, Time-in-Force, Order Blotter, Cancel/Replace |
| Routing | Broker/Venue Adapters, Routing Rules, Execution Reports |
| Portfolio | Positions, Cost Basis, Unrealized P&L, Multi-Asset Aggregation |
| Margin | Buying Power, Maintenance Monitoring, Margin Calls, Forced Liquidation |
| Settlement | Settlement Cycle Tracking, Exception Handling, Trade Confirmations |
| Market Data | Feed Ingestion, Quote Normalization, Watchlists |
| Tax Lot Accounting | Lot Selection Methods, Realized/Unrealized Gain-Loss, Year-End Export |
| Compliance | Pre-Trade Checks, Restricted Lists, Surveillance Flags, Best-Execution Reporting |

## 14. Core Data Model

The 12 entities below are the load-bearing core; full ER diagram is queued
(see [Roadmap (spec depth)](#roadmap-spec-depth)). There is no `tenant_id`
column anywhere in this model — each deployed instance belongs to exactly
one broker-dealer/RIA/fintech, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md#what-single-tenant-changes-in-the-data-model).

| Entity | Key columns |
|---|---|
| `brokerage_accounts` | id, customer_id, account_type (cash/margin), status, opened_at |
| `orders` | id, account_id, instrument_id, side, order_type, quantity, limit_price, stop_price, time_in_force, status |
| `executions` | id, order_id, fill_quantity, fill_price, venue, executed_at |
| `positions` | id, account_id, instrument_id, quantity, average_cost, updated_at |
| `tax_lots` | id, position_id, acquired_at, quantity, cost_basis, disposed_at, disposal_method |
| `instruments` | id, symbol, asset_class, exchange, tick_size, tradable |
| `settlements` | id, order_id, trade_date, settlement_date, status, settled_at |
| `trade_confirmations` | id, settlement_id, document_id, delivered_at |
| `margin_accounts` | id, account_id, buying_power, maintenance_requirement, status |
| `margin_calls` | id, margin_account_id, amount_due, issued_at, cure_deadline, resolved_at |
| `watchlists` | id, user_id, name, is_shared |
| `watchlist_items` | id, watchlist_id, instrument_id, sort_order |

**Multi-currency brokerage cash balances**: ZodiTrade genuinely needs a
brokerage account to hold cash in more than one currency simultaneously
(e.g. a USD settlement balance and a EUR settlement balance on the same
account, for a multi-market brokerage). Per
[ADR-0002](../../decisions/0002-single-currency-wallet-by-default.md), the
inherited single-currency `User.balance`/`Transaction` engine is not
extended globally; ZodiTrade instead extends the pattern in its own domain
module following
[wallet-system.md's Multi-currency gap](../../standards/wallet-system.md#multi-currency-gap):

| Entity | Key columns |
|---|---|
| `brokerage_account_balances` | id, brokerage_account_id, currency, cash_balance, updated_at |
| `brokerage_cash_transactions` | id, brokerage_account_balance_id, amount (signed), currency, trigger (deposit/withdrawal/trade_settlement/fee/dividend), post_balance_snapshot, reference_id, created_at |

`brokerage_account_balances` is scoped per `brokerage_account_id`, not per
user, so a single trader with multiple brokerage accounts (e.g. an
individual and a joint account) holds independent currency balances per
account. `brokerage_cash_transactions` follows the same append-only,
post-balance-snapshot invariant as the inherited `Transaction` model — a
correction is a new offsetting row, never an edit to a past one.

## 15. Key API Endpoints

The endpoints below are the primary implementation surface; the full
catalog is queued (see [Roadmap (spec depth)](#roadmap-spec-depth)). All
conform to [api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md).

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/orders` | Place a new order |
| PATCH | `/api/v1/orders/{id}/cancel` | Cancel an open order |
| PATCH | `/api/v1/orders/{id}/replace` | Cancel/replace an open order |
| GET | `/api/v1/accounts/{id}/orders` | List orders for an account |
| GET | `/api/v1/accounts/{id}/positions` | List current positions |
| GET | `/api/v1/accounts/{id}/portfolio` | Aggregated portfolio view |
| GET | `/api/v1/accounts/{id}/margin` | Margin account status/buying power |
| GET | `/api/v1/margin-calls` | List open margin calls |
| POST | `/api/v1/margin-calls/{id}/resolve` | Mark a margin call resolved |
| GET | `/api/v1/accounts/{id}/settlements` | List settlement status by trade |
| GET | `/api/v1/trade-confirmations/{id}` | Fetch a trade confirmation document |
| GET | `/api/v1/accounts/{id}/tax-lots` | List tax lot detail |
| PATCH | `/api/v1/accounts/{id}/tax-lot-method` | Set default lot selection method |
| GET | `/api/v1/instruments/{symbol}/quote` | Fetch latest quote |
| GET | `/api/v1/watchlists` | List a user's watchlists |
| POST | `/api/v1/watchlists` | Create a watchlist |
| POST | `/api/v1/watchlists/{id}/items` | Add an instrument to a watchlist |
| GET | `/api/v1/compliance/restricted-list` | Fetch the current restricted list |
| GET | `/api/v1/compliance/surveillance-flags` | List open surveillance flags |
| GET | `/api/v1/reports/best-execution` | Generate a best-execution report |

## 16. Events

Domain events registered on the inherited event bus (see
[caching-queues-events.md](../../architecture/caching-queues-events.md)):
`order.submitted`, `order.accepted`, `order.rejected`, `order.filled`,
`order.partially_filled`, `order.canceled`, `position.updated`,
`margin.call_issued`, `margin.call_resolved`, `margin.liquidation_triggered`,
`settlement.completed`, `settlement.exception_raised`,
`trade_confirmation.generated`, `tax_lot.disposed`,
`surveillance.flag_raised`, `restricted_list.updated`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `order.filled` | ✔ | — | — | ✔ |
| `order.rejected` | ✔ | ✔ | — | ✔ |
| `margin.call_issued` | ✔ | ✔ | ✔ | ✔ |
| `margin.liquidation_triggered` | ✔ | ✔ | ✔ | ✔ |
| `settlement.exception_raised` | ✔ (to ops queue) | ✔ | — | — |
| `trade_confirmation.generated` | ✔ | ✔ (document attached/linked) | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Built on the inherited `Role`/`Permission` engine per
[admin-template.md](../../templates/admin-template.md), with
brokerage-specific roles registered on top of the
[default system roles](../../security/rbac-permissions.md#default-system-roles):
`Trading Operations Manager`,
`Compliance Officer`, `Margin/Risk Analyst`, `Settlement Analyst`, `Trader`.
Key permissions: `orders.place`, `orders.cancel`, `orders.cancel_any`
(ops-only override), `margin.issue_call`, `margin.override_liquidation`,
`compliance.manage_restricted_list`, `compliance.review_surveillance`,
`settlements.resolve_exception`, `tax_lots.change_method`. Full model per
[rbac-permissions.md](../../security/rbac-permissions.md).

## 19. Workflows & Approval Chains

- **Large-order review**: orders above an admin-configured notional
  threshold route to a Trading Operations Manager for pre-trade approval
  before routing to the venue.
- **Margin call cure/liquidation**: a margin call enters a cure window;
  if uncured, forced liquidation requires the risk engine's automated
  decision to be logged and is subject to a configurable manual-override
  approval by a Margin/Risk Analyst before execution, unless the deployment
  has opted into fully automated liquidation.
- **Restricted list changes**: adding/removing an instrument from a
  restricted list requires Compliance Officer approval and is
  audit-logged with an effective-date.
- **Settlement exception resolution**: a failed or exception-flagged
  settlement requires a Settlement Analyst to resolve or escalate before
  the trade confirmation can be generated.

## 20. Audit Logs

Every order state transition, fill, margin call, liquidation decision,
settlement status change, restricted-list change, and tax lot disposal
writes an immutable audit entry via the inherited audit log
([audit-logging.md](../../security/audit-logging.md)), capturing actor
(including "system" for automated liquidation), timestamp, and before/after
state. Order and execution records are never edited in place — corrections
occur via a new offsetting record referencing the original.

## 21. Reports & Analytics & Dashboards

- Operational dashboard: order volume, fill rate, average execution
  latency, open margin calls, settlement exception backlog — per
  [dashboard-standards.md](../../standards/dashboard-standards.md).
- Compliance: best-execution report, surveillance flag register,
  restricted-list change history.
- Client-facing: portfolio performance summary, realized/unrealized
  gain-loss statement, year-end tax lot export.
- Report builder and scheduled report delivery per the second-layer
  baseline in [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Clearing/execution brokers** (external API mode, §11.2): order routing
  to a clearing broker or execution venue (e.g. an Apex/Alpaca-class
  clearing relationship or direct exchange connectivity) behind a
  `BrokerRoutingContract`. Where a deployment instead runs in internal
  engine mode, no clearing-broker integration is required at all — see
  §11.2.
- **Market data vendors**: real-time quote feeds (e.g. Polygon.io, IEX
  Cloud, Refinitiv-class vendors) behind a `MarketDataFeedContract`,
  normalized to one internal quote schema.
- **KYC/AML and suitability verification**: account opening uses the
  inherited base engine's KYC form-builder and review flow (see
  [admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md#kyc)),
  plus investor-suitability questionnaires for margin/options approval
  tiers.
- **Tax reporting**: year-end 1099-B-style export integration for realized
  gain/loss reporting.
- **Corporate actions data**: dividend, split, and symbol-change feeds that
  keep positions and tax lots accurate through corporate events.

## 23. AI Features

- Trade surveillance assistant: pattern detection layered on top of the
  inherited audit log's anomaly detection, tuned for wash-trading-like and
  layering-like order sequences, routed to the Compliance queue for human
  review.
- Portfolio insight summaries: plain-language summary of a client's
  portfolio concentration, unrealized gain/loss position, and margin
  utilization, generated on demand for the trader/advisor persona.
- Order-entry assistant: natural-language order intent ("buy $5,000 of
  AAPL") translated into a pre-filled order ticket that still requires
  explicit user confirmation before submission — never auto-submits.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: end-of-day settlement status polling, margin
  maintenance-requirement recalculation, tax lot year-end export
  generation, restricted-list sync, trade confirmation batch generation.
- CLI commands (Artisan): `trade:recalculate-margin`,
  `trade:poll-settlements`, `trade:generate-confirmations`,
  `trade:sync-restricted-list` — each requires the same authorization
  context as its API equivalent.

## 25. Seed/Demo Data

`TradeDemoSeeder` provisions the demo deployment with a realistic instrument
universe across equities/ETFs/options, 100+ synthetic accounts with varied
cash/margin status, 12 months of order and execution history, a populated
tax lot ledger spanning multiple lot methods, and open/resolved margin call
and surveillance-flag history — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: portfolio and position reads reflect the latest
settled and unsettled activity within 1 second of an execution event, and
the order blotter supports real-time updates for at least 5,000 concurrent
open orders per deployment without polling.

## 27. Security Requirements

Financial products carry Zodize's highest security/compliance bar. Full
baseline from [security-standards.md](../../security/security-standards.md)
applies, plus:

- **PCI-DSS-equivalent handling** for any stored payment/funding
  instrument data (ACH/card funding of a brokerage account), tokenized and
  never logged in plaintext.
- **SOC2-equivalent controls**: change management and access review apply
  to order-routing and settlement modules with the same rigor as the
  inherited base engine.
- **KYC/AML and suitability verification** required before an account can
  place its first order, including enhanced due diligence for margin and
  options trading tiers.
- **Immutable audit trails**: order, execution, and settlement audit
  entries are append-only, matching §20.
- **MFA is mandatory, not optional**, for every human user role placing
  orders or approving margin/compliance actions, enforced at the
  deployment's security policy level per
  [authentication-authorization.md](../../security/authentication-authorization.md).
- Large-order review and margin-liquidation override (§19) are security/
  risk controls, not workflow conveniences, and cannot be disabled by an
  admin of the deployment.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated order-state-machine test suite covering every legal and illegal
transition, and a tax-lot-accounting regression suite validating FIFO/LIFO/
specific-lot gain-loss calculations against known reference cases.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md), onto
the buyer's own shared/VPS hosting per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
Order entry and market data streaming services deploy with a zero-downtime
requirement during the deployment's configured market hours, with deploys
scheduled outside trading hours where feasible.

## 30. Acceptance Criteria

- A trader can place a market or limit order, see it route and fill, and
  see the resulting position and tax lot update, entirely through the API.
- A margin account crossing the maintenance threshold generates a margin
  call and, if configured for automated liquidation, executes it with a
  complete audit trail.
- A settled trade produces a trade confirmation document delivered per the
  account's notification preferences.
- Tax lot disposal calculations match reference FIFO/LIFO/specific-lot
  results for a defined regression fixture set.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md)
and [security-checklist.md](../../checklists/security-checklist.md).
ZodiTrade additionally requires sign-off from a compliance stakeholder that
pre-trade checks, restricted-list enforcement, and best-execution reporting
have been validated against the buyer's actual broker-dealer/RIA
obligations before go-live.

## 32. Future Roadmap

- Options chain support with multi-leg order types.
- Fixed income and international equity asset-class expansion.
- Algorithmic/programmatic order submission via a dedicated API tier.

## 33. Known Risks

- Venue/broker dependency: order fill latency and reliability depend on
  the integrated execution broker — mitigated by the
  `BrokerRoutingContract` abstraction, but remains an external dependency.
- Market data vendor divergence: quote latency and depth vary by vendor
  tier; the normalized quote schema abstracts format differences but not
  underlying feed latency.

## 34. Future Improvements

- Smart order routing across multiple execution venues by price/liquidity.
- Configurable margin methodology beyond Reg T (e.g. portfolio margining).

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
  functional slice (brokerage/OMS-relevant vs. exchange/wallet-relevant vs.
  something else entirely) it actually implements before this spec treats
  it as a validated feature/UX reference for any specific ZodiTrade module
  beyond the general observation that it demonstrates a real Laravel-based
  payment/crypto-adjacent commercial script exists. Do not assume
  equivalence to Bicrypto's feature set until that follow-up audit runs.

## Roadmap (spec depth)

This spec's Architecture and Core Data Model sections were revised to
reflect the corrected standalone, self-hosted, single-tenant deployment
model — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)
and [base-codebase-strategy.md](../../architecture/base-codebase-strategy.md).
This spec was further revised to correct a possible misreading of the
`dash`/Bicrypto and `web3chainlink` reference codebases as something
ZodiTrade could be cloned or ported from: neither is a base ZodiTrade
builds on. ZodiTrade remains a fresh Laravel build on the sanitized
qfsfountains base per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
and the
[genericization checklist](../../architecture/product-genericization-checklist.md);
`dash`/Bicrypto and `web3chainlink` are feature/UX references only (§11.1),
and the dual external-API/internal-engine trading-execution architecture
(§11.2) is now the documented default for ZodiTrade's order routing. This
spec is Foundation-depth. Queued for Deep-depth expansion: a full ER
diagram and migration set for the order/position/settlement schema
(companion `DATA_MODEL.md`), a complete endpoint catalog (companion
`API_REFERENCE.md`), and a full report catalog covering additional
asset-class-specific compliance reports. Changes follow
[CONTRIBUTING.md](../../../CONTRIBUTING.md).

# ZodiXchange — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth) at the bottom of
> this document. See [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for
> spec status definitions.

ZodiXchange is a standalone, self-hosted Laravel application built by
cloning the sanitized [base codebase](../../architecture/base-codebase-strategy.md),
running the
[genericization checklist](../../architecture/product-genericization-checklist.md)
to strip the base engine's banking-specific loan/DPS/FDR/branch tables, and
layering exchange-domain modules on top. It does not depend on any other
Zodize product or on a central "ZodiCore" platform for identity, billing,
notifications, or tenancy — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
`ZodiCore` is itself just another standalone product in the catalog (a
general-purpose back-office/ERP starter), not a platform ZodiXchange runs
on.

## 1. Vision

ZodiXchange is exchange infrastructure for operators launching a spot and/
or derivatives trading venue — a real matching engine, real custody
integration, and real market surveillance, not a thin order-form UI in
front of someone else's liquidity. It gives an exchange operator the
matching, settlement, and surveillance infrastructure to run a licensed,
defensible trading venue from day one.

## 2. Purpose

Standing up exchange infrastructure today typically means building a
matching engine from scratch or licensing one in isolation from custody,
surveillance, and liquidity-provider tooling. ZodiXchange exists to give an
exchange operator a coherent, self-hosted venue platform — matching,
custody integration, LP connectivity, and surveillance — built on a base
codebase whose RBAC, KYC, and audit engine already work, so the operator's
engineering effort goes into market design and liquidity, not plumbing.

## 3. Target Market

Licensed digital asset exchanges, regional/national securities exchanges
launching a digital venue, ATS (alternative trading system) operators, and
institutional trading venues offering both spot and derivatives products.
Buyers are typically a Head of Market Operations, Chief Compliance Officer,
or exchange CTO evaluating a build-vs-buy decision against an in-house
matching engine build.

## 4. Industries

Capital markets, digital asset exchanges, derivatives markets.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Matching engine infrastructure | Nasdaq Matching Engine (licensed), B2C2/DXtrade-class venue tech, Talos | Ships with RBAC/audit already built into the inherited base codebase, faster to stand up a compliant venue |
| Digital asset exchange platform | Coinbase Prime infrastructure tier, ErisX/CoinFLEX-class venue tech | Custody integration and surveillance share one audit trail with matching, not bolted-on separately |
| Market surveillance | Nasdaq SMARTS, Eventus Systems | Surveillance rules operate directly on the order/trade ledger of record, not a delayed downstream feed |
| Liquidity provider connectivity | FIX-based prime brokerage gateways, LMAX Digital LP APIs | Standardized LP API contract shared across every Zodize capital-markets product |
| Clearing/settlement | DTCC-class clearing infrastructure (traditional), on-chain settlement rails (digital assets) | Settlement abstraction lets an operator support both custodial and on-chain settlement models |
| Digital asset trading UX (order book, copy trading, P2P) | Bicrypto-class exchange/wallet tooling (feature/UX reference only, per §11.1) | Reimplemented as a fresh PHP/Laravel matching engine and custody layer (§11.2), not ported Node.js code, so the deployment still runs on ordinary shared/VPS hosting |

## 6. Personas

- **Market Operations Manager** — oversees trading session state, halts,
  and order book health.
- **Compliance/Market Surveillance Officer** — reviews manipulation
  detection alerts and manages market conduct rules.
- **Liquidity Provider (LP)** — connects via API to stream quotes and
  provide two-sided liquidity, earning maker rebates.
- **Institutional/Retail Trader** — places orders against the venue's order
  book via the trading UI or API.
- **Custody/Settlement Analyst** — manages wallet/custody reconciliation
  and settlement/clearing operations.
- **Buyer's own IT/support staff** — the only support layer this deployment
  has; there is no Zodize-operated support console, since each deployment is
  the buyer's own standalone codebase (see
  [admin-template.md](../../templates/admin-template.md)).

## 7. User Journeys

1. **Order submission to the matching engine**: trader or LP submits an
   order via API or UI → order enters the appropriate order book (by
   symbol/market) → the matching engine attempts to match against resting
   liquidity per price-time priority → resulting trade(s) generate maker/
   taker fee assessments and stream execution reports back to both parties
   in real time.
2. **Liquidity provider onboarding**: an LP applies for market-maker status
   → Market Operations reviews the LP's connectivity and capital
   requirements → on approval, the LP receives API credentials scoped to
   quoting/order endpoints and a maker fee tier → LP begins streaming
   two-sided quotes into the order book.
3. **Market surveillance alert triage**: the surveillance engine flags a
   pattern consistent with spoofing or wash trading → a case opens in the
   Compliance/Market Surveillance Officer's queue with the flagged order/
   trade sequence → officer reviews and dispositions (dismiss, warn,
   restrict account, escalate to regulator) → decision and rationale are
   audit-logged.
4. **Trading halt and resume**: an operational or regulatory trigger (e.g.
   extreme volatility, a surveillance escalation) requires a symbol-level
   or venue-wide halt → Market Operations Manager initiates the halt → new
   order acceptance for the affected symbol(s) stops, resting orders are
   preserved not canceled → on resume, the order book reopens per the
   deployment's configured reopening auction or continuous-resume policy.
5. **Settlement and custody reconciliation**: filled trades enter
   settlement/clearing per the market's settlement model (custodial
   ledger movement or on-chain settlement) → Custody/Settlement Analyst
   monitors settlement completion → any settlement exception (e.g. a
   custody balance mismatch) raises a case for resolution before the
   affected account's balances are released for further trading.

## 8. Business Goals

- Let an exchange operator launch a compliant spot and/or derivatives venue
  without building a matching engine, surveillance system, or LP
  connectivity layer from scratch.
- Keep every order, match, and settlement traceable end-to-end for
  regulatory examination and market conduct investigation.
- Attract and retain liquidity providers through a standardized, low-
  latency LP API and transparent maker/taker fee schedule.

## 9. Functional Requirements

- Matching engine: price-time priority continuous matching per symbol, with
  configurable market microstructure (tick size, lot size, opening/closing
  auction behavior).
- Order book depth: full depth-of-book (level 2/3) maintenance and
  distribution to subscribed clients.
- Order types: market, limit, stop, iceberg (hidden quantity), and
  post-only (maker-guaranteed) orders.
- Maker/taker fee schedule: configurable per deployment, per symbol, and per
  volume tier, applied at trade settlement.
- Custody/wallet integration: abstraction over custodial balance ledgers
  and/or on-chain wallet balances backing tradable assets.
- Liquidity provider APIs: low-latency order/quote submission, cancel, and
  market data subscription endpoints distinct from the retail trading API
  tier.
- Market surveillance/manipulation detection: rule-based detection for
  spoofing, layering, wash trading, and quote stuffing patterns, with a
  case management queue.
- Settlement/clearing: configurable settlement model (custodial ledger
  movement or on-chain settlement), with settlement exception handling.
- Trading session controls: symbol-level and venue-wide halts, opening/
  closing auctions, circuit breakers.
- Full second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  approval chains (LP onboarding, halt authorization), rule engine
  (surveillance rules), saved filters and global search over orders/trades,
  custom fields on account/LP records, full audit history, soft delete with
  restore on non-ledger entities, mass actions (bulk order cancel on
  halt), import/export wizards for market/symbol configuration, command
  palette, scheduled/report-builder reporting, system health dashboard,
  white-labeling of the trading UI.

## 10. Non-Functional Requirements

See [performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md) for the
inherited baseline. ZodiXchange-specific additions:

- Matching engine order-to-acknowledgment latency targets p99 < 5ms measured
  from order receipt to book acceptance, materially stricter than the
  general API latency budget because matching engine performance is
  competitive market infrastructure.
- Market data (order book depth) distribution targets p99 < 20ms from
  matching engine event to subscriber delivery.
- 99.99% uptime target for the matching engine and order-entry gateway
  during each configured market's trading session.

## 11. Architecture

ZodiXchange is built by cloning the sanitized
[base codebase](../../architecture/base-codebase-strategy.md) — a single,
independent Laravel application the buyer deploys entirely on their own
shared/VPS hosting (or a larger dedicated/VPS host where matching-engine
throughput requires it), per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
Building ZodiXchange means running the full
[genericization checklist](../../architecture/product-genericization-checklist.md):
the inherited `loans`/`dps`/`fdr`/`branches`/`other_banks` tables and
controllers are stripped (they do not apply to an exchange), and ZodiXchange
keeps and builds on top of the base engine's RBAC/auth (`Role`/`Permission`
models), KYC, i18n, and admin configuration surface (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#inherited-as-is-the-admin-engine-every-product-keeps)).
Because ZodiXchange needs multi-asset custody balances rather than a
single-currency wallet, it extends the inherited wallet engine's pattern per
§14 with its own `custody_balances` table rather than using `User.balance`.

On that foundation, ZodiXchange makes one deliberate architectural
exception: the matching engine itself runs as a dedicated, in-memory,
low-latency service per market/symbol (outside the request-response Laravel
app) that publishes order-book and execution events onto the application's
event bus (per
[caching-queues-events.md](../../architecture/caching-queues-events.md))
for the Laravel app to persist, audit, and expose via API/WebSocket. This
keeps the inherited RBAC/audit guarantees intact for every account and
order action while not forcing matching-engine-critical-path latency
through a general-purpose web framework request cycle. Custody/wallet
integration and settlement run behind a `CustodyLedgerContract` supporting
both custodial and on-chain settlement models. ZodiXchange has no runtime
dependency on any other Zodize product or on a Zodize-operated central
service; the only external dependencies are the third-party custody, KYC,
and clearing integrations the buyer's own venue configures (§22).

### 11.1 Reference codebases: feature/UX study only, never ported

A direct filesystem audit of the build server identified two existing
crypto/trading codebases the engineering team may be tempted to reuse:
`dash` (confirmed via its own `package.json` to be "Bicrypto" v6.3.0, at
`/home/dash/public_html`) and `web3chainlink` (at
`/home/web3chainlink/public_html/project/`). **Neither is a base ZodiXchange
clones, forks, or ports code from.** `dash`/Bicrypto is a Node.js/TypeScript
pnpm monorepo (separate `frontend/`/`backend/` directories, PM2 process
management via `ecosystem.config.js`) — a fundamentally different runtime
from the PHP/Laravel, shared/VPS-hosting-deployable architecture every
Zodize product commits to (see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
and [overview.md](../../architecture/overview.md)). Its Node
process-manager deployment model is incompatible with a buyer running the
product on ordinary shared/VPS hosting with zero DevOps involvement, so its
code is never ported. `web3chainlink`, by contrast, is confirmed to be an
ordinary Laravel application (has `app/`, `artisan`, `composer.json`, a
`Modules/`-pattern directory) — but its exact functional coverage relative
to ZodiXchange's exchange-infrastructure scope has not been fully audited;
see [Open Questions](#open-questions) below.

Both codebases are used exclusively as **feature and UX references** for
concepts like order-book/matching-engine design, multi-chain wallet
handling, copy trading, and P2P escrow: ZodiXchange's Matching Engine,
Custody & Settlement, Liquidity Provider Management, and Market
Surveillance modules are fresh PHP/Laravel implementations, built against
this spec's own data model (§14) and endpoint catalog (§15), that aim for
equivalent buyer-facing capability to what these reference codebases
demonstrate — never a line-for-line or structural port of either one's
source.

### 11.2 Dual trading-execution mode: external liquidity API vs. internal matching engine

ZodiXchange's order execution supports two modes, selected per deployment
(and, where the admin configures it, per market) from the admin panel —
never hard-coded to one or the other:

- **External API mode**: the buyer's admin panel accepts credentials for a
  third-party liquidity provider, prime brokerage, or execution API (§22),
  and orders route through that external connection instead of (or in
  addition to) the venue's own resting liquidity.
- **Internal/native trading engine mode**: the dedicated in-memory matching
  engine described earlier in this section *is* the internal engine — no
  third-party liquidity API is required for a market running purely against
  its own order book. This is ZodiXchange's out-of-the-box default: a buyer
  can launch a market with zero third-party liquidity dependency and let
  the matching engine work purely from the liquidity its own users and
  onboarded market makers (§13, Liquidity Provider Management) provide.

Both modes sit behind the same `CustodyLedgerContract`/order-routing
abstraction, following the same pluggable-gateway pattern documented in
[payment-gateways.md](../../standards/payment-gateways.md) for payment
gateways: the exchange core (order acceptance, audit log, custody ledger,
surveillance) is written once against the contract and is unaware which
implementation is active for a given market. A buyer can switch a market
from internal-only matching to blended/external-liquidity mode (or back)
from the admin panel — per
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md)'s
pattern of zero-code-change configuration — without any change to
application code elsewhere. `InternalMatchingEngineProvider` and
`ExternalLiquidityApiProvider` are the two concrete implementations shipped
by default; a third-party or custom provider can be added following the
same contract.

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md)
for the account, custody-reconciliation, surveillance, and admin surfaces;
the matching engine itself is a dedicated low-latency service (in-memory
order book per symbol, persisted asynchronously) integrated via the
platform event bus; PostgreSQL for order/trade/settlement records of
record + Redis for order book depth caching and real-time distribution per
[database-standards.md](../../development/database-standards.md);
WebSocket/FIX-style gateway for LP and institutional trading connectivity.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Matching Engine | Order Book Management, Price-Time Priority Matching, Auctions, Circuit Breakers |
| Order Gateway | Retail API, LP/Institutional API, FIX-Style Gateway, Rate Limiting |
| Market Data | Depth-of-Book Distribution, Trade Tape, Symbol Reference Data |
| Fees | Maker/Taker Schedule, Volume Tiers, Fee Assessment |
| Custody & Settlement | Custodial Ledger, On-Chain Settlement Adapter, Reconciliation |
| Liquidity Provider Management | LP Onboarding, Quoting Permissions, Rebate Calculation |
| Market Surveillance | Manipulation Detection Rules, Case Management, Market Conduct Actions |
| Session Controls | Halts, Auctions, Reopening Policy |

## 14. Core Data Model

The 12 entities below are the load-bearing core; full ER diagram is queued
(see [Roadmap (spec depth)](#roadmap-spec-depth)). There is no `tenant_id`
column anywhere in this model — each deployed instance belongs to exactly
one exchange operator, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md#what-single-tenant-changes-in-the-data-model).

| Entity | Key columns |
|---|---|
| `markets` | id, symbol, asset_class, tick_size, lot_size, status |
| `orders` | id, account_id, market_id, side, order_type, quantity, price, hidden_quantity, status |
| `trades` | id, market_id, buy_order_id, sell_order_id, price, quantity, executed_at |
| `order_book_snapshots` | id, market_id, sequence_number, captured_at, depth_json |
| `fee_schedules` | id, market_id, tier, maker_bps, taker_bps, effective_at |
| `custody_balances` | id, account_id, asset, available_balance, held_balance, updated_at |
| `settlements` | id, trade_id, settlement_model, status, settled_at |
| `liquidity_providers` | id, account_id, status, quoting_permissions, rebate_tier, onboarded_at |
| `surveillance_cases` | id, rule_triggered, related_order_ids, status, disposition, reviewed_by |
| `trading_halts` | id, market_id, reason, initiated_by, initiated_at, resumed_at |
| `market_makers_quotes` | id, liquidity_provider_id, market_id, bid_price, ask_price, updated_at |
| `symbol_reference_data` | id, market_id, description, underlying_asset, contract_spec_json |

**Multi-asset custody balances**: ZodiXchange genuinely needs an account to
hold balances in multiple tradable assets/currencies simultaneously — this
is the exchange's fundamental reason for existing. Per
[ADR-0002](../../decisions/0002-single-currency-wallet-by-default.md), the
inherited single-currency `User.balance`/`Transaction` engine is not
extended globally; `custody_balances` above already implements the required
extension pattern from
[wallet-system.md's Multi-currency gap](../../standards/wallet-system.md#multi-currency-gap)
— `asset` is ZodiXchange's `currency`-equivalent column, and the table is
scoped per `account_id` rather than per `User.balance`. The corresponding
append-only, post-balance-snapshot ledger row is `custody_ledger_entries`
(`id`, `custody_balance_id`, `amount` (signed), `asset`, `trigger`
(trade_settlement/deposit/withdrawal/fee), `post_balance_snapshot`,
`reference_trade_id`, `created_at`), mirroring the inherited `Transaction`
model's invariants rather than the shared base engine's schema itself.

## 15. Key API Endpoints

The endpoints below are the primary implementation surface; the full
catalog is queued (see [Roadmap (spec depth)](#roadmap-spec-depth)). All
conform to [api-standards.md](../../development/api-standards.md) and
[rest-standards.md](../../development/rest-standards.md).

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/orders` | Submit an order to the matching engine |
| DELETE | `/api/v1/orders/{id}` | Cancel an open order |
| GET | `/api/v1/markets/{symbol}/order-book` | Fetch current order book depth |
| GET | `/api/v1/markets/{symbol}/trades` | Fetch recent trade tape |
| WS | `/ws/v1/markets/{symbol}/stream` | Real-time order book/trade stream |
| GET | `/api/v1/accounts/{id}/orders` | List an account's orders |
| GET | `/api/v1/accounts/{id}/custody-balances` | Fetch custody balances |
| POST | `/api/v1/liquidity-providers/apply` | Apply for LP/market-maker status |
| PATCH | `/api/v1/liquidity-providers/{id}/permissions` | Update LP quoting permissions |
| GET | `/api/v1/fee-schedules` | List current maker/taker fee schedule |
| GET | `/api/v1/settlements/{trade_id}` | Fetch settlement status for a trade |
| POST | `/api/v1/markets/{symbol}/halt` | Initiate a trading halt |
| POST | `/api/v1/markets/{symbol}/resume` | Resume trading after a halt |
| GET | `/api/v1/surveillance-cases` | List surveillance case queue |
| POST | `/api/v1/surveillance-cases/{id}/disposition` | Resolve a surveillance case |
| GET | `/api/v1/markets` | List configured markets/symbols |
| POST | `/api/v1/markets` | Create/configure a new market |
| GET | `/api/v1/reports/market-activity` | Generate a market activity report |
| GET | `/api/v1/reports/lp-performance` | Generate an LP performance/rebate report |
| GET | `/api/v1/reports/surveillance-summary` | Generate a surveillance case summary report |

## 16. Events

Domain events registered on the inherited event bus (see
[caching-queues-events.md](../../architecture/caching-queues-events.md)):
`order.accepted`, `order.rejected`, `order.matched`, `order.canceled`,
`market.halted`, `market.resumed`, `market.opened`, `market.closed`,
`liquidity_provider.onboarded`, `liquidity_provider.suspended`,
`fee_schedule.updated`, `settlement.completed`,
`settlement.exception_raised`, `surveillance.case_opened`,
`surveillance.case_resolved`, `custody_balance.reconciled`,
`custody_balance.mismatch_detected`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `order.matched` | ✔ | — | — | ✔ (large fills) |
| `market.halted` | ✔ (all active traders) | ✔ (LPs/institutional) | — | ✔ |
| `liquidity_provider.onboarded` | ✔ | ✔ | — | — |
| `settlement.exception_raised` | ✔ (to ops queue) | ✔ | — | — |
| `surveillance.case_opened` | ✔ (to compliance queue) | — | — | — |
| `custody_balance.mismatch_detected` | ✔ (to ops queue) | ✔ | ✔ (on-call) | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Built on the inherited `Role`/`Permission` engine per
[admin-template.md](../../templates/admin-template.md), with
exchange-specific roles registered on top of the
[default system roles](../../security/rbac-permissions.md#default-system-roles):
`Market Operations Manager`,
`Compliance/Market Surveillance Officer`, `Liquidity Provider`,
`Custody/Settlement Analyst`, `Trader`. Key permissions: `markets.configure`,
`markets.halt`, `markets.resume`, `orders.submit`, `orders.cancel_any`
(ops-only override), `liquidity_providers.approve`,
`fee_schedules.manage`, `surveillance.disposition`,
`custody.reconcile`. Full model per
[rbac-permissions.md](../../security/rbac-permissions.md).

## 19. Workflows & Approval Chains

- **LP onboarding approval**: an LP application requires Market Operations
  Manager review of connectivity and capital requirements before quoting
  permissions activate; approval is audit-logged with the reviewer's
  identity.
- **Trading halt authorization**: a symbol-level or venue-wide halt
  requires Market Operations Manager (or an automated circuit-breaker
  trigger, itself logged as a system actor) authorization; resume requires
  an explicit action, never an automatic timeout without review for
  regulatory-triggered halts.
- **Surveillance case escalation**: cases dispositioned as "restrict
  account" or "escalate to regulator" require Compliance/Market
  Surveillance Officer sign-off and generate a downstream account-
  restriction workflow.
- **Fee schedule changes**: changes to maker/taker fee schedules require
  Market Operations Manager approval and take effect only from a
  configured future effective date, never retroactively.

## 20. Audit Logs

Every order acceptance/rejection, match, halt/resume, LP onboarding
decision, fee schedule change, settlement status change, and surveillance
disposition writes an immutable audit entry via the inherited audit log
([audit-logging.md](../../security/audit-logging.md)), capturing actor
(including "system" for automated circuit breakers), timestamp, and
before/after state. Order book snapshots are retained at a
regulator-defined granularity to support post-trade reconstruction of
market state at any point in time.

## 21. Reports & Analytics & Dashboards

- Operational dashboard: order book depth, trade volume, matching engine
  latency, active halts, surveillance case backlog — per
  [dashboard-standards.md](../../standards/dashboard-standards.md).
- Market activity: daily/monthly volume by symbol, maker/taker fee revenue,
  LP performance and rebate summary.
- Compliance: surveillance case register, market conduct action history,
  halt/resume audit trail.
- Report builder and scheduled report delivery per the second-layer
  baseline in [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog).

## 22. Integrations

- **Custody providers**: institutional custodian or qualified-custodian
  integration for asset safekeeping (traditional securities) or digital
  asset custody providers (e.g. Fireblocks/Anchorage-class vendors) behind
  a `CustodyLedgerContract`.
- **External liquidity/execution APIs** (external API mode, §11.2): a
  third-party liquidity provider, prime brokerage, or execution API a
  deployment can plug in to source liquidity beyond its own order book.
  Not required where a market runs purely on the internal matching engine
  (§11.2's default mode).
- **On-chain settlement**: blockchain node/wallet infrastructure
  integration for venues settling on-chain, abstracted so the settlement
  module is chain-agnostic where feasible.
- **Market data distribution**: outbound feed integration for operators
  that redistribute their own market data to third parties (consolidated
  tape-style distribution).
- **KYC/AML providers**: uses the inherited base engine's KYC form-builder
  and review flow (see
  [admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md#kyc))
  for account and LP onboarding.
- **Clearing houses**: optional integration for venues that clear through
  a third-party central counterparty rather than settling bilaterally.

## 23. AI Features

- Market surveillance pattern detection: the rule-based surveillance engine
  in §9 is augmented with anomaly-scoring that layers on top of the
  inherited audit log's anomaly detection, surfacing lower-confidence
  patterns for human review rather than auto-dispositioning.
- LP quote quality scoring: summarizes an LP's quoting behavior (spread
  tightness, uptime, fill quality) to help Market Operations manage rebate
  tiers.
- Halt-impact summary: on a trading halt, generates a plain-language
  summary of affected open orders and estimated market impact for Market
  Operations Manager review before resume.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: end-of-day custody balance reconciliation, order book
  snapshot archival, fee schedule effective-date activation, surveillance
  rule batch re-scan of recent trade history, LP rebate calculation.
- CLI commands (Artisan): `xchange:reconcile-custody`,
  `xchange:archive-order-books`, `xchange:calculate-rebates`,
  `xchange:rescan-surveillance` — each requires the same authorization
  context as its API equivalent.

## 25. Seed/Demo Data

`XchangeDemoSeeder` provisions the demo deployment with several configured
markets (spot and a simple derivatives symbol), a populated order book with
resting synthetic liquidity, 12 months of trade tape history, onboarded
demo liquidity providers with rebate history, and open/resolved
surveillance case examples — per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: the matching engine sustains at least 10,000 order
actions per second per symbol without queue backlog, and order book depth
snapshots distributed to subscribers never diverge from the true engine
state by more than the configured sequence-number tolerance.

## 27. Security Requirements

Financial products carry Zodize's highest security/compliance bar. Full
baseline from [security-standards.md](../../security/security-standards.md)
applies, plus:

- **PCI-DSS-equivalent handling** for any stored funding-instrument data
  (fiat on/off-ramp for a digital asset venue), tokenized and never logged
  in plaintext.
- **SOC2-equivalent controls**: change management, access review, and
  vendor risk management apply to the matching engine, custody integration,
  and surveillance modules with the same rigor as the inherited base
  engine, including change control specifically for matching-engine logic
  given its market-integrity sensitivity.
- **KYC/AML** required before an account can submit its first order, with
  enhanced due diligence for liquidity provider onboarding given the
  larger capital and market-access footprint.
- **Immutable audit trails**: order, trade, halt, and surveillance-decision
  audit entries are append-only, matching §20.
- **MFA is mandatory, not optional**, for every human user role submitting
  orders, managing markets, or dispositioning surveillance cases, enforced
  at the deployment's security policy level per
  [authentication-authorization.md](../../security/authentication-authorization.md).
- Custody balance segregation (customer assets never commingled with
  operator assets) is enforced at the data-model level in
  `custody_balances`, not merely by operating convention.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated matching-engine determinism test suite (identical input order
sequence always produces identical match output) and a surveillance-rule
regression suite validated against known manipulation-pattern fixtures.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md). The
matching engine and order-entry gateway deploy with a zero-downtime
requirement during each market's trading session; matching-engine code
changes require a documented market-integrity review in addition to the
standard PR review per
[pr-standards.md](../../development/pr-standards.md).

## 30. Acceptance Criteria

- An order submitted through the API is acknowledged, matched (if
  marketable), and produces correct execution reports to both counterparties
  with fees assessed per the active fee schedule.
- A trading halt stops new order acceptance for the affected symbol while
  preserving resting orders, and resume correctly reopens the book per the
  configured policy.
- A surveillance rule fixture representing a known manipulation pattern
  produces a case in the surveillance queue without manual intervention.
- Custody balances reconcile to zero variance against the settlement ledger
  at the end of each reconciliation cycle, or raise an exception if they do
  not.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md)
and [security-checklist.md](../../checklists/security-checklist.md).
ZodiXchange additionally requires sign-off from a compliance stakeholder
that market surveillance rules, halt authorization, and custody segregation
have been validated against the operator's actual licensing/regulatory
obligations before go-live.

## 32. Future Roadmap

- Expanded derivatives support: perpetual and dated futures contract types
  with funding-rate mechanics.
- Cross-margining across spot and derivatives positions for the same
  account.
- Smart-order-routing to external venues for symbols where the venue's own
  liquidity is thin.

## 33. Known Risks

- Matching engine latency under load is the single largest competitive and
  reliability risk for an exchange operator — mitigated by the dedicated
  low-latency service architecture in §11, but requires ongoing
  capacity/load testing as symbol count and volume grow.
- Custody/settlement model divergence: traditional custodial settlement and
  on-chain settlement have materially different failure modes; the
  `CustodyLedgerContract` abstracts the interface but not the underlying
  operational risk profile of each model.

## 34. Future Improvements

- Configurable auction mechanics (opening/closing auction algorithms)
  beyond the current default.
- Expanded LP rebate program with tiered volume-based incentive structures.

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
  functional slice (matching/order-book-relevant vs. custody/wallet-relevant
  vs. something else entirely) it actually implements before this spec
  treats it as a validated feature/UX reference for any specific
  ZodiXchange module beyond the general observation that it demonstrates a
  real Laravel-based payment/crypto-adjacent commercial script exists. Do
  not assume equivalence to Bicrypto's feature set until that follow-up
  audit runs.

## Roadmap (spec depth)

This spec's Architecture and Core Data Model sections were revised to
reflect the corrected standalone, self-hosted, single-tenant deployment
model — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md)
and [base-codebase-strategy.md](../../architecture/base-codebase-strategy.md).
This spec was further revised to correct a possible misreading of the
`dash`/Bicrypto and `web3chainlink` reference codebases as something
ZodiXchange could be cloned or ported from: neither is a base ZodiXchange
builds on. ZodiXchange remains a fresh Laravel build on the sanitized
qfsfountains base per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md)
and the
[genericization checklist](../../architecture/product-genericization-checklist.md);
`dash`/Bicrypto and `web3chainlink` are feature/UX references only (§11.1),
and the dual external-liquidity-API/internal-matching-engine architecture
(§11.2) is now the documented default for ZodiXchange's order execution.
This spec is Foundation-depth. Queued for Deep-depth expansion: a full ER
diagram and migration set for the order book/trade/settlement schema
(companion `DATA_MODEL.md`), a complete endpoint catalog including the LP/
FIX-style gateway (companion `API_REFERENCE.md`), and a full surveillance
rule catalog beyond the initial spoofing/layering/wash-trading/
quote-stuffing set. Changes follow [CONTRIBUTING.md](../../../CONTRIBUTING.md).

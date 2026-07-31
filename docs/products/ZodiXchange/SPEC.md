# ZodiXchange — Product Specification

> Status: **Foundation**. Vision, market, personas, architecture, modules,
> core data model, key workflows, integrations, permissions model, and
> acceptance criteria are complete and implementation-usable. Deep artifacts
> (full ER diagrams, exhaustive endpoint listings, full report catalogs) are
> queued — see [Roadmap (spec depth)](#roadmap-spec-depth) at the bottom of
> this document. See [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md) for
> spec status definitions.

Built on [ZodiCore](../ZodiCore/SPEC.md) — ZodiXchange does not reimplement
identity, tenancy, billing, notifications, RBAC, plugins, or audit logging;
it consumes those services and adds exchange-domain modules on top.

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
exchange operator a coherent venue platform — matching, custody
integration, LP connectivity, and surveillance — built on ZodiCore's
identity and audit backbone, so the operator's engineering effort goes into
market design and liquidity, not plumbing.

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
| Matching engine infrastructure | Nasdaq Matching Engine (licensed), B2C2/DXtrade-class venue tech, Talos | Ships with tenant/RBAC/audit already built via ZodiCore, faster to stand up a compliant venue |
| Digital asset exchange platform | Coinbase Prime infrastructure tier, ErisX/CoinFLEX-class venue tech | Custody integration and surveillance share one audit trail with matching, not bolted-on separately |
| Market surveillance | Nasdaq SMARTS, Eventus Systems | Surveillance rules operate directly on the order/trade ledger of record, not a delayed downstream feed |
| Liquidity provider connectivity | FIX-based prime brokerage gateways, LMAX Digital LP APIs | Standardized LP API contract shared across every Zodize capital-markets product |
| Clearing/settlement | DTCC-class clearing infrastructure (traditional), on-chain settlement rails (digital assets) | Settlement abstraction lets an operator support both custodial and on-chain settlement models |

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
- **Zodize Support/Ops** — as defined in [ZodiCore §6](../ZodiCore/SPEC.md#6-personas).

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
   tenant's configured reopening auction or continuous-resume policy.
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
- Maker/taker fee schedule: configurable per tenant, per symbol, and per
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

ZodiXchange is a Laravel application consuming ZodiCore's identity,
tenancy, billing, notification, RBAC, plugin, and audit packages exactly as
described in [ZodiCore §11](../ZodiCore/SPEC.md#11-architecture), with one
deliberate exception: the matching engine itself runs as a dedicated,
in-memory, low-latency service per market/symbol (outside the request-
response Laravel app) that publishes order-book and execution events onto
the platform's event bus (per
[caching-queues-events.md](../../architecture/caching-queues-events.md))
for the Laravel app to persist, audit, and expose via API/WebSocket. This
keeps ZodiCore's identity/audit/RBAC guarantees intact for every account
and order action while not forcing matching-engine-critical-path latency
through a general-purpose web framework request cycle. Custody/wallet
integration and settlement run behind a `CustodyLedgerContract` supporting
both custodial and on-chain settlement models.

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
(see [Roadmap (spec depth)](#roadmap-spec-depth)).

| Entity | Key columns |
|---|---|
| `markets` | id, tenant_id, symbol, asset_class, tick_size, lot_size, status |
| `orders` | id, account_id, market_id, side, order_type, quantity, price, hidden_quantity, status |
| `trades` | id, market_id, buy_order_id, sell_order_id, price, quantity, executed_at |
| `order_book_snapshots` | id, market_id, sequence_number, captured_at, depth_json |
| `fee_schedules` | id, tenant_id, market_id, tier, maker_bps, taker_bps, effective_at |
| `custody_balances` | id, account_id, asset, available_balance, held_balance, updated_at |
| `settlements` | id, trade_id, settlement_model, status, settled_at |
| `liquidity_providers` | id, account_id, status, quoting_permissions, rebate_tier, onboarded_at |
| `surveillance_cases` | id, rule_triggered, related_order_ids, status, disposition, reviewed_by |
| `trading_halts` | id, market_id, reason, initiated_by, initiated_at, resumed_at |
| `market_makers_quotes` | id, liquidity_provider_id, market_id, bid_price, ask_price, updated_at |
| `symbol_reference_data` | id, market_id, description, underlying_asset, contract_spec_json |

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

Domain events registered on ZodiCore's event bus (see
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

Extends [ZodiCore's default roles](../../security/rbac-permissions.md#default-system-roles)
with exchange-specific roles: `Market Operations Manager`,
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
disposition writes an immutable audit entry via ZodiCore's audit log
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
- **On-chain settlement**: blockchain node/wallet infrastructure
  integration for venues settling on-chain, abstracted so the settlement
  module is chain-agnostic where feasible.
- **Market data distribution**: outbound feed integration for tenants that
  redistribute their own market data to third parties (consolidated tape-
  style distribution).
- **KYC/AML providers**: same identity-verification integration category
  as [ZodiBank §22](../ZodiBank/SPEC.md#22-integrations) for account and LP
  onboarding.
- **Clearing houses**: optional integration for venues that clear through
  a third-party central counterparty rather than settling bilaterally.

## 23. AI Features

- Market surveillance pattern detection: the rule-based surveillance engine
  in §9 is augmented with anomaly-scoring that layers on top of ZodiCore's
  audit-log anomaly detection ([ZodiCore §23](../ZodiCore/SPEC.md#23-ai-features)),
  surfacing lower-confidence patterns for human review rather than
  auto-dispositioning.
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
  context as its API equivalent, per
  [ZodiCore §24](../ZodiCore/SPEC.md#24-automation-scheduled-jobs-cron-jobs-cli-commands).

## 25. Seed/Demo Data

`XchangeDemoSeeder` provisions a demo tenant with several configured
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
  and surveillance modules with the same rigor as the ZodiCore platform
  baseline, including change control specifically for matching-engine
  logic given its market-integrity sensitivity.
- **KYC/AML** required before an account can submit its first order, with
  enhanced due diligence for liquidity provider onboarding given the
  larger capital and market-access footprint.
- **Immutable audit trails**: order, trade, halt, and surveillance-decision
  audit entries are append-only, matching §20.
- **MFA is mandatory, not optional**, for every human user role submitting
  orders, managing markets, or dispositioning surveillance cases, enforced
  at the tenant policy level per
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

## Roadmap (spec depth)

This spec is Foundation-depth. Queued for Deep-depth expansion: a full ER
diagram and migration set for the order book/trade/settlement schema
(companion `DATA_MODEL.md`), a complete endpoint catalog including the LP/
FIX-style gateway (companion `API_REFERENCE.md`) matching
[ZodiCore's structure](../ZodiCore/SPEC.md), and a full surveillance rule
catalog beyond the initial spoofing/layering/wash-trading/quote-stuffing
set. Changes follow [CONTRIBUTING.md](../../../CONTRIBUTING.md).

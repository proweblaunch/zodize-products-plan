# ZodiPOS — Product Specification

> Status: **Foundation**. Vision through acceptance criteria are complete and
> implementation-usable; exhaustive ER diagrams and a full endpoint catalog
> are queued — see [Roadmap (spec depth)](#roadmap-spec-depth) and
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

## 1. Vision

ZodiPOS is the point-of-sale system for retail and hospitality operators who
need a register that works when the internet doesn't, reconciles a till to
the cent, and feeds every sale straight into the same inventory and
accounting truth the rest of the business runs on — not a standalone tablet
app that happens to also print receipts.

## 2. Purpose

A POS outage during a Friday dinner rush or a Saturday retail peak is not an
inconvenience, it's lost revenue and an angry line. ZodiPOS exists because
most POS software either handles offline operation poorly or handles
enterprise inventory/accounting integration poorly — rarely both. It treats
"the internet is down" as a normal operating condition, not an edge case.

## 3. Target Market

Multi-location retail chains, quick-service and full-service restaurants,
cafes, and hospitality operators with 1–500 register locations who need
enterprise-grade reliability, inventory accuracy, and reporting rollup
across locations without per-location software silos.

## 4. Industries

Retail (grocery, convenience, specialty), hospitality/food service
(quick-service, full-service, cafes, bars), and service-counter retail
(salons, quick-repair shops) with a shared checkout/till model.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Restaurant POS | Toast, Square for Restaurants | Built from the same base codebase and inventory engine as [ZodiCommerce](../ZodiCommerce/SPEC.md), so a merchant running both never maintains a separate online/in-store inventory system |
| Retail POS | Square, Lightspeed Retail | Native multi-location rollup reporting via the product's own company/branch model, not a per-store license silo |
| Offline-first checkout | Square (offline mode), Clover | Full transaction queue with conflict-safe sync, not a degraded-feature offline mode |
| Till/cash management | Toast cash management, Loyverse | Till reconciliation ledger shares the same audit/accounting model as [ZodiBusiness](../ZodiBusiness/SPEC.md) |
| Hardware ecosystem | Clover hardware ecosystem | Vendor-neutral hardware abstraction layer (§22) instead of locking merchants into proprietary terminals |

## 6. Personas

- **Cashier/Server** — operates the register: rings items, applies
  discounts, takes payment, prints/emails receipts.
- **Shift Supervisor** — opens/closes shifts, approves till overrides
  (voids, price overrides), resolves reconciliation discrepancies.
- **Store/Restaurant Manager** — configures the local menu/catalog subset,
  reviews daily sales and reconciliation reports.
- **Multi-Location Operator** — the owner/regional manager viewing rollup
  performance across all registers and locations.
- **Customer** — the person being checked out; may be a loyalty-program
  member with a linked account.

## 7. User Journeys

1. **Shift open to till reconciliation**: Cashier clocks in and opens their
   till with a counted starting cash amount → processes transactions through
   the shift → at shift end counts the drawer → the system compares counted
   cash to expected cash (starting float + cash sales − cash paid out) →
   any variance is logged and, above a threshold, requires Shift Supervisor
   sign-off before the till closes.
2. **Split-tender checkout**: Customer's order totals $42.50 → pays $20 cash
   and the remainder on a card → Cashier records the cash tender, then runs
   the card through the connected reader → both tenders post against the
   same transaction → a single itemized receipt reflects both payment
   methods → the sale posts to inventory and the accounting feed as one
   transaction.
3. **Offline sale during a connectivity outage**: internet drops mid-shift →
   ZodiPOS detects the outage and switches to offline mode, visibly
   indicated on screen → Cashier continues ringing sales against a locally
   cached catalog and price list → card payments queue via the reader's
   offline-authorization capability where supported, or fall back to a
   configured offline-tender policy → once connectivity resumes, every
   queued transaction syncs in order, inventory decrements apply, and any
   sync conflict (e.g. an item deactivated centrally mid-outage) is flagged
   for manager review rather than silently dropped or double-applied.
4. **Barcode-scanned retail sale**: Customer brings items to the register →
   Cashier scans each barcode → price and stock lookup resolve from the
   locally cached catalog for scan speed → a promotion configured in
   [ZodiCommerce](../ZodiCommerce/SPEC.md)'s shared discount rules engine
   auto-applies at the register → sale completes, inventory decrements
   against the same stock ledger used by the online storefront.
5. **End-of-day close and rollup**: Store Manager closes the day → the
   system reconciles every till against expected cash, aggregates card
   settlement batches, and produces a daily sales/close report → for a
   multi-location operator, the same report rolls up across every location
   into a single dashboard visible the next morning.

## 8. Business Goals

- Eliminate lost sales during connectivity outages via true offline
  operation, not a "please check your connection" failure screen.
- Reduce till-reconciliation variance and the manual investigation time it
  currently costs store managers.
- Give multi-location operators same-day, cross-location sales visibility
  without waiting on a nightly batch export from each store's separate
  system.

## 9. Functional Requirements

- Register/checkout flow: item lookup (scan/search), quantity/discount
  entry, tax calculation, order hold/recall, receipt printing/emailing.
- Offline-first operation: locally cached catalog/pricing, local
  transaction queue, automatic sync-on-reconnect with conflict detection.
- Cash drawer/till reconciliation: shift open/close, expected-vs-counted
  cash comparison, variance logging, paid-in/paid-out tracking.
- Split tender payments: any combination of cash, card, gift
  card/store-credit, and configurable custom tender types on one
  transaction.
- Receipt printing: thermal printer output plus email/SMS digital receipt
  option, configurable receipt template.
- Barcode scanning: camera-based and dedicated hardware scanner support,
  SKU/UPC lookup against the shared catalog.
- Shift/employee clock-in: time clock tied to the employee's own admin-panel
  user account (the base engine's `admin` guard, per
  [admin-template.md](../../templates/admin-template.md)), shift-level sales
  attribution, break tracking.
- Hardware integration as an architecture concern (§11, §22): card readers,
  receipt printers, barcode scanners, cash drawers, customer-facing displays.
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  saved register configurations per location, mass void/refund actions with
  approval gating, custom fields on transactions (e.g. table number, order
  type), full audit trail per transaction and till session, soft
  delete/void with reversal rather than deletion of financial records.

## 10. Non-Functional Requirements

Inherits the baseline in
[performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md).
ZodiPOS-specific additions:

- Item scan-to-cart response time: under 150ms from scan to line-item
  display, since register speed directly affects checkout-line throughput.
- Register must remain fully operational (ring sales, accept card payment
  where the reader supports offline auth, print receipts) for a minimum of
  24 hours of continuous connectivity loss without data loss.
- Sync-on-reconnect must process a full shift's queued offline transactions
  (target: 500 transactions) within 60 seconds of connectivity restoration.

## 11. Architecture

ZodiPOS — the third product in the build order
([ROADMAP.md](../../../ROADMAP.md)), chosen to prove the offline-first and
hardware-integration patterns on top of the same pipeline validated by
[ZodiBusiness](../ZodiBusiness/SPEC.md) and
[ZodiCommerce](../ZodiCommerce/SPEC.md) — is built by cloning the sanitized
base codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md):
the banking-specific `loans`/`dps`/`fdr`/`branches`/`branch_staff`/
`other_banks`/`beneficiaries`/`airtime` tables are stripped, and the
`branch_staff` guard is dropped by default. ZodiPOS inherits the base
engine's wallet/ledger (used for gift-card/store-credit tenders), payment
gateways (§22), RBAC/auth, i18n, and admin configuration surface unmodified
— see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md) —
and layers its own Register, Offline Sync, Payments, Till Management,
Employees, Receipts, and Hardware modules (§13) on top. There is no shared
tenant boundary and no ZodiCore platform dependency: each ZodiPOS deployment
is one merchant's standalone, self-hosted instance, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).

ZodiPOS and [ZodiCommerce](../ZodiCommerce/SPEC.md) are independently
deployed products, each cloned from the same base codebase and sharing the
same catalog/inventory-ledger *design*, but not a runtime or database — a
merchant running both installs them as two separate codebases on their own
hosting and connects them via an API-based inventory-sync integration (the
same class of integration documented for ZodiCommerce's channel connectors,
§22 of [ZodiCommerce's spec](../ZodiCommerce/SPEC.md#22-integrations)) so an
item sold in-store and an item sold online reconcile against one
merchant-controlled stock truth without either product depending on the
other being reachable at runtime. The register client is a local-first PWA,
built as part of this same product's codebase: it holds a synced copy of the
active catalog, price list, and tax rules in local storage/IndexedDB, and
every transaction is written locally first, then queued for sync via an
idempotent transaction API keyed on a client-generated UUID — the same
offline-sync pattern used by [ZodiTrack](../ZodiTrack/SPEC.md)'s own,
independently deployed mobile scanning client. A local lightweight sync
daemon on multi-register locations (or the register client itself for
single-register sites) manages the queue and conflict resolution, surfacing
unresolved conflicts to the Store Manager rather than auto-resolving
silently. Multi-location merchants use the `company_id`/`location_id`
scoping in §14, per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping) —
this is scoping within one deployment, not tenancy.

## 12. Technology

Laravel (PHP) per the base codebase's stack (Laravel 11, PHP ^8.3, Vite 5) —
see
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md) —
following
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md);
MySQL/MariaDB + Redis (where the buyer's hosting supports it, with a file/DB
cache fallback) per
[database-standards.md](../../development/database-standards.md); a PWA
register client with IndexedDB-backed local storage and a service worker for
offline asset/catalog caching; a hardware abstraction layer (`HardwareDriverContract`)
so card readers, printers, and scanners plug in through a common interface
rather than vendor-specific code paths scattered through the checkout flow.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Register | Checkout, Item Lookup, Discounts, Order Hold/Recall |
| Offline Sync | Local Transaction Queue, Conflict Resolution, Catalog Cache |
| Payments | Split Tender, Card Reader Integration, Gift Card/Store Credit |
| Till Management | Shift Open/Close, Cash Counting, Paid-In/Paid-Out, Variance Resolution |
| Employees | Clock-In/Out, Shift Attribution, Break Tracking |
| Receipts | Print Templates, Email/SMS Digital Receipts |
| Hardware | Card Reader Drivers, Printer Drivers, Scanner Drivers, Cash Drawer Control |
| Reporting | Daily Close Reports, Multi-Location Rollup, Employee Sales Reports |

## 14. Core Data Model

| Entity | Key columns |
|---|---|
| `companies` | id, name, default_currency, is_active |
| `locations` | id, company_id, name, address, is_active |
| `registers` | id, location_id, name, hardware_profile_id |
| `till_sessions` | id, register_id, opened_by, opening_float, closed_by, closing_counted, variance |
| `pos_transactions` | id, location_id, register_id, till_session_id, client_uuid, status, total, synced_at |
| `pos_transaction_items` | id, transaction_id, variant_id, quantity, unit_price, discount_amount |
| `pos_tenders` | id, transaction_id, tender_type (cash/card/gift_card/custom), amount, gateway_reference |
| `till_movements` | id, till_session_id, type (paid_in/paid_out), amount, reason, recorded_by |
| `employee_shifts` | id, user_id, clocked_in_at, clocked_out_at, register_id |
| `hardware_profiles` | id, location_id, printer_config, reader_config, scanner_config |
| `sync_conflicts` | id, transaction_id, conflict_type, detected_at, resolved_by, resolution |
| `gift_cards` | id, code, balance, issued_at, last_used_at |
| `voids_refunds` | id, transaction_id, type, amount, approved_by, reason |

`companies` and `locations` model the multi-brand/multi-location scoping a
merchant may need within its one deployment (§11), per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping);
a single-location merchant has exactly one seeded `companies` row and one
`locations` row.

## 15. Key API Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/pos/transactions` | Submit a completed transaction (online or synced-from-offline) |
| POST | `/api/v1/pos/transactions/batch-sync` | Sync a queued batch of offline transactions |
| GET | `/api/v1/pos/catalog-snapshot` | Fetch the current catalog/price snapshot for local caching |
| POST | `/api/v1/pos/till-sessions/open` | Open a till with a starting float |
| POST | `/api/v1/pos/till-sessions/{id}/close` | Close a till with counted cash, compute variance |
| POST | `/api/v1/pos/till-sessions/{id}/movements` | Record a paid-in/paid-out |
| POST | `/api/v1/pos/transactions/{id}/void` | Void a transaction (requires approval per §19) |
| POST | `/api/v1/pos/transactions/{id}/refund` | Refund a completed transaction |
| POST | `/api/v1/pos/employees/clock-in` | Clock in an employee to a register/shift |
| POST | `/api/v1/pos/employees/clock-out` | Clock out an employee |
| GET | `/api/v1/pos/registers/{id}/status` | Register health/connectivity/sync status |
| GET | `/api/v1/pos/sync-conflicts` | List unresolved sync conflicts |
| POST | `/api/v1/pos/sync-conflicts/{id}/resolve` | Resolve a flagged sync conflict |
| POST | `/api/v1/pos/gift-cards/redeem` | Redeem/apply a gift card tender |
| GET | `/api/v1/pos/reports/daily-close` | Daily close report for a location |
| GET | `/api/v1/pos/reports/rollup` | Multi-location sales rollup |

## 16. Events

`transaction.completed`, `transaction.synced`, `transaction.sync_conflict`,
`till.opened`, `till.closed`, `till.variance_flagged`,
`transaction.voided`, `transaction.refunded`, `employee.clocked_in`,
`employee.clocked_out`, `register.went_offline`, `register.reconnected`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `till.variance_flagged` (above threshold) | ✔ (Shift Supervisor) | ✔ | — | — |
| `transaction.sync_conflict` | ✔ (Store Manager) | ✔ | — | — |
| `register.went_offline` (extended outage) | ✔ (Store Manager) | ✔ | ✔ | — |
| `transaction.completed` (customer digital receipt) | — | ✔ (opt-in) | ✔ (opt-in) | — |
| `transaction.voided` | ✔ (Store Manager) | — | — | — |
| Daily close report ready | ✔ (Multi-Location Operator) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Built on the base engine's inherited `Role`/`Permission` RBAC (not Spatie),
per
[admin-template.md](../../templates/admin-template.md#roles--permissions-inherited-not-spatie).
ZodiPOS's `DemoSeeder` ships its own default admin roles — Multi-Location
Operator, Store/Restaurant Manager, Shift Supervisor, and Cashier/Server —
each granted a subset of ZodiPOS's product-specific permissions:
`register.operate`, `till.open`, `till.close`, `till.approve_variance`,
`transactions.void`, `transactions.refund`, `price.override`,
`sync_conflicts.resolve`. `transactions.void` and `price.override` require
Shift Supervisor or above by default; a Cashier can request either action
but it routes through the approval chain in §19. A merchant can create
additional custom roles and reassign any permission from the admin panel
with no code change, per
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md#roles--permissions).

## 19. Workflows & Approval Chains

- **Void/refund approval**: a Cashier-initiated void or refund above an
  admin-configured amount requires Shift Supervisor approval, captured as
  a PIN/badge confirmation at the register before the transaction reverses.
- **Till variance resolution**: a till closing with a cash variance beyond
  the configured tolerance cannot fully close without Shift Supervisor
  sign-off explaining the discrepancy.
- **Price override**: manually overriding an item's price at the register
  requires Shift Supervisor approval and is logged with the original and
  overridden price.
- **Sync conflict resolution**: a flagged offline-sync conflict (e.g. an
  item sold offline that was deactivated centrally during the outage)
  blocks final settlement of that transaction until a Store Manager
  resolves it.

## 20. Audit Logs

Every transaction, void, refund, price override, till open/close, paid-in/
paid-out, and sync conflict resolution is recorded to this deployment's own
audit log with actor, register, till session, and before/after amounts, per
[audit-logging.md](../../security/audit-logging.md). Because register
actions often happen under a shared physical device, every action is tied
to the authenticated employee session (PIN or badge login), never to the
device alone.

## 21. Reports & Analytics & Dashboards

Daily close/reconciliation report per register and rolled up per location,
multi-location sales dashboard (real-time and historical), employee sales
performance, void/refund rate by employee (an operational-integrity signal),
till variance trend over time, and item/category sales velocity feeding
back into [ZodiCommerce](../ZodiCommerce/SPEC.md) inventory planning.
Dashboard-builder and scheduled-report capability per
[dashboard-standards.md](../../standards/dashboard-standards.md).

## 22. Integrations

- **Card readers**: Stripe Terminal, Verifone, Ingenico via the
  `HardwareDriverContract` abstraction, supporting offline-capable
  authorization where the reader/gateway combination allows it.
- **Receipt printers**: Epson, Star Micronics thermal printer drivers.
- **Barcode scanners**: Zebra, Honeywell handheld/fixed scanners, plus
  camera-based scanning as a hardware-free fallback.
- **Cash drawers**: standard RJ11/USB-triggered drawer integration via the
  connected receipt printer or a dedicated controller.
- **Payment gateways**: the inherited gateway catalog documented in
  [payment-gateways.md](../../standards/payment-gateways.md) — Stripe and
  Authorize.Net for card processing, Flutterwave and Paystack for merchants
  in Zodize's primary African market, plus Mollie/Razorpay for
  region-specific merchants — accessed through the same `HardwareDriverContract`-adjacent
  card-reader integration above, extended with offline-authorization support
  where the reader/gateway combination provides it, and the native
  manual/offline gateway as an always-available fallback tender.

## 23. AI Features

- Anomaly detection on void/refund and discount patterns per employee,
  surfaced to Store Managers as a loss-prevention signal, never an automatic
  penalty.
- Peak-hour staffing suggestion based on historical transaction-volume
  patterns per location.
- Till-variance root-cause suggestion (e.g. correlating a variance with a
  specific till movement or a known cash-handling training gap) surfaced to
  Shift Supervisors during reconciliation.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: catalog/price snapshot refresh push to registers, nightly
  multi-location rollup aggregation, stale-sync-conflict escalation
  reminder, card settlement batch reconciliation against the gateway.
- CLI commands: `pos:push-catalog-snapshot {location_id}`,
  `pos:reconcile-settlement {date}`, `pos:force-resync {register_id}`,
  `pos:export-daily-close {location_id} {date}`.

## 25. Seed/Demo Data

`DemoSeeder` provisions a demo deployment with 3 locations (one retail, one
quick-service restaurant, one cafe), each with 2 registers, 30 days of
transaction history including split-tender sales, at least one voided
transaction, at least one till with a resolved variance, and one simulated
historical sync conflict with its resolution recorded, per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: the multi-location rollup dashboard must aggregate
100+ locations' daily totals in under 3 seconds, and catalog snapshot sync
to a register completes in under 10 seconds for a 5,000-SKU catalog.

## 27. Security Requirements

Full baseline from
[security-standards.md](../../security/security-standards.md) applies.
Card data never persists on the register client or ZodiPOS's own database —
tokenization happens at the reader/gateway per
[data-protection-privacy.md](../../security/data-protection-privacy.md), and
the register client's local storage holds only catalog/price/transaction
metadata, never raw card data, even transiently. Employee register access
uses short-lived PIN/badge sessions distinct from the product's own full
admin-panel login, scoped to register operation only.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated offline-sync test suite covering extended-outage queuing,
out-of-order sync delivery, duplicate-submission idempotency, and conflict
detection when catalog data changes centrally during an outage.

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md). The
register PWA client is versioned independently from the backend API with
backward-compatible sync endpoint contracts, since registers in the field
may run an older client version for days after a backend deploy while
mid-outage.

## 30. Acceptance Criteria

- A register can complete a full sale (scan, split tender, receipt) with no
  network connectivity, and that sale syncs correctly once connectivity
  returns.
- A till's expected-vs-counted cash variance is computed correctly across a
  shift containing sales, refunds, and paid-in/paid-out movements.
- A sync conflict (e.g. item deactivated centrally during an offline sale)
  is detected and blocks silent auto-resolution, routing to a Store
  Manager.
- A void or refund above the configured threshold cannot complete without
  Shift Supervisor approval, and is fully audit-logged.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md);
ZodiPOS additionally requires sign-off that the offline-sync test suite
(§28) passes against a 24-hour simulated outage scenario, and that every
supported hardware driver has passed an integration test against physical
or vendor-certified simulator hardware before go-live.

## 32. Future Roadmap

- Kitchen display system (KDS) integration for full-service/QSR order
  routing.
- Table/seat management for full-service hospitality.
- Loyalty program integration tying register transactions to
  [ZodiReach](../ZodiReach/SPEC.md) customer segmentation.

## 33. Known Risks

- Offline-sync correctness is the module's highest-risk surface, identical
  in kind to [ZodiTrack](../ZodiTrack/SPEC.md)'s scan-sync risk: a
  double-applied or lost transaction directly corrupts both the sales
  ledger and inventory — mitigated by idempotent client UUIDs and the
  dedicated test suite (§28), but this warrants continued scrutiny as
  register count scales.
- Hardware fragmentation: supporting many card reader/printer/scanner
  vendors through one abstraction layer risks driver-specific edge cases —
  mitigated by the `HardwareDriverContract` interface and required
  integration testing (§31), but new hardware vendor requests should budget
  real integration-test time, not just a config flag.

## 34. Future Improvements

- Predictive till-float suggestions based on historical cash-sale volume
  per location/day-of-week.
- Configurable partial-refund line-item selection (currently refund flows
  assume whole-transaction or manually specified line items).

## Roadmap (spec depth)

This spec is Foundation-depth. Its Architecture and Core Data Model sections
were revised to the standalone, self-hosted, single-tenant base-codebase
model described in
[architecture/overview.md](../../architecture/overview.md) and
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).
Queued for Deep-depth expansion: a full ER diagram covering hardware-profile
configuration and settlement-batch reconciliation tables, the complete
endpoint catalog (hardware pairing, receipt template management endpoints),
and a dedicated `DATA_MODEL.md`/`API_REFERENCE.md` pair matching
[ZodiCore](../ZodiCore/SPEC.md)'s companion-document structure.

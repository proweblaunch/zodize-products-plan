# ZodiCommerce — Product Specification

> Status: **Foundation**. Vision through acceptance criteria are complete and
> implementation-usable; exhaustive ER diagrams and a full endpoint catalog
> are queued — see [Roadmap (spec depth)](#roadmap-spec-depth) and
> [PRODUCT_CATALOG.md](../../../PRODUCT_CATALOG.md).

## 1. Vision

ZodiCommerce is the enterprise storefront and order management system for
merchants who have outgrown templated e-commerce platforms and need a
storefront, catalog, and fulfillment engine that behaves like the rest of
their operation — audited, permissioned, multi-brand, and integrated with
real inventory and accounting rather than a bolted-on app ecosystem.

## 2. Purpose

Mid-market and enterprise retailers run their storefront on one platform,
their marketplace listings on another, their inventory truth in a
spreadsheet, and their returns process by email. ZodiCommerce exists to be
the single order-management system of record across every channel a
merchant sells through — web storefront, marketplaces, and (via
[ZodiPOS](../ZodiPOS/SPEC.md)) physical retail — so that "how many units of
SKU X do we actually have" always has one correct answer.

## 3. Target Market

Mid-market to enterprise retailers and B2B distributors doing $2M–$500M in
annual revenue across 1–50 sales channels, who have outgrown Shopify
Plus/BigCommerce Enterprise-class platforms' inventory and order-orchestration
limits, or who need tighter integration with an ERP than a marketplace-app
ecosystem provides.

## 4. Industries

Retail, consumer packaged goods (CPG), fashion/apparel, B2B distribution,
and direct-to-consumer (DTC) brands selling across web, marketplace, and
retail channels.

## 5. Competitor Analysis

| Capability | Comparable to | Zodize differentiation |
|---|---|---|
| Enterprise storefront platform | Shopify Plus, BigCommerce Enterprise | Self-hosted, single-merchant deployment with the base engine's own RBAC/audit built in, instead of a third-party app marketplace bolted onto checkout |
| Order management / channel aggregation | Salesforce Order Management, Fluent Commerce | Inventory sync and order routing built into the same data model as accounting and POS, not a middleware layer |
| Marketplace channel management | ChannelAdvisor, CommerceHub | First-party integration category (§20) rather than a separate paid tool |
| Returns/RMA | Loop Returns, Narvar | RMA lifecycle lives in the same order record and audit trail as the original sale |
| Promotions engine | Shopify Scripts / Bold Discounts (app-based) | Native rule engine with the same approval-chain model as every other Zodize product |

## 6. Personas

- **Merchandiser** — manages the product catalog, variants, pricing, and
  promotions.
- **Fulfillment Manager** — owns inventory sync, shipping rate configuration,
  and order routing across warehouses/3PLs.
- **Customer Service Rep** — handles order lookups, cancellations, and RMA
  processing.
- **Store Ops / Channel Manager** — manages marketplace listings and channel
  health (web, Amazon, POS-driven retail).
- **Customer** — end shopper browsing, purchasing, and managing their own
  account and orders on the storefront.
- **Finance/Accounting** — reconciles orders, refunds, and payouts against
  [ZodiBusiness](../ZodiBusiness/SPEC.md) or an external accounting system.

## 7. User Journeys

1. **Product launch**: Merchandiser creates a product with variants (size,
   color) → sets channel-specific pricing and inventory allocation → publishes
   to the web storefront and a connected marketplace simultaneously → stock
   levels sync bidirectionally so an oversell on one channel is impossible.
2. **Guest checkout to fulfilled order**: Customer adds items to cart →
   applies a promotion code validated by the discount rules engine → checks
   out with a saved or guest payment method → order enters `pending` →
   inventory is reserved → Fulfillment Manager batches it for pick/pack →
   shipping label purchased via the shipping-rate integration → order moves
   to `shipped` with tracking synced to the customer's order history.
3. **Multi-channel order aggregation**: An order placed on a marketplace
   channel is pulled in via the channel connector → normalized into the same
   `orders` table as web orders → inventory decrement applies against the
   same shared stock pool → Fulfillment Manager sees one unified fulfillment
   queue regardless of channel origin.
4. **Return and refund**: Customer initiates a return from their account →
   RMA created with reason code → Customer Service Rep approves and issues a
   prepaid label → warehouse receives and inspects the item → condition
   determines restock vs. write-off → refund issued to original payment
   method → order status moves to `returned` with the RMA linked to the
   original order for audit.
5. **Flash-sale promotion**: Merchandiser configures a time-boxed percentage
   discount rule scoped to a product collection → rule engine validates
   against stacking policy (does it combine with existing codes?) → sale goes
   live at the scheduled time via a scheduled job → analytics dashboard shows
   real-time redemption and inventory drawdown so Merchandiser can end the
   sale early if stock runs low.

## 8. Business Goals

- Eliminate oversell/underfulfillment incidents caused by disconnected
  channel inventory by keeping one authoritative stock ledger.
- Reduce average return-to-refund cycle time versus email-based RMA handling.
- Give Finance a reconciled, audit-ready order/refund trail without manual
  spreadsheet reconciliation across channels.

## 9. Functional Requirements

- Product catalog with variants (size/color/material/etc.), bundles, and
  channel-specific pricing and visibility.
- Real-time inventory sync across warehouses and sales channels with
  configurable allocation rules (e.g. reserve buffer stock for retail).
- Cart and checkout: guest and account checkout, saved payment methods,
  address book, tax calculation, and promotion code redemption.
- Order lifecycle state machine: `pending` → `payment_confirmed` →
  `fulfilling` → `shipped` → `delivered`, with `canceled` and `returned` as
  branch states reachable from the appropriate points.
- Shipping rate integration: live carrier rates at checkout, label purchase,
  and tracking number sync back to the order.
- Promotions/discount rules engine: percentage/fixed/BOGO/free-shipping
  rules, stacking policy, usage limits per customer and globally, scheduled
  activation windows.
- Customer accounts: order history, saved addresses/payment methods, RMA
  self-service initiation.
- Multi-channel order aggregation: normalize orders from web, POS
  (ZodiPOS-originated in-store sales), and connected marketplaces into one
  order table and fulfillment queue.
- Returns/RMA: reason codes, approval step, label generation, receiving
  inspection, restock-vs-writeoff disposition, refund issuance.
- Second-layer baseline per
  [product-philosophy.md](../../development/product-philosophy.md#second-layer-feature-catalog):
  saved filters on the order list, mass actions (bulk-fulfill, bulk-cancel),
  CSV import/export for catalog and orders, custom fields on product and
  order, API tokens/webhooks for order events, full audit history per order,
  soft delete + restore for catalog entities.

## 10. Non-Functional Requirements

Inherits the baseline in
[performance-standards.md](../../quality/performance-standards.md) and
[security-standards.md](../../security/security-standards.md).
ZodiCommerce-specific additions:

- Storefront product/category pages: p95 < 200ms server-rendered response
  time; checkout API calls: p95 < 300ms.
- Inventory sync propagation across channels: under 5 seconds from a stock
  change to the change being reflected on every connected channel, to bound
  oversell risk during high-traffic sales.
- Storefront must remain read-available (browse/cart) during a checkout
  payment-provider outage, degrading gracefully rather than fully failing.

## 11. Architecture

ZodiCommerce is built by cloning the sanitized base codebase and running the
[genericization checklist](../../architecture/product-genericization-checklist.md):
the banking-specific `loans`/`dps`/`fdr`/`branches`/`branch_staff`/
`other_banks`/`beneficiaries`/`airtime` tables are stripped, since none of
ZodiCommerce's target retailers need them, and the `branch_staff` guard is
dropped by default. ZodiCommerce inherits the base engine's wallet/ledger
(used for store-credit issuance and refunds, per
[wallet-system.md](../../standards/wallet-system.md)), payment gateway
integrations (§22), RBAC/auth, KYC, i18n, and admin configuration surface
(per
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md))
unmodified, then layers its own Catalog, Inventory, Storefront, Orders,
Promotions, Channels, and Returns modules (§13) on top, per
[base-codebase-strategy.md](../../architecture/base-codebase-strategy.md#layering-a-products-domain-modules-onto-the-sanitized-base).

The storefront and the merchant's admin console are one Laravel application
per buyer deployment — see
[overview.md](../../architecture/overview.md#modular-monolith-one-codebase-per-product) —
with the public storefront pages rendered through the
[frontend–backend bridge](../../architecture/frontend-backend-bridge.md) so a
merchant edits catalog and CMS content from the admin panel with zero code
changes. There is no shared tenant boundary and no ZodiCore platform
dependency: each ZodiCommerce deployment is one merchant's standalone,
self-hosted instance, per
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md).

Channel connectors (marketplaces, shipping carriers, and, where a merchant
separately operates [ZodiPOS](../ZodiPOS/SPEC.md) on their own hosting, an
API-based POS order-ingestion connector) run as queued integration workers so
a third-party API outage cannot block checkout — this is an API integration
between two independently deployed products the merchant happens to own,
never a shared database or runtime. Inventory is modeled as a single ledger
service within this one deployment's database that every channel connector
writes through; because the deployment is single-tenant, that ledger is
unambiguously this one merchant's stock truth, with no `tenant_id` needed to
scope it. Merchants operating multiple brands or storefronts under one
deployment use the `company_id`-scoped multi-company model in §14, per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping) —
this is scoping within one deployment, not tenancy.

## 12. Technology

Laravel (PHP) + Vue per
[coding-standards-php-laravel.md](../../development/coding-standards-php-laravel.md)
and [coding-standards-vue.md](../../development/coding-standards-vue.md);
PostgreSQL + Redis per
[database-standards.md](../../development/database-standards.md); a
dedicated storefront caching/CDN layer for catalog pages; a queue-driven
integration layer for marketplace and shipping-carrier connectors so
third-party latency never blocks the checkout critical path.

## 13. Modules & Submodules

| Module | Submodules |
|---|---|
| Catalog | Products, Variants, Bundles, Categories, Channel Pricing |
| Inventory | Stock Ledger, Warehouse Allocation, Reservation, Low-Stock Alerts |
| Storefront | Cart, Checkout, Customer Accounts, Address Book |
| Orders | Order Lifecycle, Fulfillment Queue, Shipping Labels, Order Notes |
| Promotions | Discount Rules Engine, Coupon Codes, Scheduled Campaigns |
| Channels | Marketplace Connectors, POS Order Ingestion, Channel Health |
| Returns | RMA Lifecycle, Inspection, Disposition, Refunds |
| Reporting | Sales Dashboards, Channel Performance, Return-Rate Analytics |

## 14. Core Data Model

| Entity | Key columns |
|---|---|
| `companies` | id, name, default_currency, is_active |
| `products` | id, company_id, sku_prefix, name, status, category_id, created_at |
| `product_variants` | id, product_id, sku, attributes (jsonb), price, weight |
| `channels` | id, company_id, type (web/marketplace/pos), name, connector_config |
| `channel_listings` | id, variant_id, channel_id, channel_price, is_active |
| `inventory_ledger` | id, variant_id, warehouse_id, quantity_on_hand, quantity_reserved |
| `warehouses` | id, company_id, name, address, is_fulfillment_active |
| `orders` | id, company_id, channel_id, customer_id, status, subtotal, tax, total, placed_at |
| `order_items` | id, order_id, variant_id, quantity, unit_price, fulfilled_quantity |
| `shipments` | id, order_id, carrier, tracking_number, label_url, shipped_at |
| `promotions` | id, company_id, type, value, stacking_policy, starts_at, ends_at |
| `promotion_redemptions` | id, promotion_id, order_id, customer_id, redeemed_at |
| `customers` | id, company_id, user_id, default_address_id, lifetime_value |
| `rma_requests` | id, order_id, reason_code, status, disposition, refund_amount |
| `payment_transactions` | id, order_id, gateway, amount, status, gateway_reference |

`companies` is optional multi-brand/multi-storefront scoping within this one
merchant's single deployment (e.g. a merchant running two separate
storefront brands from one install) per
[localization-i18n.md](../../standards/localization-i18n.md#multi-company--multi-branch-data-scoping) —
a merchant running one brand has exactly one seeded `companies` row.

## 15. Key API Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/products` | List catalog products with variant/channel filters |
| POST | `/api/v1/products` | Create a product with variants |
| PATCH | `/api/v1/products/{id}` | Update product/variant fields |
| GET | `/api/v1/inventory/{variant_id}` | Read stock ledger across warehouses |
| POST | `/api/v1/inventory/{variant_id}/adjust` | Manual stock adjustment (audited) |
| POST | `/api/v1/cart` | Create/update a cart |
| POST | `/api/v1/checkout` | Finalize checkout, create order |
| GET | `/api/v1/orders` | List orders with channel/status filters |
| GET | `/api/v1/orders/{id}` | Order detail with items, shipments, RMA |
| POST | `/api/v1/orders/{id}/fulfill` | Mark items fulfilled, trigger label purchase |
| POST | `/api/v1/orders/{id}/cancel` | Cancel order, release reserved inventory |
| POST | `/api/v1/orders/{id}/rma` | Open an RMA against an order |
| PATCH | `/api/v1/rma/{id}` | Update RMA status/disposition |
| POST | `/api/v1/promotions` | Create a discount rule |
| POST | `/api/v1/promotions/validate` | Validate a code against a cart at checkout |
| GET | `/api/v1/channels` | List connected sales channels |
| POST | `/api/v1/channels/{id}/sync` | Force a channel inventory/order sync |
| GET | `/api/v1/shipping/rates` | Fetch live carrier rates for a cart/order |
| POST | `/api/v1/webhooks` | Register a webhook subscription |
| GET | `/api/v1/reports/sales` | Sales summary by channel/date range |

## 16. Events

`product.published`, `inventory.adjusted`, `inventory.low_stock`,
`order.placed`, `order.payment_confirmed`, `order.fulfilled`,
`order.shipped`, `order.delivered`, `order.canceled`, `order.returned`,
`rma.opened`, `rma.approved`, `rma.refunded`, `promotion.activated`,
`promotion.redeemed`, `channel.sync_failed`.

## 17. Notifications, Emails, SMS, Push

| Trigger event | In-app | Email | SMS | Push |
|---|---|---|---|---|
| `order.placed` (customer confirmation) | — | ✔ | ✔ (opt-in) | — |
| `order.shipped` (tracking) | — | ✔ | ✔ (opt-in) | ✔ |
| `inventory.low_stock` | ✔ (Fulfillment Manager) | ✔ | — | — |
| `rma.approved` | — | ✔ | — | — |
| `rma.refunded` | — | ✔ | — | — |
| `channel.sync_failed` | ✔ (Channel Manager) | ✔ | — | — |

All channels follow
[email-sms-standards.md](../../standards/email-sms-standards.md) and
[notification-standards.md](../../standards/notification-standards.md).

## 18. Permissions & Roles

Built on the base engine's inherited `Role`/`Permission` RBAC (not Spatie),
per
[admin-template.md](../../templates/admin-template.md#roles--permissions-inherited-not-spatie).
ZodiCommerce's `DemoSeeder` ships its own default admin roles — Store Owner,
Store Manager, Merchandiser, Fulfillment Manager, and Customer Service Rep —
each granted a subset of ZodiCommerce's product-specific permissions:
`catalog.manage`, `inventory.adjust`, `orders.fulfill`, `orders.cancel`,
`promotions.manage`, `rma.approve`, `rma.refund`, `channels.manage`.
`rma.refund` is not granted to the default `Customer Service Rep` role —
refund issuance requires `Store Manager` or above, consistent with the
approval chain in §19. A merchant can create additional custom roles and
reassign any permission entirely from the admin panel, with no code change,
per
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md#roles--permissions).

## 19. Workflows & Approval Chains

- **Refund approval**: refunds above an admin-configured threshold require a
  `Store Manager`-role approval step before the payment gateway is called,
  mirroring
  [modal-standards.md](../../standards/modal-standards.md#confirmation-dialogs).
- **Promotion activation approval**: promotions discounting more than a
  configurable percentage require a second approver before `starts_at` is
  honored, to prevent an accidental full-catalog giveaway.
- **RMA disposition**: warehouse inspection outcome (restock vs. write-off)
  is a required gate before a refund can be issued — a refund cannot be
  issued against an RMA still in `pending_inspection`.

## 20. Audit Logs

Every catalog change, inventory adjustment, order status transition, RMA
disposition, and promotion activation is recorded to this deployment's own
audit log (`audit_logs`) with actor, before/after values, and channel origin,
per [audit-logging.md](../../security/audit-logging.md). Manual inventory
adjustments always require a reason code captured in the audit entry.

## 21. Reports & Analytics & Dashboards

Sales-by-channel, sell-through rate by SKU, return-rate by product/reason
code, promotion redemption and margin impact, inventory turnover by
warehouse, and a fulfillment-SLA dashboard (time from `payment_confirmed` to
`shipped`). Dashboard-builder and scheduled-report capability inherited per
[dashboard-standards.md](../../standards/dashboard-standards.md).

## 22. Integrations

- **Payment gateways**: the inherited gateway catalog documented in
  [payment-gateways.md](../../standards/payment-gateways.md) — Stripe and
  Authorize.Net for card processing, BTCPay Server and CoinGate for optional
  cryptocurrency acceptance, Mollie and Razorpay for EU/India-focused
  merchants, Flutterwave and Paystack for merchants in Zodize's primary
  African market (added to the base per that standard's open action item if
  not already present), and the native manual/offline gateway as a
  always-available fallback. A merchant enables and configures only the
  gateways relevant to their market entirely from the admin panel — no code
  change either way.
- **Shipping carriers**: UPS, FedEx, USPS, DHL via a rate-shopping
  abstraction layer built as a ZodiCommerce domain module.
- **Marketplaces**: Amazon, Walmart Marketplace, eBay connectors normalizing
  orders/inventory into the shared ledger.
- **Tax calculation**: Avalara/TaxJar-class tax-engine integration.
- **Accounting**: order/refund export via an API integration into a
  separately-deployed [ZodiBusiness](../ZodiBusiness/SPEC.md) instance the
  merchant may run on their own hosting, or into an external ERP's chart of
  accounts — never a shared database, per
  [single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md#no-shared-platform-service).

## 23. AI Features

- Demand-forecasting assistance surfaced on the inventory dashboard,
  flagging SKUs trending toward stockout before the low-stock threshold
  fires.
- AI-assisted product description generation from a structured attribute
  set, always left as an editable draft, never auto-published.
- Return-reason clustering to surface systemic product-quality issues to
  Merchandisers.

## 24. Automation, Scheduled Jobs, CLI Commands

- Scheduled jobs: channel inventory reconciliation sweep, abandoned-cart
  reminder emails, promotion activation/expiration at scheduled timestamps,
  stale-reservation release (unpaid carts holding stock past a TTL).
- CLI commands: `commerce:sync-channel {id}`, `commerce:reconcile-inventory`,
  `commerce:release-stale-reservations`, `commerce:export-orders`.

## 25. Seed/Demo Data

`DemoSeeder` provisions a demo storefront with 3 warehouses, a 150-SKU
catalog with variants, 2 connected demo channels (web + a marketplace
connector in sandbox mode), 90 days of order history across all lifecycle
states including at least one RMA, and an active promotion, per
[migration-seeder-standards.md](../../development/migration-seeder-standards.md#seeders)
and the Demo Standard in [README.md](../../../README.md).

## 26. Performance Requirements

See §10; additionally: the fulfillment queue view must paginate/filter
10,000+ open orders without a full-table scan, and checkout must complete
inventory reservation and payment authorization within a single 5-second
user-perceived budget under normal load.

## 27. Security Requirements

Full baseline from
[security-standards.md](../../security/security-standards.md) applies.
Payment data never touches ZodiCommerce's own database — card details are
tokenized at the gateway per
[data-protection-privacy.md](../../security/data-protection-privacy.md);
only gateway references are stored. Customer PII (addresses, order history)
belongs to this one deployment's single merchant by construction — see
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md) —
and access within the deployment is governed by the RBAC model in §18, not a
tenant-isolation boundary.

## 28. Testing Requirements

Full baseline from
[testing-standards.md](../../development/testing-standards.md); additionally
a dedicated inventory-concurrency test suite validating that simultaneous
checkouts against the last unit of stock never both succeed (no oversell
race condition).

## 29. Deployment Requirements

Per [deployment-template.md](../../templates/deployment-template.md).
Checkout and storefront browse paths are deployed with zero-downtime
releases; channel connector workers can be redeployed independently without
storefront downtime.

## 30. Acceptance Criteria

- A product created with channel-specific pricing is correctly visible and
  priced on every connected channel within the inventory sync SLA (§10).
- Two simultaneous checkouts against the last unit of a variant never both
  succeed; exactly one order is placed and the other sees an out-of-stock
  state.
- An order can move end-to-end from `pending` to `shipped` with a real
  carrier label and tracking number without manual database intervention.
- An RMA can be opened, approved, inspected, and refunded, with every
  transition recorded in the audit log and reflected in the order's
  customer-facing status.

## 31. Production Checklist

See [production-readiness-checklist.md](../../checklists/production-readiness-checklist.md);
ZodiCommerce additionally requires sign-off that every connected channel
connector has passed a sandbox-mode order/inventory round-trip test before
being enabled in a live deployment.

## 32. Future Roadmap

- Subscription/recurring-order commerce (replenishment boxes).
- B2B wholesale storefront mode with tiered/contract pricing.
- In-store pickup (BOPIS) orchestration tying ZodiCommerce orders to
  ZodiPOS-fulfilled pickup.

## 33. Known Risks

- Channel connector drift: a marketplace API change can silently desync
  inventory — mitigated by the channel-health dashboard and sync-failure
  notifications (§17), but connector maintenance remains an ongoing
  operational cost as more marketplaces are added.
- Promotion stacking complexity: an under-specified stacking policy can
  produce unintended deep discounts at scale — mitigated by the promotion
  approval chain (§19), but the rule engine's edge cases warrant expanded
  test coverage as more promotion types are added.

## 34. Future Improvements

- Real-time inventory push (webhooks) to channels instead of polling-based
  reconciliation sweeps.
- Configurable fulfillment-routing rules (e.g. ship-from-nearest-warehouse
  optimization) beyond the current default allocation rule.

## Roadmap (spec depth)

This spec is Foundation-depth. Its Architecture and Core Data Model sections
were revised to the standalone, self-hosted, single-tenant base-codebase
model described in
[architecture/overview.md](../../architecture/overview.md) and
[single-tenant-deployment-model.md](../../architecture/single-tenant-deployment-model.md);
ZodiCommerce is the second product in the build order
([ROADMAP.md](../../../ROADMAP.md)) validating the clone → genericize →
bridge → extend pipeline this correction assumes. Queued for Deep-depth
expansion: a full ER diagram covering tax-jurisdiction and multi-currency
pricing tables, the complete endpoint catalog (bulk catalog operations,
webhook management endpoints), and a dedicated
`DATA_MODEL.md`/`API_REFERENCE.md` pair matching
[ZodiCore](../ZodiCore/SPEC.md)'s companion-document structure.

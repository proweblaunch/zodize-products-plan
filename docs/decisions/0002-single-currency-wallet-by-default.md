# 0002. Single-currency wallet by default; multi-currency is an explicit per-product extension

- **Status**: Accepted
- **Date**: 2026-07-31

## Context

The base codebase's inherited wallet/ledger engine (see
[`../standards/wallet-system.md`](../standards/wallet-system.md)) stores one
`balance` column per `User` in the deployment's single configured base
currency, with foreign-currency payments converted to that base currency
only at the payment-gateway layer (`gateway_currencies`). This is a real
architectural limitation relative to a "true" multi-currency wallet where a
single user could hold, say, both a USD and a NGN balance simultaneously.

Several financial-grade products in the catalog (ZodiBank, ZodiTrade,
ZodiXchange, ZodiCapital, ZodiYield) plausibly need multi-currency balances
for their real-world domain. We need to decide whether to redesign the
shared base wallet engine to be multi-currency for every product, or to keep
it single-currency by default and extend it per product where justified.

## Decision

The shared base wallet engine (`User.balance` + `Transaction`) **remains
single-currency by default** for every product. It is not redesigned
globally.

Products that genuinely require multiple simultaneous currency balances per
user/account extend the pattern **in their own domain module**: a
product-specific balances table with an explicit `currency` column, scoped
per account rather than per user, and a product-specific transaction/ledger
table following the same append-only, post-balance-snapshot invariant as the
inherited `Transaction` model — rather than modifying the shared
`User`/`Transaction` schema itself.

## Consequences

- The majority of Zodize products (retail, hospitality, healthcare,
  education, construction, agriculture, logistics verticals) get a working,
  audited wallet engine with zero additional design work, matching their
  actual single-currency operating reality.
- Financial-grade products that need multi-currency balances carry the cost
  of that extension in their own codebase and their own `SPEC.md`, isolated
  from the shared base — a defect or complexity increase in a
  multi-currency extension cannot destabilize the wallet engine every other
  product depends on.
- A product that later discovers it needs multi-currency balances (a scope
  change after launch) can add the extension without a breaking migration
  of the shared engine, since the shared engine's schema is untouched.
- This means `ZodiTrade`, `ZodiXchange`, `ZodiBank`, `ZodiCapital`, and
  `ZodiYield` — see
  [`../../PRODUCT_CATALOG.md`](../../PRODUCT_CATALOG.md) — each explicitly
  document in their own `SPEC.md` whether they require this extension and,
  if so, the shape of their product-specific multi-currency balances table.

## Alternatives considered

- **Redesign the shared engine to be multi-currency for every product.**
  Rejected: the majority of products don't need it, and a `currency`-scoped
  balance/transaction model is meaningfully more complex to reason about
  (which balance is "the" balance for a simple product with one operating
  currency) for no benefit to those products.
- **Leave the gap undocumented and let each product discover it during
  implementation.** Rejected: this is exactly the kind of undocumented
  implementation assumption [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md)
  prohibits — the gap is real, so it must be a written decision, not a
  silent one.

# Wallet & Ledger System

> Documents the EXISTING double-entry balance engine inherited from the base
> codebase (see
> [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)).
> This is not a new design — every product inherits this engine as-is and
> builds domain behavior on top of it.

## The inherited engine

Every product inherits a working wallet/ledger engine from the base
codebase's `Admin/ManageUsersController.php`, `DepositController.php`,
`WithdrawalController.php`, and the `User`, `Transaction`, and
`BalanceTransfer` models:

- Every `User` row carries a `balance` column — the user's current spendable
  balance in the deployment's base currency (see
  [`localization-i18n.md`](./localization-i18n.md#what-the-base-codebase-actually-provides-today)).
- **Every balance change writes an immutable `Transaction` row.** A
  `Transaction` records: the user, the amount (signed — positive for
  credit, negative for debit), the transaction type/trigger (deposit,
  withdrawal, transfer, referral commission, admin adjustment, and any
  product-specific trigger a domain module adds), a human-readable remark,
  and — critically — a **post-balance snapshot**: the user's resulting
  balance immediately after this transaction was applied. This makes the
  ledger self-verifying: summing signed amounts from any point forward must
  reconcile against the snapshot at that point, and a corrupted/tampered
  `balance` column value on the `User` row is detectable by replaying
  `Transaction` history.
- `Transaction` rows are never updated or deleted by application code — the
  ledger is append-only, matching the audit-log immutability requirement in
  [`../security/audit-logging.md`](../security/audit-logging.md#append-only-audit-log).
  A correction is a new, offsetting `Transaction`, never an edit to a past
  one.
- **`BalanceTransfer`** handles user-to-user balance movement (e.g. a peer
  transfer feature) as a paired operation: a debit `Transaction` on the
  sender and a credit `Transaction` on the recipient, created atomically in
  one database transaction so a failure partway through cannot leave the
  ledger unbalanced.
- **Deposits** (`DepositController`) originate from a payment gateway
  callback/webhook (see
  [`payment-gateways.md`](./payment-gateways.md)) or an admin manual credit,
  and always produce a `Transaction` crediting the user's balance.
- **Withdrawals** (`WithdrawalController`) debit the user's balance against
  a configured withdrawal method (see
  [`admin-configuration-baseline.md`](./admin-configuration-baseline.md#withdraw-method-configuration))
  and go through an admin approval step before the payout is marked
  complete — the debit and its approval state are both tracked on the
  `Withdrawal` record, itself linked to its resulting `Transaction`.

## What a product's domain modules do with this engine

A product's own domain modules never implement their own balance-tracking
logic. Instead they call the inherited engine's transaction-creation path
whenever a domain event should move money:

- ZodiCommerce: an order refund credits the customer's wallet balance as
  store credit via a `Transaction`, rather than a separate "store credit"
  table.
- ZodiBank: a loan disbursement, interest accrual, and repayment are each a
  `Transaction` against the borrower's `User` balance, with the
  loan-specific business rules (amortization, rate) living in ZodiBank's own
  `Loan`/`LoanPlan` domain module (re-added per
  [`../architecture/product-genericization-checklist.md`](../architecture/product-genericization-checklist.md#step-6--verify-the-plan-pattern-fits-or-extend-it)),
  which calls into the shared ledger rather than reimplementing it.
- ZodiPOS: a till/cash-drawer reconciliation reads `Transaction` history for
  the shift's timeframe rather than maintaining a parallel total.

Any new domain module that moves money MUST create a `Transaction` for
every balance change it causes and MUST NOT write directly to `User.balance`
without going through the same path — a direct `UPDATE users SET balance =
...` that bypasses `Transaction` creation breaks the self-verifying ledger
property above and is a defect, not a shortcut.

## Multi-currency gap

**Status: known gap, resolved per product, not silently assumed.** As
documented in
[`localization-i18n.md`](./localization-i18n.md#what-the-base-codebase-actually-provides-today),
the inherited `User.balance` column and `Transaction` model are
single-currency — one balance, in the deployment's one configured base
currency, per user. There is no `currency` column on `Transaction` or
`User.balance` in the inherited engine; multi-currency handling today exists
only as a payment-gateway-layer conversion at the moment of deposit.

This is sufficient for the majority of Zodize products, where the buyer's
business operates in one currency and a foreign-currency payment is simply
converted to that one currency at deposit time — no product should extend
the base wallet engine to hold multiple simultaneous currency balances
unless its own `SPEC.md` explicitly requires it.

**Decision** (see
[`../decisions/0002-single-currency-wallet-by-default.md`](../decisions/0002-single-currency-wallet-by-default.md)
for the full ADR): the base wallet engine remains single-currency by
default for every product. Products whose domain genuinely requires holding
multiple currency balances simultaneously for the same user/account — most
notably **ZodiXchange** (multi-asset exchange balances), **ZodiTrade**
(multi-currency brokerage cash balances), and **ZodiBank** (multi-currency
deposit accounts, if the product's target market requires them) — MUST
extend the engine explicitly in their own domain module: add a `currency`
column to a product-specific balances table (not the inherited
single-column `User.balance`) and a `currency` column to that product's own
transaction/ledger rows, scoped per account rather than per user, while
still following the same append-only, post-balance-snapshot pattern
established by the inherited `Transaction` model. This extension is scoped
and justified per product in that product's own `SPEC.md`, never applied to
the shared base engine globally.

## Related standards

- [`payment-gateways.md`](./payment-gateways.md)
- [`localization-i18n.md`](./localization-i18n.md)
- [`../security/audit-logging.md`](../security/audit-logging.md)
- [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)

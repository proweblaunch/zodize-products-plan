# Payment Gateway Standards

> Documents the EXISTING gateway integration layer inherited from the base
> codebase (see
> [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)).
> Every product inherits this catalog as-is; a buyer enables and configures
> only the gateways relevant to their market from the admin panel — no code
> change either way.

## The inherited engine

Gateway integrations live under `app/Http/Controllers/Gateway/` in the base
codebase, one controller per gateway, plus two tables:

- **`gateways`** — one row per available gateway, holding its enabled/
  disabled state, display name/logo, and the credential fields it needs
  (API key, secret key, webhook secret, merchant ID — field set varies per
  gateway).
- **`gateway_currencies`** — one row per gateway per supported currency,
  holding the conversion rate used to translate a payment made in that
  currency into the deployment's base currency at deposit time (see
  [`localization-i18n.md`](./localization-i18n.md#multi-currency-standard)
  and [`wallet-system.md`](./wallet-system.md)).

A buyer configures a gateway entirely from the admin panel: enable it,
paste in the API credentials the gateway provider issued them, set which
currencies it accepts and at what rate, and the gateway is live — this is
part of the buyer's zero-code configuration workflow described in
[`admin-configuration-baseline.md`](./admin-configuration-baseline.md).

## Confirmed gateway inventory

The following automated (API-driven) gateways are confirmed present in the
base codebase's `composer.json` dependencies and their corresponding
`Gateway/` controllers, as of the source audit:

| Gateway | Package | Category |
|---|---|---|
| Authorize.Net | `authorizenet/authorizenet` | Card processor |
| BTCPay Server | `btcpayserver/btcpayserver-greenfield-php` | Cryptocurrency |
| CoinGate | `coingate/coingate-php` | Cryptocurrency |
| Mollie | `mollie/laravel-mollie` | Card / bank processor (EU-focused) |
| Razorpay | `razorpay/razorpay` | Card / UPI processor (India-focused) |
| Stripe | `stripe/stripe-php` | Card processor |
| Manual/Offline | (native, no package) | Bank transfer / cash instructions with admin-side manual confirmation |

The base codebase's own marketing materials and gateway count (30+)
indicate substantially more automated gateways than the packages
enumerated in the audited `composer.json` excerpt above account for — the
full `Gateway/` controller directory listing was not exhaustively enumerated
during this audit pass. Treat the table above as **confirmed**, not
exhaustive; before extending or removing any gateway during a product's
[genericization pass](../architecture/product-genericization-checklist.md),
enumerate `app/Http/Controllers/Gateway/*.php` directly against the actual
codebase rather than relying on this table alone.

## Flutterwave and Paystack — required for Zodize's primary market

Flutterwave and Paystack are not confirmed among the gateway packages
identified in this audit's `composer.json` excerpt, and are essential for
the Nigerian/African market Zodize primarily serves. **This is an open
action item, not a resolved fact:**

- Before a product ships to a buyer in Zodize's primary market, verify
  directly against the actual `app/Http/Controllers/Gateway/` directory and
  `composer.json` of the base codebase whether `flutterwavedev/flutterwave-v3`
  (or equivalent) and a Paystack PHP SDK are already integrated.
- **If either is missing**, add it as part of the
  [one-time base cleanup](../architecture/base-codebase-strategy.md#one-time-base-cleanup-fix-once-before-first-clone) —
  once, in the base codebase, before the first product clone — following
  the same integration shape as the confirmed gateways above: a
  `Gateway/FlutterwaveController.php` / `Gateway/PaystackController.php`
  implementing the initiate-payment, callback/webhook-verify, and
  deposit-credit flow (creating a `Transaction` via
  [`wallet-system.md`](./wallet-system.md)), a `gateways` row seeded for it,
  and its required credential fields (public key, secret key, webhook
  secret hash) added to the admin gateway configuration form.
- This is a base-codebase task, done exactly once, not a per-product task —
  every product cloned afterward inherits both gateways automatically.

## Gateway categories and selection guidance per product

| Category | Gateways | Typical fit |
|---|---|---|
| Card processors (global) | Stripe, Authorize.Net | Any product accepting international card payments |
| Card / mobile money (Africa) | Flutterwave, Paystack (pending confirmation above) | Any product targeting Nigerian/African buyers and their end customers |
| Regional processors | Mollie (EU), Razorpay (India) | Products with a specific target market matching that processor's region |
| Cryptocurrency | BTCPay Server, CoinGate | Products explicitly supporting crypto payment (e.g. ZodiXchange, ZodiTrade), or as an optional payment method for any product |
| Manual/offline | Native manual gateway | Every product — always enabled as a fallback for buyers whose market lacks automated processor coverage, or during initial setup before automated gateways are configured |

A product's own [`SPEC.md`](../products/) documents which gateways are
enabled by default in its `DemoSeeder`, matching that product's stated
target market — see each product's Payment Gateways / Integrations section.

## Webhook handling requirement

Every automated gateway controller MUST verify the authenticity of its
callback/webhook (signature or secret verification per that gateway's own
mechanism) before crediting a deposit `Transaction` — an unverified webhook
credit path is a critical financial vulnerability and fails
[`../security/security-checklist.md`](../checklists/security-checklist.md).

## Related standards

- [`wallet-system.md`](./wallet-system.md)
- [`localization-i18n.md`](./localization-i18n.md)
- [`admin-configuration-baseline.md`](./admin-configuration-baseline.md)
- [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)

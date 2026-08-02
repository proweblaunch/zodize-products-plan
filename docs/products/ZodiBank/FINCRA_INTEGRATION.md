# ZodiBank — Fincra Integration

> Documents a new, admin-configurable Fincra integration module for
> ZodiBank. Fincra (https://docs.fincra.com/docs/getting-started) is a
> Nigerian/African payments infrastructure provider offering collections,
> disbursements, virtual account issuance, and identity verification. This
> module does not exist in Pay Secure's codebase today — it is new work,
> built to the same admin-configuration standard as every other gateway in
> [payment-gateways.md](../../standards/payment-gateways.md) and
> [admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md).
> See [SPEC.md §11](./SPEC.md#11-architecture) and
> [SPEC.md §22](./SPEC.md#22-integrations) for how this module fits into
> ZodiBank's overall architecture.

## Why Fincra, and how it differs from the existing gateways

ZodiBank's Pay Secure foundation already integrates Authorize.Net,
Flutterwave, CoinGate, and CinetPay (confirmed by audit, see
[SPEC.md §11](./SPEC.md#11-architecture)). Fincra is additive: it covers
Nigerian/African payins and payouts the same way Flutterwave does, but its
Virtual Accounts and Identity Management feature areas have no equivalent
among ZodiBank's existing gateways, and this module ties both of those
directly into ZodiBank's own domain modules (Account Numbers and KYC,
respectively) rather than standing alone as a payment processor only.

## Feature areas

Fincra exposes four independent feature areas. Each has its own
admin-panel enable/disable toggle (see [Settings screen](#settings-screen)
below) so a buyer can, for example, enable only Payouts without exposing
Payins, Virtual Accounts, or Identity Management.

### Payins (collections)

Collects funds from a ZodiBank customer via card, bank transfer, or virtual
account funding, and credits the customer's ZodiBank balance on successful
completion. Implemented as `Gateway/FincraController.php` following the
same initiate-payment / callback-verify / deposit-credit shape as every
other gateway documented in
[payment-gateways.md](../../standards/payment-gateways.md#the-inherited-engine):

- ZodiBank initiates a payin request against Fincra's collections endpoint
  with amount, currency, and customer reference.
- The customer completes payment on Fincra's hosted page or via the
  selected in-app payment method.
- Fincra sends a webhook on completion (see
  [Webhook handling](#webhook-handling)); on verified success, ZodiBank
  creates a `Transaction` crediting the customer's balance, per
  [wallet-system.md](../../standards/wallet-system.md).

### Payouts (disbursements)

Sends funds out of ZodiBank to a bank account or mobile money wallet —
used for withdrawals and any outbound disbursement flow. Implemented
alongside the existing
[withdraw method configuration](../../standards/admin-configuration-baseline.md#withdraw-method-configuration):
a buyer adds "Fincra Bank Transfer" and/or "Fincra Mobile Money" as
`WithdrawMethod` entries backed by this module, so the existing
`WithdrawalController` admin-approval flow governs payout release exactly
as it does for every other withdrawal method — this module supplies the
Fincra-side payout call and status polling/webhook, not a parallel
withdrawal-approval mechanism.

- ZodiBank submits a payout request to Fincra's disbursement endpoint with
  destination account/mobile-money details, amount, and currency.
- Payout status (`pending` → `successful`/`failed`) updates the linked
  `Withdrawal` record; a `failed` payout reverses the debit via an
  offsetting `Transaction`, never an edit to the original debit entry, per
  [wallet-system.md](../../standards/wallet-system.md).

### Virtual Accounts

Issues a dedicated Fincra virtual account number per ZodiBank customer, so
a customer receives a permanent bank account number that routes incoming
transfers directly to their ZodiBank balance without a manual payin step
each time. This feature area is the integration point between Fincra and
ZodiBank's own **Account Numbers** module (a new module ZodiBank must build
— see [SPEC.md §13](./SPEC.md#13-modules--submodules)):

- When Virtual Accounts is enabled and a customer opts in (or the deposit
  account product requires it), ZodiBank calls Fincra's virtual account
  issuance endpoint and stores the returned account number/bank details
  against the customer's `deposit_accounts` row, alongside (not instead of)
  the account number ZodiBank's own Account Numbers module generates for
  its internal ledger identity.
- Incoming credits to the Fincra-issued virtual account arrive via the same
  webhook path as a Payin, and are treated identically: a verified webhook
  creates a `Transaction` crediting the linked customer's balance.

### Identity Management

Performs KYC/BVN/identity verification lookups against Fincra's identity
API. This feature area ties into the base engine's existing KYC
form-builder (`Admin/KycController.php`, `Form` model — see
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md#kyc))
rather than introducing a second, parallel KYC review flow:

- An admin adds a "Fincra BVN Lookup" (or equivalent) field/action to the
  KYC form definition from the existing form-builder, with no code change.
- On submission, ZodiBank calls Fincra's identity verification endpoint
  server-side and stores the verification result against the customer's
  `users.kyc_data`, feeding into the same approve/reject review screen
  every other KYC submission goes through.
- This feature area is independent of Payins/Payouts/Virtual Accounts — a
  buyer may enable Identity Management for KYC purposes while leaving all
  payment-moving features disabled, or vice versa.

## Authentication flow

> See [Verification Required](#verification-required) below — the exact
> header/field names in this section are the pattern to implement, not
> independently confirmed against Fincra's live API in this session.

Fincra uses API key pair authentication (a public key and a secret key,
each issued per test/live mode from the Fincra dashboard) plus a Business
ID that scopes most endpoints to the merchant's Fincra account:

- Server-to-server calls (payin initiation, payout submission, virtual
  account issuance, identity lookups) authenticate by sending the **secret
  key** in an `api-key` request header.
- Most endpoints additionally require the merchant's **Business ID**,
  either as a path segment or a request-body field, scoping the call to
  the correct Fincra business/merchant account.
- The **public key** is used only for any client-side/hosted-page
  initialization step (e.g. rendering a hosted payment page), never for
  server-to-server calls, following the same public/secret split pattern
  as Stripe and other card processors already in ZodiBank's gateway set.
- ZodiBank never exposes the secret key or Business ID to the browser; both
  are stored server-side only, in the `gateways` table's credential fields
  for this gateway row, the same as every other gateway per
  [payment-gateways.md](../../standards/payment-gateways.md#the-inherited-engine).

## Settings screen

A new admin panel screen (`Admin/FincraSettingController.php`, following
the same shape as the other gateway config screens described in
[admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md#payment-gateways))
exposes, with no code editing required:

| Field | Notes |
|---|---|
| Mode | Test / Live toggle |
| API key (secret) | Entered once per mode (test key, live key) |
| API key (public) | Entered once per mode, used only for hosted-page initialization |
| Business ID | Single field, scopes all authenticated calls |
| Webhook URL | Auto-generated by ZodiBank (e.g. `https://{buyer-domain}/webhooks/fincra`), displayed read-only for the admin to copy and paste into Fincra's own dashboard webhook configuration — ZodiBank does not call out to Fincra to register this URL automatically |
| Webhook secret | Entered by the admin, copied from Fincra's dashboard, used to verify inbound webhook signatures (see below) |
| Payins enabled | Toggle |
| Payouts enabled | Toggle |
| Virtual Accounts enabled | Toggle |
| Identity Management enabled | Toggle |

Each of the four feature toggles is independent: disabling Payins does not
disable Payouts, Virtual Accounts, or Identity Management, and a disabled
feature's routes/controllers reject requests rather than silently no-op,
so a buyer can confirm from the admin panel exactly which Fincra
capabilities are live at any time.

## Webhook handling

Fincra delivers webhooks for payin completion, payout status changes, and
virtual account credit events to the single auto-generated webhook URL
above, disambiguated by an event-type field in the payload. ZodiBank's
webhook receiver for this gateway follows the same inbound-verification
posture required of every gateway webhook by
[payment-gateways.md](../../standards/payment-gateways.md#webhook-handling-requirement)
and the general inbound-verification pattern in
[webhook-standards.md](../../development/webhook-standards.md#signing-and-verification):

- **Signature verification is mandatory before any balance-affecting
  action.** The receiver computes an HMAC over the raw request body using
  the admin-configured webhook secret and compares it against the
  signature Fincra sends in a request header (see
  [Verification Required](#verification-required) for the exact header
  name). A request with a missing or invalid signature is rejected with no
  `Transaction` created and no `Withdrawal` status change applied — an
  unverified webhook credit path is a critical financial vulnerability per
  [payment-gateways.md](../../standards/payment-gateways.md#webhook-handling-requirement).
- **Idempotency is mandatory.** Every Fincra webhook payload carries a
  unique event/reference ID; ZodiBank records processed event IDs and
  discards a duplicate delivery without creating a second `Transaction` or
  applying a second status change, matching the at-least-once delivery
  assumption in
  [webhook-standards.md](../../development/webhook-standards.md#delivery-guarantees).
- **Ledger effect**: a verified Payin or Virtual Account credit webhook
  creates a `Transaction` crediting the linked customer's ZodiBank balance,
  with a post-balance snapshot, exactly the same pattern every other
  gateway's deposit webhook follows per
  [wallet-system.md](../../standards/wallet-system.md#the-inherited-engine).
  A verified Payout webhook updates the linked `Withdrawal` record's status
  and, on failure, creates an offsetting `Transaction` reversing the
  original debit rather than editing it.
- **Replay/staleness protection**: the receiver rejects a webhook whose
  timestamp (if Fincra's payload includes one) is older than a bounded
  window, following the same 5-minute replay-protection posture as
  [webhook-standards.md](../../development/webhook-standards.md#signing-and-verification),
  pending confirmation of whether Fincra's payload includes a timestamp
  field to check against (see
  [Verification Required](#verification-required)).

## Verification Required

The following details are the pattern to implement for a payment API of
this class (API key header auth, business/merchant ID scoping, webhook
signature verification header), following the same shape as ZodiBank's
other gateway integrations — they are **not** independently confirmed
against Fincra's live API documentation in this session (an attempt to
fetch `https://docs.fincra.com/docs/getting-started` in this session
returned an HTTP 403 and could not be read). Before implementation,
whoever builds this module MUST verify each of the following directly
against the live Fincra API docs and correct this document if any differ:

- The exact header name used for secret-key authentication on
  server-to-server calls (documented above as `api-key`).
- Whether the Business ID is passed as a path segment, a request-body
  field, or a header, and its exact field/parameter name.
- The exact request header name Fincra uses to deliver a webhook's HMAC
  signature (documented above as unconfirmed — do not assume a name until
  verified).
- The exact hashing algorithm and signing-string construction for webhook
  signatures (e.g. whether it signs the raw body alone or a
  timestamp-prefixed string, matching or differing from the
  `t=<timestamp>,v1=<hmac>` shape ZodiBank's own outbound webhooks use per
  [webhook-standards.md](../../development/webhook-standards.md#signing-and-verification)).
- Whether Fincra's webhook payload includes a timestamp field usable for
  replay protection, and if so, its field name and format.
- The exact endpoint paths, request/response shapes, and rate limits for
  the Payins, Payouts, Virtual Accounts, and Identity Management APIs
  referenced above — this document describes the integration's shape and
  admin-configuration surface, not a verified endpoint catalog.
- Whether Fincra requires separate API key pairs for test and live mode
  (assumed above, matching every other gateway's test/live pattern) or a
  single key pair with a mode flag elsewhere in the request.

## Related standards

- [payment-gateways.md](../../standards/payment-gateways.md)
- [admin-configuration-baseline.md](../../standards/admin-configuration-baseline.md)
- [wallet-system.md](../../standards/wallet-system.md)
- [webhook-standards.md](../../development/webhook-standards.md)
- [SPEC.md](./SPEC.md)

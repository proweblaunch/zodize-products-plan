# Admin Configuration Baseline

> Every setting a buyer configures from the admin panel with zero code
> access, once they've uploaded the codebase and pointed `.env` at their
> database. This is the concrete inventory behind the "no code editing, no
> CLI" promise in
> [`../architecture/overview.md`](../architecture/overview.md#the-business-model-this-architecture-serves).
> Every product inherits every item below from the base codebase — a
> product's own domain modules ADD to this list, they never remove from it.

## General settings & branding

Controller: `Admin/GeneralSettingController.php`. Model: `GeneralSetting`
(single-row table, cached under key `GeneralSetting`).

- Site name, logo, favicon, base currency symbol/code, timezone.
- Contact details, social links, admin notification email/phone.
- Social login (Google/Facebook/LinkedIn) — enable/disable and
  client ID/secret per provider, via
  `GeneralSettingController@socialiteCredentials`.

## Payment gateways

Controller: `Admin/*` gateway config screens over `gateways` /
`gateway_currencies`. See [`payment-gateways.md`](./payment-gateways.md).

- Enable/disable each available gateway.
- Enter API credentials per gateway (varies per gateway: API key, secret
  key, webhook secret, merchant ID).
- Configure accepted currencies and conversion rate per gateway.

## Withdraw method configuration

Controller: `Admin/WithdrawMethodController.php`. Models: `WithdrawMethod`,
`Withdrawal`.

- Define payout methods available to users (bank transfer, mobile money,
  crypto wallet, etc.), each with an admin-defined, dynamic JSON-schema
  input form (the fields a user fills in to request that withdrawal method —
  e.g. bank name/account number for a bank transfer method) with no code
  required to add a new method's field layout.
- Set minimum/maximum withdrawal limits and processing fee per method.
- Review and approve/reject pending withdrawal requests.

## Referral program

Controller: `Admin/ReferralSettingController.php`.

- Enable/disable the referral program.
- Configure commission percentage per referral level (multi-level, via
  `ref_by` on `users`).
- Set which triggering events (signup, first deposit, plan purchase) award
  a referral commission.

## Plans

The genericized `Plan` pattern (see
[`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md#genericizing-the-plan-pattern)).

- Create/edit/deactivate plans: name, description, price or rate, term,
  limits, and a features JSON — no code required to add a new plan or
  change its terms.

## KYC

Controller: `Admin/KycController.php`. Model: `Form` (dynamic JSON
form-builder schema), `users.kyc_data`.

- Define the KYC form's fields entirely from the admin panel (text field,
  file upload, dropdown, date — form-builder driven, not hardcoded Blade).
- Review submitted KYC data per user; approve/reject with a reason.

## Language / i18n

Controller: `Admin/LanguageController.php`. Model: `Language`. See
[`localization-i18n.md`](./localization-i18n.md).

- Add a language, set it active/inactive, set the default language.
- Edit every translation string for every active language directly in the
  admin panel.

## Cron / scheduled tasks

Controller: `Admin/CronConfigurationController.php`. Models: `CronJob`,
`CronJobLog`, `CronSchedule`.

- View every scheduled task the product runs, its schedule, and its
  DB-logged run history (success/failure, last run time) — a buyer can
  confirm their server's cron entry is actually firing without SSH access.

## Extensions

Controller: `Admin/ExtensionController.php`.

- Toggle and configure Google Analytics, Tawk.to live chat, Facebook Pixel,
  and custom captcha providers — credential/config entry only, no Blade
  edits.

## Frontend / CMS / page builder

Controller: `Admin/FrontendController.php`. Models: `Frontend`, `Page`. See
[`../architecture/frontend-backend-bridge.md`](../architecture/frontend-backend-bridge.md).

- Add/edit/reorder page sections using the registered `x-zodize.*` component
  types, with per-section content entered as form fields (not raw JSON
  editing, once the frontend bridge's admin schema extension is in place).
- Edit SEO metadata (title, description, OG image, canonical URL) per page.
- Edit static policy pages (Terms, Privacy, Refund Policy, etc.).
- Toggle maintenance mode, with an admin-editable maintenance message.
- Toggle and edit the GDPR cookie consent banner text.
- Edit `sitemap.xml` / `robots.xml` content directly from an in-panel code
  editor (writes the physical file — see the file-permission requirement
  noted in
  [`base-codebase-strategy.md`](../architecture/base-codebase-strategy.md#one-time-base-cleanup-fix-once-before-first-clone)).

## Roles & permissions

Custom RBAC (not Spatie) via `Role`/`Permission` models +
`AdminPermissionMiddleware`. See
[`../security/rbac-permissions.md`](../security/rbac-permissions.md).

- Create/edit admin roles and assign granular permissions per role, entirely
  from the admin panel.
- Assign roles to admin staff accounts.

## Notifications

Dispatcher: `App\Notify\Notify`, with Mailjet, MessageBird, SendGrid,
Twilio, and Vonage already integrated.

- Configure which notification provider is active per channel (email/SMS)
  and enter that provider's API credentials.
- Edit notification templates (subject/body, with merge-field placeholders)
  per triggering event, from the admin panel.

## What is NOT in this baseline

Domain-specific configuration a product's own modules introduce (e.g.
ZodiHotel's room rate calendar, ZodiMed's clinical note templates) is
documented in that product's own
[`SPEC.md`](../products/) and MUST be built to the same standard as this
baseline: configurable from the admin panel, never requiring a code change
by the buyer. A product spec that describes a setting only a developer can
change fails
[`../checklists/production-readiness-checklist.md`](../checklists/production-readiness-checklist.md).

## Related standards

- [`../architecture/base-codebase-strategy.md`](../architecture/base-codebase-strategy.md)
- [`../architecture/frontend-backend-bridge.md`](../architecture/frontend-backend-bridge.md)
- [`payment-gateways.md`](./payment-gateways.md)
- [`wallet-system.md`](./wallet-system.md)
- [`localization-i18n.md`](./localization-i18n.md)

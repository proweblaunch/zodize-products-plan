# Build State Ledger

> The single source of truth for exactly where autonomous/human build
> execution stands, across all 21 Zodize products (20 original + ZodiChain,
> promoted from future-expansion — see
> [`PRODUCT_CATALOG.md`](./PRODUCT_CATALOG.md)). This file is the
> resumability mechanism for
> [`docs/architecture/deployment-paths.md`](./docs/architecture/deployment-paths.md)'s
> build convention — read it before touching any code.

## Protocol (every session MUST follow this, no exceptions)

1. **Read this file first**, before touching any code, at the start of any
   session — fresh or resumed.
2. **Verify before trusting.** Before starting or resuming work on a
   product, confirm its actual on-disk state at
   `/home/script/public_html/<product-slug>/` matches what this ledger
   claims. If they disagree, **trust the filesystem**, investigate the
   discrepancy, and correct this ledger before proceeding — never assume
   the ledger is right when the code disagrees with it.
3. **Commit small, commit often.** After completing any meaningfully-sized
   unit of work (a module, a migration set, a feature), commit it with a
   clear message **and** update this ledger's relevant row **in the same
   commit**. Never leave more than one unit of work uncommitted — if a
   session ends unexpectedly, at most one small unit of work is lost, never
   a half-finished product.
4. **Never re-run destructive setup** (fresh migrations, base-codebase
   re-clone, dependency reinstall-from-scratch) against a product already
   marked `in-progress` or later. Resume forward from the last committed
   state; don't restart it.
5. **When in doubt, stop and flag.** If a session cannot safely determine
   what state a product is in — the path is unreachable, the ledger and the
   filesystem disagree in a way that isn't obviously resolvable, or the
   next step is genuinely ambiguous — STOP and record the blocker in this
   file's Flagged Items section rather than guessing or overwriting
   anything that isn't certainly safe to overwrite.
6. **`Live — Extend Only` products are never scaffolded.** For any product
   in that status (see [`PRODUCT_CATALOG.md`](./PRODUCT_CATALOG.md)), a
   session MUST NOT clone the base codebase, run destructive migrations, or
   treat it as a from-scratch build in any way. The only allowed workflow:
   audit the existing live codebase against `docs/products/<product>/SPEC.md`,
   maintain a gap list (features in the spec not present in the live code)
   in that product's row below, and add only additive features via normal,
   rollback-safe migrations — never modifying or removing anything already
   functioning.
7. **Alternate-base products are not genericized from qfsfountains.**
   ZodiBank (Pay Secure), ZodiCore (Ultimate POS), and ZodiCapital/ZodiYield
   (novavest) are each built from a different, already-substantial existing
   codebase — see each product's own `SPEC.md` §11 for the specifics. The
   standard clone-qfsfountains-and-genericize pipeline in
   [`docs/architecture/product-genericization-checklist.md`](./docs/architecture/product-genericization-checklist.md)
   does not apply to these four; treat them as "audit the existing base,
   extend/fix additively" work, closer in spirit to rule 6 above than to a
   from-scratch build, even though they are not sold to real buyers yet
   (so not `Live — Extend Only` in PRODUCT_CATALOG.md's strict sense).

## Environment access

**Resolved.** An earlier version of this ledger recorded that
`/home/script/public_html/` and the source codebases were unreachable —
that blocker is resolved as of the session that added this paragraph. The
**Zodize MCP Server** (filesystem/git/GitHub/terminal/database/deployment
tools) provides real access to the build VPS, with a workspace root of
`/home/` — confirmed by direct listing, which surfaced `script/`,
`qfsfountains/`, `zodize/`, `novavest/`, `dash/`, `web3chainlink/`, and
several other product/client directories not part of Zodize's 20(+1)
product catalog (e.g. `altaramarkets`, `clintrade`, `davidomakamba`,
`elitratrustbank`, `jaguarmarkets`, `refinedresidence`, `rochygreen`,
`shieldsafebank`, `trustsharelogistics`, `veluxtech`, and others under
`script/public_html/` itself: `altaramarkets`, `demo1`/`demo2`/`demo3`,
`g3ph`, `zodira`). **Do not touch anything outside the 20(+1) product
slugs** — those other directories belong to unrelated work and are out of
scope for this repository's build execution, per the task instruction that
established this rule.

## Status definitions

- `not-started` — no code exists at this product's path yet.
- `in-progress` — base clone/genericization, alternate-base audit/extend,
  or domain modules are actively underway; not yet feature-complete
  against its `SPEC.md`.
- `feature-complete` — every Functional Requirement in the product's
  `SPEC.md` is implemented; GA-gate hardening (deep artifacts, load
  testing, etc.) may still be outstanding.
- `extending-existing` — the `Live — Extend Only` workflow (ZodiTrack) or
  the alternate-base audit/extend workflow (ZodiBank, ZodiCore,
  ZodiCapital, ZodiYield) — auditing and additively extending a codebase
  that already exists, rather than a from-scratch clone/genericize build.
- `blocked` — cannot safely proceed; see the Flagged Items section for why.

## Product ledger

| Product | Status | Current step | Last updated | Next |
|---|---|---|---|---|
| ZodiTrack | `extending-existing` | Confirmed on disk at `script/public_html/zoditrack/`: native procedural PHP (mysqli, page-routed `.php` files) freight/shipment-tracking site with public tracking-number lookup, customer portal, and a 33-file admin back office (shipments, branches, staff, customers, vendors, invoices, reports). **Domain mismatch found**: `docs/products/ZodiTrack/SPEC.md` was written describing an ITAM/enterprise-asset-tracking tool, which does not match — see the correction notice added to that spec's §0 | 2026-07-31 | Read every file under `admin/` in full, rewrite SPEC.md §1–§7 to describe the real freight-tracking business, then populate a real gap list |
| ZodiBank | `in-progress` | Confirmed on disk at `script/public_html/zodibank/`: Laravel 11 + `nwidart/laravel-modules`, built on "Pay Secure," with `Modules/Agent` and `Modules/Merchant` already present, and Authorize.Net/Flutterwave/CoinGate/CinetPay gateways already in `composer.json`. Freshly `git init`'d (baseline commit, then a second commit for the work below); confirmed via direct code search that `users` has only a single implicit `balance` column (no named/numbered per-type accounts) and `admins` has no branch scoping at all. Built the FDR/DPS/account-number/staff-branch **data and business-logic layer**: `branches` table + `Branch` model; `admins.branch_id` (staff = admin assigned to a branch) + `Admin::branch()`; `bank_accounts` table + `BankAccount` model (multiple named accounts per user: savings/current/fdr/dps, each with its own account number); `fixed_deposits` and `recurring_deposits` (+ `recurring_deposit_installments`) tables/models; `app/Services/BankingService.php` with `generateAccountNumber()` (branch-code-prefixed, sequential, row-locked), `openAccount()`, `openFixedDeposit()` (simple-interest maturity), and `openRecurringDeposit()` (declining-balance simple-interest RD maturity + full installment schedule generation). Verified via tinker: account numbers sequential per branch, FDR maturity 10000@6.5%/12mo → 10650 (correct), DPS maturity 500/mo@6%/12mo → 6195 (correct, matches the declining-balance formula), 12 installments generated, app boots. **UI layer added in a follow-up commit**: `BranchController`/`StaffController` (full CRUD), `BankAccountController` (open/list/view), `FixedDepositController` (open/mature/break-early), `RecurringDepositController` (open/view/pay-installment, auto-matures on final installment) — 24 routes registered under `routes/partials/admin.php`'s existing `auth:admin` group, blade views under `resources/views/admin/banking/`. Verified via tinker: every view renders with real data (an empty `ViewErrorBag` shared manually, since tinker skips the `ShareErrorsFromSession` middleware `@error()` needs), and the actual controller `store`/`mature`/`break`/`payInstallment` methods were called end-to-end (not just unit-tested in isolation): branch creation, distinct per-branch sequential account numbers (MAIN vs DWTN), FDR mature + idempotent double-mature, DPS installment payment with correct balance debit (1000→900). Sidebar navigation is DB-driven via `ManageMenuController` (not static Blade) — wiring menu entries for these pages in was scoped out this pass to avoid risking the existing menu; pages are reachable directly by URL for now. A pre-existing duplicate route name (`admin.currency.exchange.api.config`) in the Pay Secure base itself (not caused by this work) breaks `route:cache` — the app works normally uncached | 2026-08-02 | **Not yet built**: FDR/DPS maturity-processing as a scheduled job (an active FDR/DPS currently only matures when an admin manually clicks Mature, or when the final DPS installment is paid — nothing yet auto-runs on `maturity_date` for FDRs, or flags a DPS installment `missed` after its `due_date` passes with no payment); a `defaulted` DPS status is defined in the schema but nothing yet transitions a recurring deposit into it. Then: wire the new pages into the DB-driven sidebar menu; verify Fincra's exact auth/webhook header names against live docs before implementing FINCRA_INTEGRATION.md's integration; audit Pay Secure's own wallet/ledger against `docs/standards/wallet-system.md`'s invariants; fix the pre-existing `admin.currency.exchange.api.config` duplicate route name if a future pass needs `route:cache` to work |
| ZodiCore | `extending-existing` | **All 22 addon packages installed and confirmed working** (18 unique modules — the other 4 zips are alternate/pre-extracted variants of the same modules, see the resolution note below). Install order followed dependency-free alphabetical-ish order since no addon's own docs stated a hard dependency on another (`Superadmin`, `Accounting` [Advance Accounting v1.3.1 chosen over base v0.8.5], `Essentials`/HRM, `AssetManagement`, `Cms`, `Crm` [With-SaaS-Compatibility variant chosen], `ProductCatalogue`, `AiAssistance`, `Gym`, `Hms`, `InventoryManagement`, `Manufacturing`, `Partners`, `Project`, `Repair`, `Spreadsheet`, `Woocommerce`, `Connector`). Two real latent bugs found in the shipped addon code and fixed (not addon-vs-addon conflicts): (1) Accounting's `2021_08_23_..._add_contact_and_location_id_to_journal_entries_table` migration wrapped DDL `ALTER TABLE` statements in `DB::transaction()`, which MySQL auto-commits around, breaking the migration — fixed by removing the transaction wrapper; (2) Gym's `create_packages_table` migration's `down()` dropped the wrong table (`packages` instead of `gym_packages`) — fixed. A third class of bug surfaced only by running the full `php artisan test` suite (not by individual per-addon smoke checks): Accounting's `Helpers/general_helper.php` and InventoryManagement's `start.php` each declare a global function without (or with a mismatched) `function_exists()` guard, which fatals with "Cannot redeclare" the moment the app boots more than once in the same PHP process (exactly what running the full test suite via `php artisan test` triggers, since Laravel's test runner rebuilds the application per test case) — all three instances found and fixed. `modules_statuses.json` confirmed **unreliable** as either an install-state indicator or a complete scope list: it never reflected real install state at any point in this pass (a stale demo/marketing artifact), lists 6 names with no corresponding addon package at all (`Ecommerce`, `FieldForce`, `InboxReport`, `CustomDashboard`, `ZatcaIntegrationKsa`, `Cheque`), and omits 2 real installed modules (`InventoryManagement`, `Partners`). Full test suite run clean of fatal errors after the fixes; the only remaining test failure is Laravel's own stock `Tests/Feature/ExampleTest.php` (`GET /` expects 200, gets 404) — pre-existing scaffold boilerplate, not caused by any addon. The piracy-provenance concern (`nullcave.club` zip-archive-comment markers, also present on the Connector.zip used in this pass) remains resolved per the product owner's scanned-and-confirmed-clean decision recorded below. **All 3 originally-reported known bugs are now fixed, each with a confirmed root cause found by direct reproduction** (not assumed): (1) cheque due-date — `transaction_payments` had no `cheque_due_date` column at all and `TransactionUtil`'s payment-array builders never included the field; added the column via migration and wired it through both create/edit payment-line paths, plus fixed 3 blade views referencing a translation key from a "Cheque" module that was never actually installed; (2) extra top spacing in several non-English languages — the login/register page's "Register" pill had a hardcoded fixed width sized for English, and longer translations (German "Registrieren", Portuguese "Registration") wrapped to 2 lines inside it, growing the header row taller than the page's hardcoded top-padding reserve; fixed by sizing the pill to its content; (3) customer/supplier creation error — installing Superadmin activated its package/subscription gate (`ModuleUtil::isSubscribed()`) on all business CRUD, but zero packages/subscriptions existed, so every contact-creation attempt was silently short-circuited into a "subscription expired" response; since ZodiCore is single-tenant with no Zodize-to-buyer billing relationship, fixed via a seeder (`SingleTenantSuperadminSubscriptionSeeder`) giving the deployment's one business an unlimited, non-expiring package/subscription, rather than touching Superadmin's own gating code — also fixed a smaller related bug in CRM's `creatContactPerson()` (hashing a null password before checking whether login was requested). **Architectural note**: Superadmin's subscription gate is a multi-tenant SaaS mechanic in tension with the single-tenant model; the seeded package covers every status-only check, but any future feature checking a package's specific *limits* should be reviewed against this same tension | 2026-08-02 | Proceed to the ERP feature-gap analysis (what a complete ERP still needs beyond these 18 unique installed addon modules), now that the install record and all 3 known-bug fixes are accurate and verified; the parallel non-blocking piracy-marker verification task below remains outstanding and unrelated to this pass |
| ZodiCapital | `extending-existing` | Confirmed on disk at `novavest/public_html/core/`: Laravel app, same `assets/`+`core/` structural split as the qfsfountains-lineage convention. Exact existing feature set (app/ contents, migrations) NOT yet audited — only the directory shape and Laravel identity are confirmed | 2026-07-31 | Audit `novavest/core/app/` and `novavest/core/database/migrations/` against `docs/products/ZodiCapital/SPEC.md` to produce a concrete gap list before building new modules — coordinate with ZodiYield's session, same codebase |
| ZodiYield | `extending-existing` | Same novavest/core base as ZodiCapital (see above); same audit-not-yet-done caveat | 2026-07-31 | Same novavest audit as ZodiCapital — do not duplicate; check whether ZodiCapital's session already ran it first |
| ZodiBusiness | `not-started` | Not begun — first in build order (`ROADMAP.md` #1), reference pipeline product | 2026-07-31 | Clone sanitized base to `/home/script/public_html/zodibusiness/`, run `product-genericization-checklist.md` |
| ZodiCommerce | `not-started` | Not begun (`ROADMAP.md` #2) | 2026-07-31 | Blocked behind ZodiBusiness validating the pipeline first |
| ZodiPOS | `not-started` | Not begun (`ROADMAP.md` #3) | 2026-07-31 | Queued |
| ZodiFleet | `not-started` | Not begun (`ROADMAP.md` #4) | 2026-07-31 | Queued |
| ZodiEstate | `not-started` | Not begun (`ROADMAP.md` #5) | 2026-07-31 | Queued |
| ZodiHotel | `not-started` | Not begun (`ROADMAP.md` #6) | 2026-07-31 | Queued |
| ZodiReach | `not-started` | Not begun (`ROADMAP.md` #7) | 2026-07-31 | Queued |
| ZodiMed | `not-started` | Not begun (`ROADMAP.md` #9, renumbered — ZodiCore is now `extending-existing`, not a from-scratch build slot) | 2026-07-31 | Queued |
| ZodiCampus | `not-started` | Not begun | 2026-07-31 | Queued |
| ZodiLaw | `not-started` | Not begun | 2026-07-31 | Queued |
| ZodiBuild | `not-started` | Not begun | 2026-07-31 | Queued |
| ZodiAgro | `not-started` | Not begun | 2026-07-31 | Queued |
| ZodiGov | `not-started` | Not begun | 2026-07-31 | Queued |
| ZodiTrade | `not-started` | Not begun. Fresh Laravel build on sanitized qfsfountains base — dash/Bicrypto and web3chainlink are feature/UX references only, never ported (dash is a Node/pnpm/PM2 monorepo, confirmed not portable) | 2026-07-31 | Queued; dual trading-mode architecture (external API / internal engine) documented in SPEC.md §11.2 |
| ZodiXchange | `not-started` | Same as ZodiTrade — fresh Laravel build, dash/web3chainlink as references only | 2026-07-31 | Queued |
| ZodiChain | `not-started` | New product, promoted from future-expansion this session. Fresh Laravel build on sanitized qfsfountains base. `web3chainlink/public_html/project/` confirmed to be an ordinary Laravel app (not Bicrypto's Node monorepo) with crypto-adjacent composer deps (`flutterwavedev/flutterwave-v3`, `anandsiddharth/laravel-paytm-wallet`, `bacon/bacon-qr-code`) — its `Modules/` contents and README were NOT yet inspected, so its exact feature scope as a reference is unconfirmed | 2026-07-31 | Inspect `web3chainlink/public_html/project/Modules/` and `README.md` before assuming it's a validated reference for any specific ZodiChain module; then queued behind the earlier build-order items |

## ZodiTrack gap list

**Not yet populated in feature-by-feature form.** §0 of
[`docs/products/ZodiTrack/SPEC.md`](./docs/products/ZodiTrack/SPEC.md) now
records what's confirmed present (tracking-number lookup, shipment CRUD,
branches, staff, customers, vendors, invoicing, reporting, notifications,
activity log, settings) and what's still needed before a real gap list is
meaningful (a full rewrite of the spec's Vision/Purpose/Personas sections,
which currently describe the wrong business domain entirely).

## Flagged Items

### 2026-08-02 (RESOLVED) — ZodiCore: all 22 addon packages installed; 3 real bugs found and fixed; `modules_statuses.json` confirmed unreliable

Following the product owner's confirmation to proceed with the standard
`nwidart/laravel-modules` install method (per the resolution below) and
with working Zodize MCP Server access restored, all 22 addon packages
under `modulesfiles/Ultimate POS Addons/` were installed one at a time —
extract to `Modules/<Name>/`, `composer dump-autoload`,
`php artisan module:enable <Name>`, run migrations, lint (`php -l`),
confirm the app still boots, commit — with each addon committed on its
own to `/home/script/public_html/zodicore`'s local git repository
(no GitHub remote configured for that repo; baseline `460086a`,
final commit as of this pass includes the Connector install plus the
test-suite bugfix commit).

**Final unique module count: 18**, not 22 — 4 of the 22 zips are
alternate variants of the same module and only one of each pair was
installed:
- `Accounting` (base, v0.8.5) vs `Advance Accounting` (v1.3.1) — **Advance
  Accounting installed**, the newer/superset variant.
- `Crm` Without-SaaS vs `Crm` With-SaaS-Compatibility — **With-SaaS
  variant installed**, since it is the strict superset.

No addon's own shipped documentation stated a hard dependency on another
addon (checked each "Getting Started" PDF and each package's own
`composer.json`/migration contents for cross-module foreign keys before
installing); install order was therefore not constrained by a dependency
chain, and no addon-vs-addon conflict (route collision, migration
conflict, duplicate table name) was found across all 22 packages.

**Two real bugs in the shipped addon code itself** (not conflicts between
addons) were found and fixed during individual installs:
1. Accounting's `2021_08_23_175321_add_contact_and_location_id_to_journal_entries_table.php`
   wrapped its `ALTER TABLE` DDL statements in `DB::transaction()`; MySQL/
   MariaDB auto-commits around DDL, which broke Laravel's migration
   bookkeeping. Fixed by removing the transaction wrapper and manually
   reconciling the `migrations` table row.
2. Gym's `2024_11_18_150455_create_packages_table.php` migration's
   `down()` called `Schema::dropIfExists('packages')` — the wrong table;
   `up()` correctly created `gym_packages` (no collision with
   Superadmin's own `packages` table), but the rollback would have
   destroyed Superadmin's table instead of Gym's own. Fixed to
   `dropIfExists('gym_packages')`.
3. Spreadsheet's `Resources/lang/nl/lang.php` shipped with a corrupted
   array literal (`'spreadsheet' => 'Spreadsheet',Spreadsheet` — stray
   text appended with no quotes) that a lint check would have caught, but
   didn't, because that commit chained the lint step with `;` instead of
   `&&` before `git commit` — a process mistake, not an addon bug per se.
   Fixed in a follow-up commit; a full `php -l` sweep of all previously
   committed modules afterward found no other syntax errors from the same
   process mistake.

**A third class of bug surfaced only by running the full `php artisan
test` suite** (task step 5), not by the per-addon `php -l`/migrate-status/
`php artisan about` smoke checks used after each individual install:
Laravel's test runner re-bootstraps the entire application once per test
case. `nwidart/laravel-modules`' `Module::registerFiles()` (in
`vendor/nwidart/laravel-modules/src/Module.php`) uses `include` (not
`include_once`) to load each module's `module.json`-declared `files`
array — so any procedurally-declared global function in one of those
files that isn't wrapped in `function_exists()` fatals with "Cannot
redeclare" the moment the app boots a second time in the same PHP
process. Found and fixed:
- `Accounting/Helpers/general_helper.php`: `accounting()` was the one
  function in the file with no guard at all (every sibling function in
  the same file has one).
- `Accounting/Helpers/general_helper.php`: `get_days_past()` was guarded
  by `function_exists('get_date_range')` — the wrong function name (a
  copy-paste bug; `get_date_range` is never declared anywhere, so the
  guard never actually triggered).
- `InventoryManagement/start.php`: `inventorymanagement1()` had no guard.

`InventoryManagement/Helpers/general_helper.php`'s similarly-unguarded
`inventorypos()` was left untouched — confirmed it is dead code, not
referenced by that module's `module.json` `files` array or its
`composer.json` autoload, so it is never procedurally included and can't
trigger this bug.

After these fixes, `php artisan test` runs with no fatal errors; the only
remaining failure is Laravel's own stock `Tests/Feature/ExampleTest.php`
(`GET /` expects HTTP 200, receives 404) — pre-existing framework
scaffold boilerplate present since project creation, unrelated to any of
the 22 addons, and out of scope for this pass to fix.

**`modules_statuses.json` confirmed unreliable**, closing the open
question from the prior pass: it never once reflected the real on-disk
`Modules/` state at any point during this install (a stale artifact,
most plausibly baked into Ultimate POS's own demo/marketing build). Of
its 22 listed names, 6 have no corresponding addon package anywhere in
`modulesfiles/Ultimate POS Addons/` (`Ecommerce`, `FieldForce`,
`InboxReport`, `CustomDashboard`, `ZatcaIntegrationKsa`, `Cheque`), and 2
real installed modules aren't listed in it at all (`InventoryManagement`,
`Partners`). This file must not be trusted as a source of truth for
ZodiCore's module state going forward — `php artisan module:list` against
the live codebase is the only reliable check.

The `nullcave.club` zip-archive-comment marker (see the piracy-provenance
entry immediately below) was also present on `Connector.zip`, installed
in this pass under the already-resolved decision (scanned and confirmed
clean, no re-acquisition) — no additional marker *files* (HTML redirects,
`readme!.html`, etc.) were found inside any of the 22 extracted addons'
actual contents, only the same zip-comment-metadata pattern already
documented.

### 2026-08-02 (RESOLVED by product-owner decision) — ZodiCore: `modulesfiles/Ultimate POS Addons/` sourced from a piracy redistributor ("nullcave.club")

**Resolution**: the product owner reviewed this finding and confirmed the
flagged packages were separately scanned and are clean — proceed
installing from the same files already present in
`modulesfiles/Ultimate POS Addons/`; no re-acquisition from a legitimate
channel is required. This entry is kept in full below for the record
(what was found and why it was flagged), not retracted — the resolution is
a decision on top of the evidence, not a correction of it.

### 2026-08-02 — ZodiCore: session's Zodize MCP Server disconnected mid-task; no addon install performed

The product owner confirmed the piracy-provenance resolution above and
directed this session to proceed with the full 22-addon install (extract →
`composer dump-autoload` → `module:enable` → migrate → build/test →
commit, one addon at a time). Before any addon could be touched, this
session's Zodize MCP Server connection — the only tool providing
filesystem/terminal access to `/home/script/public_html/` — disconnected
(361 tools became unavailable). Per protocol rule 5, this session did not
attempt to fabricate install progress or guess at command output it could
not actually run. **Nothing was extracted, enabled, migrated, or
committed to the live ZodiCore codebase in this pass** — the only change
in this pass is this documentation update, made using this repository's
own (still-available) git tools, not the disconnected VPS access. The
baseline commit `460086a` from the prior pass (confirmed correct method,
before the piracy finding) is the resume point once VPS access is
restored.

### 2026-08-02 — ZodiCore: `modulesfiles/Ultimate POS Addons/` sourced from a piracy redistributor ("nullcave.club"). CRITICAL, STOPPED before extracting anything.

**Severity: supersedes every other ZodiCore item below. Do not extract,
merge, or enable any of these 20 addon packages until this is resolved by
a human decision.**

While preparing to install the addons using the confirmed-correct
`nwidart/laravel-modules` method (extract as `Modules/<Name>/`), direct
inspection of the addon package files themselves found concrete evidence
that they were sourced from a script-piracy redistribution site, not
purchased legitimately:

1. **An HTML redirect file to `https://nullcave.club/`** is present inside
   at least two addon package folders as delivered:
   `Asset-Management-V2.1/Asset-Management-V2.1/nullcave.club.html` and
   `AiAssistance-Module-V2.0/nullcave.club.html`. Content (both identical):
   a bare `<meta http-equiv="refresh" content="0; url=https://nullcave.club/">`
   redirect page.
2. **A second redirect file, `CMS-Module-V1.2/readme!.html`**, redirects to
   the same `https://nullcave.club/` URL — a "readme" that is actually an
   advertisement/redirect for the piracy site, not documentation.
3. **The zip archive comment field itself says "NullCave.club" or
   "NullCave.pro"** on at least 5 of the 17 zipped packages, visible via
   `unzip -l` output before the file listing even starts: `Accounting.zip`
   (base Accounting), `ProductCatalogue.zip`, `Connector.zip`, `Hms.zip`,
   `Spreadsheet.zip`, and `AiAssistance.zip`. This is the tool that
   originally repackaged/redistributed the zip stamping its own branding
   into the archive metadata — a strong, standard signature of "nulled"
   commercial script redistribution.
4. A `grep -rli` sweep for `nullcave|nullphp|cracked|warez` across the
   entire `modulesfiles/Ultimate POS Addons/` directory (including inside
   zip contents via `unzip -p | strings`) found the marker in **at least
   8 of the 20 addon package folders** — meaning this is not an isolated
   file, it's characteristic of the batch as a whole.

**This confirms — under a different specific domain name than originally
reported — the licensing/piracy concern flagged as "unconfirmed" in
`docs/products/ZodiCore/SPEC.md` §11 and this file's earlier flagged item
below** (the original report named `nullphpscript.com`; this audit found
`nullcave.club` instead — same category of concern, now confirmed with
concrete evidence rather than an unconfirmed claim).

**Why this stops the work, not just a note-and-continue:**

- Legally: installing and building on top of software confirmed sourced
  from a piracy redistributor is a genuine licensing risk for whatever
  product ZodiCore becomes.
- Security: "nulled"/cracked commercial PHP scripts are a well-known
  vector for injected backdoors, license-bypass code that phones home, or
  other tampering relative to the vendor's real release — the fact that no
  additional suspicious strings were found in this pass's `grep`/`strings`
  sweep does **not** rule out tampering that doesn't literally contain the
  word "nullcave" (obfuscated code, unfamiliar outbound calls, etc.).
- Scope: since the marker appears across at least 8 of 20 different addon
  packages (different modules, different vendors' original release dates),
  this strongly suggests the entire `modulesfiles/Ultimate POS Addons/`
  directory came from one piracy redistributor's bundle, not that a few
  individual files were tampered with — i.e. this is very likely true of
  all 20, not just the 8 where a marker happened to survive.

**What this pass did NOT do, given this finding**: no addon was extracted
into `Modules/`, no `composer dump-autoload` or `module:enable` was run
against any addon, and no addon-related file was committed. The **only**
ZodiCore commit made in this pass is a clean baseline snapshot of the
codebase exactly as it stood before this investigation (`git init` +
initial commit, hash `460086a`, on the live codebase itself — a rollback
point, not a merge).

**Recommended next step, pending a human decision**: acquire these 22
addon modules through Ultimate POS's legitimate purchase channel
(`https://ultimatefosters.com`) before any of them are installed into a
product ZodiCore is meant to become sellable, and treat the current
`modulesfiles/` directory as unsuitable to build on until replaced with
legitimately-sourced packages.

### 2026-08-02 — ZodiCore: addon install method — two conflicting premises, both contradicted by direct inspection. STOPPED before any merge.

**Severity: blocks Step 2 (merging addons) until a human decides how to
proceed.** No files were extracted or merged into the live app; this is a
pure investigation finding.

**Premise A** (this ledger's own prior entry, from PR #4): the 22 addons
were already installed and active, evidenced by `modules_statuses.json`
having all 22 set `true` and `bootstrap/cache/*_module.php` cache files
existing for each one.

**Premise B** (a later correction, delivered as an instruction in this
session): the addon zips are meant to be extracted directly into the main
app root (`/home/script/public_html/zodicore/` itself) — adding missing
files and updating specific existing files — NOT registered as
self-contained `Modules/<Name>/` packages.

**Both are contradicted by direct inspection in this pass:**

1. **No `Modules/` directory exists anywhere in the live app.**
   `find . -maxdepth 2 -iname Modules -type d` from
   `script/public_html/zodicore/` returns nothing. `ls -la` at the app root
   shows only `modules_statuses.json` and `modulesfiles/` — no `Modules/`.
   This directly contradicts Premise A: the cache files and status JSON are
   evidently stale artifacts (most plausibly baked into Ultimate POS's own
   demo/marketing build, which vendors commonly ship with all
   modules "enabled" for screenshot purposes) that do not reflect this
   deployment's real state. **None of the 22 addons are actually merged
   into this live codebase in any form.**
2. **Every one of the 20 addon packages is a self-contained
   `nwidart/laravel-modules`-shaped package, not loose files for a
   root-directory merge.** Confirmed via `unzip -l` on every zipped addon
   (17 of 20) and direct directory listing of the 3 pre-extracted ones
   (`Asset-Management-V2.1`, `Gym-V0.5`, `Partners-v2.0`): every package's
   entire content sits under one top-level folder matching the module name
   (e.g. `Accounting/composer.json`, `Accounting/Config/`,
   `Accounting/Console/` — same shape for all 20: Accounting, Advance
   Accounting, AiAssistance, Cms, Crm ×2, ProductCatalogue, Essentials,
   Hms, InventoryManagement, Manufacturing, Project, Repair, Spreadsheet,
   Superadmin, Woocommerce, Connector, plus the three pre-extracted ones).
   This is the standard `nwidart/laravel-modules` per-module package shape
   (own `composer.json`, `Config/`, `Console/`), meant to be extracted as
   `Modules/<Name>/` — the same pattern already confirmed in
   `docs/architecture/base-codebase-strategy.md` for the qfsfountains base
   and in ZodiBank's Pay Secure (`Modules/Agent`, `Modules/Merchant`).
   Treating these as "add missing files, update specific existing files in
   the app root" per Premise B would mean manually decomposing 20
   well-formed, self-contained Laravel packages into a scatter of files
   merged into `app/`/`resources/`/`database/` — which is not what their
   own structure suggests, is not how `nwidart/laravel-modules` addons are
   normally installed for this class of product, and risks breaking
   autoloading/namespacing for no confirmed benefit.
3. `.gitignore` at the app root does not exclude `/Modules` — so there is
   no configuration reason a `Modules/` directory would be legitimately
   present but hidden from listing/search; it is simply not there.
4. A "Getting Started" PDF ships alongside each addon zip (e.g.
   `Accounting-Module-For-UltimatePOS-V0.8.5/Getting Started - Accounting
   Module for UltimatePOS.pdf`) that would very likely state the exact
   install method definitively — `pdftotext` was not available in this
   session's terminal to extract its text, so this could not be directly
   confirmed, but the zip structure alone is unambiguous enough to flag
   this without it.

**What this pass recommends, pending explicit confirmation**: the standard
`nwidart/laravel-modules` install method — extract each addon's `<Name>/`
folder as `Modules/<Name>/`, run `composer dump-autoload`, then
`php artisan module:enable <Name>` (or the equivalent migrate/seed steps),
one module at a time, verifying no conflicts before each commit — matches
the confirmed zip structure and the same pattern already used elsewhere in
this handbook. This is a **change from both prior premises**, so it is
recorded here rather than acted on unilaterally. A human/product-owner
decision is needed before any addon is actually extracted into the live
codebase.

### 2026-07-31 — ZodiCore: unconfirmed license/piracy-marker claim (parallel, non-blocking)

The build instructions asked to record, as an open (non-blocking) security
item, a claim that ZodiCore's installed copy's readme file redirects to
`nullphpscript.com`, a script-piracy site marker. A text search for this
string across ZodiCore's codebase in this session returned no matches —
but the search may not have covered every relevant location (binary/vendor
assets, `public/docs/images`, or a runtime redirect that isn't a static
string in a text file). **This does not confirm the claim is false** — it
means this pass could not confirm it either way. A follow-up session MUST:

1. Directly inspect the actual documentation viewer / license-check code
   path (likely under `app/Http/Controllers/Install/` or a licensing
   service class) for any outbound redirect logic, not just grep static
   files.
2. Verify license authenticity through Ultimate POS's legitimate purchase
   channel if a purchase code/verification mechanism exists in the
   codebase.
3. Scan for injected/backdoor code generally (unexpected outbound HTTP
   calls, obfuscated code, unfamiliar cron entries) as a broader precaution
   before ZodiCore is treated as a clean base for a sellable product.

This is explicitly a **parallel task, not a blocker** — addon
conflict-resolution and feature-gap work on ZodiCore may proceed
concurrently, but this item MUST be resolved before ZodiCore reaches its
GA gate (per
[`docs/checklists/production-readiness-checklist.md`](./docs/checklists/production-readiness-checklist.md)).

### 2026-07-31 — ZodiTrack SPEC.md domain mismatch (see also the ledger row above)

Recorded in full in `docs/products/ZodiTrack/SPEC.md` §0. Summarized here
for visibility: the existing spec describes an ITAM/enterprise-asset
-tracking tool; the real, live, currently-resold product is a freight/
shipment-tracking and logistics-brokerage website. Do not use §1–§7 of that
spec to guide any ZodiTrack extension work until they're rewritten.

### 2026-07-31 (resolved) — Build path and base codebases were unreachable

Previously recorded as a blocker: this repository's own git-backed session
had no filesystem access to `/home/script/public_html/` or the source
codebases. **Resolved** — the Zodize MCP Server, added in a later session,
provides real access (workspace root `/home/`), used to perform every
audit finding recorded in this ledger update. Left here for history rather
than deleted, per the principle that this ledger's Flagged Items section is
a record, not just a live TODO list.

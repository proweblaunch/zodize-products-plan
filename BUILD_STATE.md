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
| ZodiBank | `extending-existing` | Confirmed on disk at `script/public_html/zodibank/`: Laravel + `nwidart/laravel-modules`, built on "Pay Secure," with `Modules/Agent` and `Modules/Merchant` already present, and Authorize.Net/Flutterwave/CoinGate/CinetPay gateways already in `composer.json`. Confirmed absent (via code search): FDR, DPS, account-number generation, staff/branch management. FINCRA_INTEGRATION.md spec added, several Fincra API specifics flagged as needing live-docs verification (WebFetch to docs.fincra.com returned HTTP 403 in this session) | 2026-07-31 | Build FDR/DPS/account-number/staff/branch modules fresh; verify Fincra's exact auth/webhook header names against live docs before implementing the integration; audit Pay Secure's own wallet/ledger against `docs/standards/wallet-system.md`'s invariants before building on it |
| ZodiCore | `extending-existing` | Confirmed on disk at `script/public_html/zodicore/`: Laravel + `nwidart/laravel-modules`, built on "Ultimate POS," with 22 addon modules **already installed and active** (confirmed via `modules_statuses.json` all-`true` plus `bootstrap/cache/*_module.php` service-provider caches for every one) — NOT merely staged as zips awaiting install, correcting the original task premise. 3 known bugs open (cheque due-date field, language-specific spacing bug, customer/supplier creation error) — not yet verified fixed. Public front pages confirmed 404; auth/admin routes function | 2026-07-31 | Reconcile SPEC.md §1–§10 against the real Ultimate POS capability (far more than "task tracker + generic records"); resolve addon conflicts; audit for ERP gaps (procurement, BI) beyond the 22 modules; fix the 3 bugs; replace front pages per `docs/standards/frontend-standard.md`; run the parallel license/backdoor security check (see Flagged Items) |
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

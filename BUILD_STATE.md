# Build State Ledger

> The single source of truth for exactly where autonomous/human build
> execution stands, across all 20 Zodize products. This file is the
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

## Environment note (read before assuming any row below is stale)

This ledger was initialized in a documentation-only Claude Code session
whose filesystem does **not** include `/home/script/public_html/` — that
path was checked directly and does not exist in this session's container.
Because of that, **no product's on-disk state could be verified or acted
on when this ledger was created**, including ZodiTrack's claimed live
codebase. Every `status` below reflects only what could be established from
this repository's specs, not a real filesystem audit. Per protocol rule 5,
this is recorded as a flagged item, not guessed past. The next session that
*does* have access to `/home/script/public_html/` MUST perform the
verification step in protocol rule 2 for every product before trusting or
advancing any row below — starting with ZodiTrack, whose audit is the most
time-sensitive since it's already live and being sold.

## Status definitions

- `not-started` — no code exists at this product's path yet.
- `in-progress` — base clone/genericization or domain modules are actively
  underway; not yet feature-complete against its `SPEC.md`.
- `feature-complete` — every Functional Requirement in the product's
  `SPEC.md` is implemented; GA-gate hardening (deep artifacts, load
  testing, etc.) may still be outstanding.
- `extending-existing` — the `Live — Extend Only` workflow: auditing and
  additively extending a codebase that already exists independent of this
  pipeline.
- `blocked` — cannot safely proceed; see the Flagged Items section for why.

## Product ledger

| Product | Status | Current step | Last updated | Next |
|---|---|---|---|---|
| ZodiTrack | `blocked` | On-disk audit against `docs/products/ZodiTrack/SPEC.md` could not be performed — see Flagged Items | 2026-07-31 | A session with real access to `/home/script/public_html/zoditrack/` must run the audit and populate this row's gap list before any extend work starts |
| ZodiBusiness | `not-started` | Not begun — first in build order (`ROADMAP.md` #1), reference pipeline product | 2026-07-31 | Clone sanitized base to `/home/script/public_html/zodibusiness/`, run `product-genericization-checklist.md` |
| ZodiCommerce | `not-started` | Not begun (`ROADMAP.md` #2) | 2026-07-31 | Blocked behind ZodiBusiness validating the pipeline first |
| ZodiPOS | `not-started` | Not begun (`ROADMAP.md` #3) | 2026-07-31 | Queued |
| ZodiFleet | `not-started` | Not begun (`ROADMAP.md` #4) | 2026-07-31 | Queued |
| ZodiEstate | `not-started` | Not begun (`ROADMAP.md` #5) | 2026-07-31 | Queued |
| ZodiHotel | `not-started` | Not begun (`ROADMAP.md` #6) | 2026-07-31 | Queued |
| ZodiReach | `not-started` | Not begun (`ROADMAP.md` #7) | 2026-07-31 | Queued |
| ZodiCore | `not-started` | Not begun (`ROADMAP.md` #8) | 2026-07-31 | Queued |
| ZodiMed | `not-started` | Not begun (`ROADMAP.md` #9) | 2026-07-31 | Queued |
| ZodiCampus | `not-started` | Not begun (`ROADMAP.md` #10) | 2026-07-31 | Queued |
| ZodiLaw | `not-started` | Not begun (`ROADMAP.md` #11) | 2026-07-31 | Queued |
| ZodiBuild | `not-started` | Not begun (`ROADMAP.md` #12) | 2026-07-31 | Queued |
| ZodiAgro | `not-started` | Not begun (`ROADMAP.md` #13) | 2026-07-31 | Queued |
| ZodiGov | `not-started` | Not begun (`ROADMAP.md` #14) | 2026-07-31 | Queued |
| ZodiBank | `not-started` | Not begun (`ROADMAP.md` #15) | 2026-07-31 | Queued |
| ZodiTrade | `not-started` | Not begun (`ROADMAP.md` #16) | 2026-07-31 | Queued |
| ZodiXchange | `not-started` | Not begun (`ROADMAP.md` #17) | 2026-07-31 | Queued |
| ZodiCapital | `not-started` | Not begun (`ROADMAP.md` #18) | 2026-07-31 | Queued |
| ZodiYield | `not-started` | Not begun (`ROADMAP.md` #19) | 2026-07-31 | Queued |

## ZodiTrack gap list

**Not yet populated.** This section exists to hold the feature-by-feature
gap list (spec requirement → present/absent in the live codebase) once a
session with real filesystem access to `/home/script/public_html/zoditrack/`
completes the required audit against
[`docs/products/ZodiTrack/SPEC.md`](./docs/products/ZodiTrack/SPEC.md).
Until then, treat ZodiTrack as `blocked`, not as having no gaps — an
unperformed audit is not the same thing as a clean audit.

## Flagged Items

### 2026-07-31 — Build path and both base codebases unreachable in this session

**Severity: blocks all of Part 2 (autonomous build execution) and the
ZodiTrack audit specifically.**

This session's container filesystem does not contain
`/home/script/public_html/` (checked directly: does not exist), nor does it
contain the two codebases this handbook's architecture docs are audited
from (`/home/qfsfountains/public_html` and `/home/zodize/public_html` —
both checked directly in an earlier session on this same repository and
also do not exist here). This session has access only to this git
repository (`zodize-products-plan`).

Consequences recorded here rather than guessed past:

- **ZodiTrack's on-disk audit could not be performed.** Its "already live"
  claim could not be verified, and no gap list could be produced, because
  there is nothing to inspect. Marking it `blocked` rather than fabricating
  an audit result.
- **No product build work could begin.** Part 2's build execution
  (clone → genericize → bridge → extend, per product) requires the base
  codebase and the target path, neither of which exist in this session.
  Writing product code without them would mean inventing a "base codebase"
  from scratch rather than genericizing the real, audited one this
  handbook describes — which would silently violate
  [`base-codebase-strategy.md`](./docs/architecture/base-codebase-strategy.md)'s
  entire premise and produce something that doesn't match what a buyer
  actually receives.

**What would resolve this**: a session (or an addition to this session's
environment) with actual access to `/home/script/public_html/` and the two
source codebases. Until that access exists, the next thing to do is
**not** to start writing Laravel code that approximates the base engine
from memory — it's to confirm where that access lives and connect to it.

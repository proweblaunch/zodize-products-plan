# Build & Deployment Paths

> Where product codebases actually live, at each stage of their life. This
> document distinguishes two different things this handbook could mean by
> "deployment," which must never be conflated.

## Two distinct locations, two distinct purposes

1. **Build-time location (this document)**: where Claude Code (or any
   Zodize engineer) does the actual work of cloning the base codebase,
   running the genericization checklist, and implementing a product's
   domain modules, before that product is ever sold to a buyer.
2. **Sale-time location** (see
   [`single-tenant-deployment-model.md`](./single-tenant-deployment-model.md)):
   where a buyer's own purchased copy of the finished source code runs,
   on hosting Zodize does not operate or have access to. That document
   describes what happens *after* a sale. This document describes what
   happens *before* one, during active development.

## Build working-directory convention

Every product's codebase is built and lives at:

```
/home/script/public_html/<product-slug>/
```

where `<product-slug>` is the product's catalog name in lowercase, exactly
as listed in [`../../PRODUCT_CATALOG.md`](../../PRODUCT_CATALOG.md) —
e.g. `/home/script/public_html/zodicore/`,
`/home/script/public_html/zodibank/`, `/home/script/public_html/zoditrack/`.
This is the on-disk convention every session building or extending a
product MUST follow, so that any session (fresh or resumed) can locate a
product's code without guessing or re-deriving a path.

Each product directory under this convention mirrors the base codebase's
own internal split, per
[`base-codebase-strategy.md`](./base-codebase-strategy.md#directory-structure):

```
/home/script/public_html/<product-slug>/
├── assets/     # public static assets
└── core/       # the Laravel application (app/, config/, database/, lang/, resources/, routes/)
```

## What lives outside this convention

This repository (`zodize-products-plan`) never contains product source
code — see [`../../ROADMAP.md`](../../ROADMAP.md#non-goals). It is the
documentation and specification set a build session reads *before* writing
any code at the path above. [`BUILD_STATE.md`](../../BUILD_STATE.md) is the
ledger that connects the two: it tracks, per product, what has actually
been built at its `/home/script/public_html/<product-slug>/` path.

## Verifying the path before relying on it

A session MUST NOT assume `/home/script/public_html/` exists or is
reachable just because this document describes the convention — environment
availability varies session to session (a documentation-only session working
purely in this git repository, for instance, may have no access to that
filesystem at all). Before beginning or resuming build work on any product,
confirm the path is actually reachable and, for a product marked
`in-progress` or later in `BUILD_STATE.md`, confirm the on-disk state
genuinely matches what the ledger claims — per
[`BUILD_STATE.md`](../../BUILD_STATE.md)'s protocol, trust the filesystem
over the ledger when they disagree, and if the path cannot be verified at
all, stop and flag it rather than guessing or fabricating progress against
a location that cannot be inspected.

## Related standards

- [`base-codebase-strategy.md`](./base-codebase-strategy.md)
- [`single-tenant-deployment-model.md`](./single-tenant-deployment-model.md)
- [`product-genericization-checklist.md`](./product-genericization-checklist.md)
- [`../../BUILD_STATE.md`](../../BUILD_STATE.md)

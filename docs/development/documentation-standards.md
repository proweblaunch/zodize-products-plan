# Documentation Standards

## Categories of documentation

| Type | Audience | Location |
|---|---|---|
| Handbook standards | Zodize engineers, AI agents | This repository, `docs/` |
| Product specifications | Zodize engineers, AI agents | `docs/products/<product>/SPEC.md` |
| API reference | External developers/integrators | Generated from OpenAPI, published via [documentation-template.md](../templates/documentation-template.md) |
| End-user documentation | Customers using the product | Product's own docs site, scaffolded per the same template |
| Runbooks | On-call engineers | `docs/products/<product>/runbooks/` once implementation begins |

## Writing standard (applies to all categories)

- Imperative/declarative voice for standards ("Services MUST..."),
  task-oriented voice for user-facing docs ("To create an invoice...").
- No placeholders, no "TBD" outside an explicit `## Open Questions` or
  `## Roadmap` section, per [CONTRIBUTING.md](../../CONTRIBUTING.md).
- Every code example is runnable/accurate as written, not illustrative
  pseudo-code presented as real — if it's pseudo-code, label it as such.
- Cross-reference with relative links; do not duplicate content that already
  has a canonical home elsewhere in the handbook.

## API documentation

Auto-generated from the OpenAPI spec ([api-standards.md](./api-standards.md#documentation)),
augmented with hand-written guides for multi-step integration flows
(webhooks setup, OAuth flow, common recipes). Every endpoint's description,
parameter docs, and example request/response are authored in the OpenAPI
spec itself — never hand-maintained separately, to prevent drift.

## User-facing documentation requirement

A product is not Production Ready
([definition-of-production-ready.md](../quality/definition-of-production-ready.md))
until it has: a Getting Started guide, a guide per major module, an FAQ, and
a changelog — matching the information architecture in
[documentation-template.md](../templates/documentation-template.md).

## Keeping documentation current

- A PR that changes user-visible behavior updates the relevant user-facing
  doc in the same PR — documentation drift is treated as a bug.
- A PR that changes API surface updates the OpenAPI spec in the same PR;
  CI's contract diff check ([api-standards.md](./api-standards.md#change-safety))
  makes this enforceable, not just aspirational.
- Handbook documents include no version number in the filename; history is
  tracked via git and summarized in [CHANGELOG.md](../../CHANGELOG.md).

## Diagrams

Architecture and data-flow diagrams are authored as Mermaid code blocks
directly in markdown (not external image files) so they render in GitHub,
stay diffable in PRs, and never go stale relative to an external tool.

# Documentation Site Template

Every Zodize product ships an end-user and developer documentation site,
separate from the marketing site's `/docs` link-out target. This document
specifies the scaffold. A product customizes the actual guide content; it
does not customize the information architecture or the API reference
generation pipeline.

## Directory structure

```
docs-site/
  content/
    getting-started/
      index.md
      installation.md
      quickstart.md
    guides/
      <guide-slug>.md
    api-reference/
      openapi.yaml            # generated, see below — never hand-edited
      index.md
    changelog/
      index.md                # generated from CHANGELOG.md, see release-template.md
    faq/
      index.md
  resources/js/
    pages/
      DocsHome.vue
      DocsPage.vue
      ApiReference.vue
      Changelog.vue
      Faq.vue
    components/
      DocsSearch.vue
      VersionSwitcher.vue
      CodeSample.vue
  routes/
    docs.php
```

## Information architecture

Every product's documentation site MUST have exactly these top-level
sections, in this order in primary navigation:

1. **Getting Started** — installation/setup, first successful action
   ("quickstart"), and core concepts. This section MUST get a first-time
   user to one completed action in under 10 minutes of reading.
2. **Guides** — task-oriented, one guide per user goal (e.g. "Set up SSO",
   "Configure webhooks"). Guides MUST NOT be organized by internal module
   name; they are organized by what the reader is trying to accomplish.
3. **API Reference** — auto-generated from the product's OpenAPI spec (see
   below). MUST NOT be hand-written prose describing endpoints.
4. **Changelog** — auto-generated from `CHANGELOG.md` per
   [release-template.md](./release-template.md); MUST list every customer-facing
   change with the version and date.
5. **FAQ** — a flat list of question/answer pairs, sourced from actual
   support tickets, reviewed quarterly for staleness.

A product MAY add sections (Tutorials, Integrations) but MUST NOT remove or
reorder the five above.

## Versioned docs

A product MUST version its documentation once it has shipped a breaking API
change (see the versioning policy in [api-template.md](./api-template.md)).
`VersionSwitcher.vue` MUST be present in the site shell from day one, even
when only one version exists, so that adding a second version requires no
layout change. Each documented version MUST be reachable at
`/docs/{version}/...` with the latest stable version served at the
unversioned root path as a redirect target, never duplicated content.

## Search

The documentation site MUST ship full-text search (`DocsSearch.vue`) covering
Getting Started, Guides, API Reference, and FAQ content. Search MUST be
available with zero network round-trip latency beyond the initial index
fetch — client-side indexed search is the default; a product MUST NOT ship
documentation without search live at GA.

## Code sample standards

Every code sample rendered via `CodeSample.vue` MUST:

- Be runnable as written — no pseudocode in a fenced code block claiming to
  be a real language.
- Include a language tag for syntax highlighting.
- Show at least a `curl` example and one SDK example (where a Zodize SDK
  exists) for every API Reference endpoint sample.
- Never embed real API keys or secrets; use the literal placeholder
  `YOUR_API_KEY`, styled distinctly so it reads as a placeholder, not a
  copy-pasteable value.

## API reference auto-generation

`content/api-reference/openapi.yaml` MUST be generated from the product's
live route/controller annotations at build time, per the authoritative
process in [`../development/api-standards.md`](../development/api-standards.md).
It MUST NOT be maintained by hand — a hand-maintained OpenAPI file drifts
from the actual API and is treated as a documentation defect. The API
Reference page renders this spec directly; guide authors MUST NOT copy
endpoint signatures into prose guides — link to the reference instead.

## What the shared component library provides vs. what a product customizes

The shared frontend component library every product's codebase includes
(see
[`../architecture/frontend-backend-bridge.md`](../architecture/frontend-backend-bridge.md))
provides: `DocsSearch.vue`, `VersionSwitcher.vue`, `CodeSample.vue`, the
OpenAPI generation build step, and the changelog-to-docs sync job. These
ship inside the product's own codebase, not as a call to another Zodize
product at runtime.

A product customizes: all content under `content/getting-started/` and
`content/guides/`, and the FAQ entries. A product MUST NOT hand-write API
Reference content — any gap in the generated reference is fixed by improving
route/controller annotations, not by adding prose to `api-reference/index.md`
beyond a short introduction.

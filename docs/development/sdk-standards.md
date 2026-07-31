# SDK Standards

## Languages and coverage

Every product's API ships an official SDK for, at minimum: JavaScript/TypeScript
(Node + browser), PHP, and Python. Additional languages are added based on
customer demand, tracked per-product in that product's spec roadmap.

## Generation strategy

SDKs are generated from the product's OpenAPI 3.1 specification (see
[api-standards.md](./api-standards.md#documentation)) using a consistent
generator pipeline, then hand-polished for idiomatic ergonomics (typed
response models, pagination iterators, webhook signature verification
helpers). Generated code and hand-written ergonomic wrappers are kept in
clearly separated directories so regeneration never clobbers hand-written
code.

## Required SDK capabilities

- Authenticated client construction from an API token or OAuth2 credentials.
- Automatic retry with exponential backoff on 429 and 5xx responses,
  respecting `Retry-After`.
- Typed request/response models matching the OpenAPI schema exactly.
- Cursor-based pagination exposed as a native iterator/async-iterator, not a
  manual cursor-passing loop.
- A `webhooks.verifySignature()`-equivalent helper implementing
  [webhook-standards.md](./webhook-standards.md#signing-and-verification).
- Structured error types mapping to the `error.code` taxonomy in
  [api-standards.md](./api-standards.md#requestresponse-envelope), so
  consumers can `catch (ZodizeApiError e) { switch(e.code) ... }` rather than
  string-matching messages.

## Versioning and release

SDK major versions track API major versions. SDK minor/patch releases follow
[versioning-releases.md](./versioning-releases.md) semantic versioning and
are published to the standard package registry per language (npm, Packagist,
PyPI) with automated release notes generated from the OpenAPI diff.

## Documentation

Every SDK ships with a quickstart, full method reference (auto-generated from
the OpenAPI spec's descriptions — descriptions are written once, in the spec,
and propagate everywhere), and runnable examples for the top 10 use cases per
product, published via [documentation-template.md](../templates/documentation-template.md).

## Testing

SDKs run a contract test suite against a mocked API server built from the
same OpenAPI spec on every release, plus a smoke test against a live staging
environment before publish.

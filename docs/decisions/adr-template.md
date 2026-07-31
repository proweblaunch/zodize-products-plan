# ADR Template

This is the standard template for an Architecture Decision Record (ADR) in
this handbook. Use it whenever [`../../CONTRIBUTING.md`](../../CONTRIBUTING.md)
requires an ADR — most commonly, a breaking change to an existing standard.
Copy this file's structure into a new file; do not link to this template
from a real ADR in place of following its structure.

## File naming and numbering

- ADRs live in `docs/decisions/` and are named `adr-XXXX-title-slug.md`,
  where `XXXX` is a zero-padded, monotonically increasing four-digit number
  (`0001`, `0002`, ... `0010`, ...) and `title-slug` is a short, lowercase,
  hyphenated summary of the decision (e.g. `adr-0002-adopt-ulid-primary-keys`).
- Numbers are never reused, even if an ADR is later superseded or the
  decision reversed — the historical record is append-only.
- Determine the next number by finding the highest existing `adr-XXXX-*.md`
  file in this directory and incrementing it. `0001` is reserved for
  [`0001-record-architecture-decisions.md`](./0001-record-architecture-decisions.md),
  the ADR that establishes this process.

## Status lifecycle

An ADR's `Status` field MUST be one of the following, and MUST be kept
current as the decision's real-world state changes:

- **Proposed** — under discussion, not yet in force. A PR containing a
  `Proposed` ADR may be merged to enable discussion, but no other document
  may treat it as a ratified standard until it moves to `Accepted`.
- **Accepted** — ratified and in force. Every other document in this
  handbook MUST be consistent with an `Accepted` ADR.
- **Deprecated** — no longer recommended for new work, but not yet fully
  replaced; existing implementations following it are not required to
  migrate immediately. The ADR MUST state what to do instead.
- **Superseded by adr-YYYY** — replaced entirely by a later ADR. The
  superseding ADR's number MUST be given explicitly, and the superseding
  ADR MUST link back to this one.

A status change MUST be made by editing the `Status` field in place; the
ADR's `Title`, `Context`, and original `Decision` sections are never
rewritten after acceptance — if the decision changes, write a new ADR that
supersedes this one, preserving the historical record of what was actually
decided and when.

## Template

Copy everything below this line into the new ADR file.

---

# ADR-XXXX: {Title}

## Status

{Proposed | Accepted | Deprecated | Superseded by adr-YYYY-title-slug}

## Context

{What is the problem or forcing function that makes a decision necessary?
Describe the situation factually and neutrally — this section should let a
future reader understand why the decision was needed without already
knowing the answer. Reference the specific document(s) this ADR will affect
via relative links.}

## Decision

{State the decision clearly and specifically, in the imperative voice used
throughout this handbook ("Zodize products MUST..."). This is the part of
the ADR that other documents will cite and depend on.}

## Consequences

{What becomes easier or harder as a result of this decision? Include both
positive and negative consequences honestly — an ADR that lists only
benefits has not done the analysis. Note any follow-up work this decision
creates (e.g. "existing product specs using the prior pattern must be
migrated; tracked in each product's SPEC.md#roadmap").}

## Alternatives Considered

{List the realistic alternatives that were evaluated and why each was not
chosen. An ADR with no alternatives listed reads as if no analysis was done
— even a two-line dismissal of an alternative is more useful than silence.}

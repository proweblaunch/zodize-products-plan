# Code Review Standards

## Purpose of review

Review exists to catch defects, verify adherence to this handbook, and
spread knowledge — not to gatekeep style, which is already enforced by
automated tooling ([pr-standards.md](./pr-standards.md#required-checks-before-requesting-review)).
A reviewer should never comment on something Pint/ESLint/PHPStan already
enforces.

## Reviewer responsibilities

A reviewer must check, in order of priority:

1. **Correctness against the spec** — does this match the relevant
   [product spec](../products/) section or handbook standard it claims to
   implement?
2. **Security** — authorization checks present and correct, tenant scoping
   present, no secrets, no injection risk (SQL/XSS/SSRF), per
   [security-standards.md](../security/security-standards.md).
3. **Test coverage of the mandatory cases** in
   [testing-standards.md](./testing-standards.md#non-negotiable-test-cases).
4. **Data integrity** — migrations are safe per
   [migration-seeder-standards.md](./migration-seeder-standards.md),
   no risk of data loss.
5. **Maintainability** — is this the simplest correct solution, per
   [engineering-principles.md](./engineering-principles.md); no premature
   abstraction, no missing necessary abstraction.
6. **UX/design system adherence** for any UI change, per
   [design-system/](../design-system/components.md) and
   [standards/](../standards/ux-principles.md).

## Response time expectations

First review response within one business day. A PR blocked on review for
longer than two business days should be escalated (ping in the team channel,
not silently left) — slow review is a process defect to fix, not a norm to
accept.

## Giving feedback

- Distinguish blocking ("this must change before merge — here's why") from
  non-blocking ("consider," "nit:") explicitly in the comment.
- Every blocking comment states the concrete failure scenario it prevents,
  not just a stylistic preference ("this will double-charge the customer if
  the job retries" beats "I'd do this differently").
- Approve with comments when only non-blocking feedback remains — do not
  hold a PR hostage to optional suggestions.

## AI-assisted review

AI code review tools (including Claude) may perform an initial pass, but a
human code owner's approval is still required per
[pr-standards.md](./pr-standards.md#review-requirements). AI review findings
are treated as input to the human reviewer, not a substitute for one. See
[ai-coding-standards.md](./ai-coding-standards.md).

## Escalation and disagreement

If author and reviewer disagree after one round of discussion, escalate to
the module's tech lead rather than looping indefinitely in PR comments. The
resolution is recorded as an ADR if it sets a new precedent
([docs/decisions/](../decisions/adr-template.md)).

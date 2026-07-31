# Feature Flags

## Purpose

Feature flags decouple deploy from release, per
[engineering-principles.md](./engineering-principles.md) principle 8
(reversibility over speed). They are the mechanism that lets incomplete work
merge to `main` continuously ([git-workflow.md](./git-workflow.md)) and lets
risky changes roll out gradually ([ci-cd-standards.md](../quality/ci-cd-standards.md)).

## Flag types

| Type | Purpose | Lifetime |
|---|---|---|
| Release flag | Hide incomplete/in-progress work | Removed once feature is fully rolled out (target: within one release cycle) |
| Ops/kill-switch flag | Disable a feature instantly if it misbehaves in production | Long-lived, tied to a specific risky capability |
| Plan/entitlement flag | Gate a feature by subscription tier | Long-lived, tied to billing/plan configuration |
| Experiment flag | A/B test a change | Removed once the experiment concludes and a decision is made |

## Implementation standard

- Flags are evaluated server-side (a single source of truth) and exposed to
  the frontend via the authenticated user/session payload — the frontend
  never re-implements flag logic independently.
- Flag checks are scoped: global (the whole deployment), per-company/branch
  on a product with multi-company/multi-branch scoping (see
  [localization-i18n.md](../standards/localization-i18n.md#multi-company--multi-branch-data-scoping)),
  per-user, or percentage rollout.
- Every flag is defined with an owner, a creation date, and an expected
  removal date at creation time — a flag without a removal plan is a defect
  waiting to happen ("flag debt").

## Release flags must be removed

A release flag that has been fully rolled out (100%, no scope excluded) for
more than one release cycle is deleted along with its dead code branch in a
dedicated cleanup PR — flags are not permanent code paths by default.

## Testing

Both states of a release flag (on/off) are covered by the test suite while
the flag is active, per [testing-standards.md](./testing-standards.md); once
a flag is fully rolled out and removed, only the resulting single code path
needs coverage.

## Entitlement flags and RBAC

Plan/entitlement flags are distinct from RBAC permissions
([rbac-permissions.md](../security/rbac-permissions.md)): RBAC answers "is
this user allowed," entitlement flags answer "does this deployment's
licensed plan/edition include this." Both are checked — a user can have the
`reports.export` permission and still be blocked if the business's licensed
plan doesn't include export.

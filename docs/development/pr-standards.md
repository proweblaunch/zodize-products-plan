# Pull Request Standards

## Size and scope

A PR does one thing. Target under ~400 changed lines excluding
generated/lockfiles and tests; a larger PR is split by vertical slice
(e.g. "add model + migration" then "add endpoint" then "add UI") using
feature flags ([feature-flags.md](./feature-flags.md)) to keep `main`
deployable between slices.

## Required PR description contents

1. **What and why** — the problem being solved, not just a restatement of
   the diff.
2. **How to verify** — exact steps a reviewer can take to confirm the change
   works (commands to run, screens to click through).
3. **Screenshots/recordings** for any UI change, using the actual rendered
   UI, in both light and dark theme where the change touches shared
   components ([dark-theme.md](../design-system/dark-theme.md)).
4. **Migration notes** if the PR includes a schema change, per
   [migration-seeder-standards.md](./migration-seeder-standards.md).
5. **Linked issue/spec section** — every PR traces back to a product spec
   section or a handbook document it implements or amends.

## Required checks before requesting review

All of the following must be green before a human reviewer is requested
(see [ci-cd-standards.md](../quality/ci-cd-standards.md)):

- Lint/format (Pint, ESLint/Prettier)
- Static analysis (PHPStan/Larastan, `vue-tsc`)
- Unit + feature + component test suites
- Contract test against the OpenAPI spec, if API surface changed
- Accessibility check (axe-core), if UI changed

## Review requirements

See [code-review-standards.md](./code-review-standards.md) for reviewer
responsibilities. At minimum: one approval from a code owner of the touched
module; two approvals for changes to `docs/security/`,
`docs/architecture/`, database schema of a shared platform table, or any
authentication/authorization code path.

## Merge policy

Squash-merge only, PR title becomes the squash commit message in
Conventional Commits format ([git-workflow.md](./git-workflow.md)). Merging
is blocked automatically if required checks are red — no admin override
without a documented, logged exception for a declared emergency.

## Full checklist

See [pr-checklist.md](../checklists/pr-checklist.md) for the enforced
checkbox list used in the PR template.

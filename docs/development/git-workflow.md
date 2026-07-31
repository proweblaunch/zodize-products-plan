# Git Workflow & Branch Strategy

## Branching model

Trunk-based development with short-lived feature branches:

- `main` is always deployable. Every commit on `main` has passed full CI.
- Feature branches: `type/short-description` (e.g. `feat/invoice-void-flow`,
  `fix/webhook-retry-loop`, `chore/upgrade-laravel`), branched from `main`,
  merged back via PR within a few days — long-lived branches are a smell
  that indicates a feature should be split or flagged.
- Release branches (`release/2026.08`) are cut only for products on a
  scheduled release cadence rather than continuous deployment; see
  [release-template.md](../templates/release-template.md).
- No direct commits to `main`. No force-push to `main`, ever.

## Commit conventions

Conventional Commits format: `type(scope): summary`, types limited to
`feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `security`.
Commit body explains **why**, not what (the diff already shows what).
Breaking changes are flagged with `!` after type/scope and a `BREAKING
CHANGE:` footer.

## Feature flags over long-lived branches

Incomplete work merges to `main` behind a feature flag
([feature-flags.md](./feature-flags.md)) rather than living on a branch for
weeks. This keeps integration continuous and avoids painful merges.

## Pull requests

See [pr-standards.md](./pr-standards.md) for the full PR contract. In
summary: small, reviewable, single-purpose PRs; CI green before requesting
review; squash-merge to `main` with a clean, conventional-commit-formatted
merge commit message.

## Tagging and releases

Tags follow semantic versioning (`v2026.8.1`) per
[versioning-releases.md](./versioning-releases.md), created only from `main`
after CI passes on the exact commit being tagged.

## Handling hotfixes

A production incident fix branches from the deployed tag
(`hotfix/<tag>-description`), goes through the same PR/CI gate (no
`--no-verify`, no skipped review even under pressure — see
[code-review-standards.md](./code-review-standards.md)), merges to `main`,
and is cherry-picked to any active release branch if applicable.

## Repository hygiene

- `.gitignore` excludes all environment files, build artifacts, and IDE
  config; secrets never enter git history — a leaked secret is rotated
  immediately and the incident is logged per
  [security-standards.md](../security/security-standards.md), history
  rewriting is not treated as sufficient remediation on its own.
- Large binary assets (design files, sample media) are not committed to
  application repositories; use the appropriate asset storage referenced in
  [deployment-template.md](../templates/deployment-template.md).

# AI Coding Standards

Zodize builds with AI coding agents (including Claude Code) as first-class
contributors. This document defines how AI-generated work is held to the
same bar as human-authored work — never a lower one.

## This handbook is the agent's specification

An AI coding agent building a Zodize product should need no context beyond:
this handbook, the target product's `docs/products/<product>/SPEC.md`, and
the existing codebase. If an agent has to guess, the gap belongs in this
handbook — see [engineering-principles.md](./engineering-principles.md)
principle 7 ("Specs before code"). When an agent identifies such a gap, it
must document the assumption made and flag it for follow-up rather than
silently guessing and moving on.

## Prompt standards

- Prompts/instructions given to an AI agent for implementation work should
  reference specific handbook sections and spec sections by path, not
  restate them from memory — this keeps the agent grounded in the current
  version of the standard rather than a paraphrase that can drift.
- Multi-step implementation work is decomposed into reviewable units
  matching [pr-standards.md](./pr-standards.md#size-and-scope) — an agent is
  not asked to "build the whole invoicing module" in one shot; it is asked to
  build it in spec-traceable slices.
- Ambiguous or high-stakes decisions (schema design affecting multiple
  modules, security-sensitive logic, anything touching
  `docs/security/` or `docs/architecture/`) require the agent to surface the
  decision for human confirmation before proceeding, rather than picking
  silently.

## Required agent behavior on every implementation task

1. Read the relevant handbook standards and product spec section before
   writing code.
2. Follow [coding-standards-php-laravel.md](./coding-standards-php-laravel.md)
   / [coding-standards-laravel-frontend.md](./coding-standards-laravel-frontend.md) exactly — an
   agent's stylistic preference never overrides a written standard.
3. Write the tests required by
   [testing-standards.md](./testing-standards.md#non-negotiable-test-cases)
   as part of the same unit of work, not as a follow-up.
4. Run linting, static analysis, and the test suite locally before
   presenting work as complete — "should pass CI" is not a substitute for
   "passes CI."
5. Never disable a lint rule, skip a test, or add a suppression to make a
   check pass, without surfacing that explicitly as a flagged decision.

## Review of AI-generated code

AI-generated PRs go through the exact same
[code-review-standards.md](./code-review-standards.md) process as
human-authored PRs, with one addition: the reviewer explicitly verifies the
mandatory test cases actually test the described scenario (agents
occasionally produce a test that exercises the code path without truly
asserting the behavior — reviewers check assertions, not just test names).

## What AI agents must not do autonomously

- Modify `docs/security/`, `docs/architecture/`, or any file in
  `docs/decisions/` without human sign-off (these require an ADR per
  [CONTRIBUTING.md](../../CONTRIBUTING.md)).
- Merge their own PRs, disable CI checks, or force-push to `main`.
- Introduce a new third-party dependency without noting it in the PR
  description for reviewer awareness (license, maintenance status, bundle
  size impact).

## Continuous improvement of this document

When an AI agent repeatedly makes the same category of mistake across
products, that is a signal to add a rule here or to a relevant standard
document, not to rely on the agent "remembering" from one session to the
next — sessions do not share memory, the handbook is the memory.

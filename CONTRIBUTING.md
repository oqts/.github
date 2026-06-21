# Contributing to OQTS

This guide applies org-wide. It documents how we work across every OQTS repository so that the codebase stays clean, reviewable, and secure as the society grows.

## Who can contribute

During the foundational build, the **core team** (`@oqts/core`) builds and maintains all infrastructure repos. Strategy Team and project contributors are added per-repository as those avenues open — see [Requesting a new repository](#requesting-a-new-repository).

## Branching & pull requests

We work on branches and merge through pull requests. **No one pushes directly to `main`.**

1. Branch off `main`: `git checkout -b your-name/short-description`
2. Make focused commits — one logical change per PR; don't mix unrelated edits.
3. Open a PR into `main` and fill in the template.
4. A **code owner review** is required (auto-requested via `CODEOWNERS`), plus **at least one approval**.
5. Once approved and checks pass, merge. Force-pushes and branch deletion on `main` are blocked.

> Branch protection is enforced by rulesets. Org owners hold a break-glass bypass for bootstrapping; regular members always go through review.

## Commit messages

Keep them short and imperative: `Add order validation for short sells`, not `added stuff`. Reference an issue where relevant (`Fixes #12`).

## Never commit secrets

This is the one rule with no exceptions.

- No credentials, tokens, API keys, deploy keys, or `.env` files in any repo — public **or** private.
- Secrets live in the org secrets manager / Actions secrets, scoped to the repo that needs them.
- The shared `.gitignore` already excludes `.env`, `*.pem`, `*.key`, and similar — don't override it.
- If you commit a secret by accident, treat it as compromised: **rotate it immediately** and tell an owner. Removing it from history is not enough.

## Code owners

Each repo has a `CODEOWNERS` file. The relevant owner is automatically requested on any PR touching their paths, and their review is required before merge. If you're added as an owner for a path, you're responsible for reviewing changes to it.

## Requesting a new repository

Repository creation is restricted to org owners so the namespace stays governed. To propose a new research or project repo:

1. Open a **[Request a new repository](https://github.com/oqts/.github/issues/new/choose)** issue.
2. Fill in the name, tier, visibility, lead, and purpose.
3. An owner creates it with the correct `tier`, team access, and branch protection in one step.

Research and project repos are public by default and owned by their project team — the core infrastructure repos stay separate.

## Questions

Reach the core team at **[oqts@oqts.org](mailto:oqts@oqts.org)**.

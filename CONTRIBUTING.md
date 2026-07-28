# Contributing to QYNE

Org-wide workflow for **every** QYNE repo. Read this and the
[Coding Standards](CODING_STANDARDS.md) before writing code. Each repo's own
`README.md` / `AGENTS.md` covers its **stack-specific** setup and rules; this
covers the parts that are the same everywhere.

## Prerequisites

- **Node** per the repo's `.nvmrc` (currently 20 for the standalone repos, 22 for
  the `qyne-app` monorepo) — `nvm use`.
- The repo's package manager: **npm** (`package-lock.json`) for the standalone
  repos, **pnpm** for the `qyne-app` monorepo.
- Copy `.env.example` → `.env` and fill local values. **Never commit `.env`.**

`npm install` / `pnpm install` also wires the git hooks (the `prepare` script).

## Git workflow

Two long-lived branches, each mapped to an environment:

| Branch | Environment | How code arrives |
|---|---|---|
| `main` | **production** | release PR from `develop` |
| `develop` | **staging** | squash-merged feature PRs |

Work branches off `develop`:

| Branch | Purpose |
|---|---|
| `feat/*` | new capability |
| `fix/*` | bug fix |
| `chore/*` | tooling, deps, config |
| `hotfix/*` | urgent production fix — branch off `main`, then back-merge to `develop` |

- **Conventional commits** (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`,
  `test:`, `ci:`).
- **Commit as `qyne-dev <support@qyne.one>`** — never a personal/other-org
  identity (`git config user.name "qyne-dev" && git config user.email "support@qyne.one"`).
- Open a PR into `develop`; it must pass CI and get a review.
- **The maintainer merges — do not self-merge.** Prefer **squash-merge**.
- A release is a PR from `develop` → `main`.

### Protected branches (interim, free plan)

`main` and `develop` are **never pushed to directly** — always via PR. Server-side
branch protection needs a paid GitHub plan; until then a client-side hook enforces
it locally:

- Enabled automatically on install (the `prepare` script runs
  `git config core.hooksPath .githooks`). To enable manually, run that once.
- `.githooks/pre-push` blocks direct pushes to `main` and `develop`.
- Emergency bypass (use deliberately): `git push --no-verify`.

## Pull requests

- One concern per PR; keep it small and reviewable.
- Fill in the PR template: what, why, how verified; screenshots for UI; link the
  issue.
- **Green CI is required** — format, lint, typecheck, test, build.
- **Keep the README current** — fix it in the same PR if the change makes it
  wrong, or state "README: no change needed."

## Keeping docs current

The `README.md` is the front door to each repo and **must never drift from
reality**. Backend changes to a route contract must also update the OpenAPI /
Swagger annotations in the same PR (they are part of the endpoint, not separate
docs). If a change genuinely touches neither, say so in the PR.

## Definition of done

See [Coding Standards §11](CODING_STANDARDS.md#11-definition-of-done), plus the
repo-specific "Definition of done" in that repo's `CONTRIBUTING.md` / `AGENTS.md`.

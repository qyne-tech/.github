# AGENTS.md — QYNE (org-wide)

Instructions for **any** AI coding agent — Claude Code, OpenAI Codex, GitHub
Copilot, Cursor, Gemini CLI, and anything else — working in a QYNE repo. Humans:
these are the same rules you follow; the detail is in
[CODING_STANDARDS.md](CODING_STANDARDS.md) and [CONTRIBUTING.md](CONTRIBUTING.md).

## How this reaches every agent

The **same content** is surfaced to every tool, so there is one standard, not one
per agent:

- **`AGENTS.md`** (this file) — the shared entry point (Codex, Cursor, Gemini,
  Factory, and most agents read it natively).
- **`CLAUDE.md`** — a one-line `@AGENTS.md` import, so Claude Code loads this.
- **`.github/copilot-instructions.md`** — points here, so GitHub Copilot loads it.

Each repo carries these three thin files; they all resolve to this standard plus
that repo's own `AGENTS.md` rules. **Do not fork or restate the standard** in a
repo — link to it.

## Read before you code

1. This repo's `AGENTS.md` (stack-specific rules) and `README.md`.
2. The org [CODING_STANDARDS.md](CODING_STANDARDS.md) and
   [CONTRIBUTING.md](CONTRIBUTING.md).

## Non-negotiables (the rules an agent must never miss)

- **Follow [CODING_STANDARDS.md](CODING_STANDARDS.md) exactly** — naming,
  functions, modules, errors, tests. Match the surrounding code.
- **TypeScript strict; no `any`, no `!` to silence the compiler.**
- **Never `fetch`/query the backend directly from an app** — go through the shared
  `@qyne/core` client. Don't reimplement shared logic; put it in `packages/*`.
- **Never log or expose PII / payloads / secrets / tokens.** Authorize
  server-side from the JWT `sub`, never from a request body.
- **Commit with the operator's own org GitHub identity** (the `@qyne.one` account
  configured for the repo) — never invent a bot/AI identity or use an unrelated
  one. **No AI/tool attribution** in commits or PRs.
- **Conventional commits; branch off `develop`; never push to `main`/`develop`
  directly — always a PR.**
- **Do NOT merge PRs.** Open the PR, ensure CI is green, and stop — a human
  maintainer merges.
- **Keep docs in sync** in the same change — README, and backend OpenAPI/Swagger
  annotations.
- **Verify before "done":** run format, lint, typecheck, test, build; don't claim
  green without running it.

## Definition of done

[CODING_STANDARDS.md §11](CODING_STANDARDS.md#11-definition-of-done) plus this
repo's own definition of done.

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
3. If the work touches wearable data in any way, read
   [docs/WEARABLE-DATA-INVARIANTS.md](docs/WEARABLE-DATA-INVARIANTS.md)
   **before designing**, not while reviewing.
4. Designing a metric surface? Read
   [ADR-0006](https://github.com/qyne-tech/qyne-service/blob/develop/docs/decisions/0006-one-producer-per-figure.md)
   first: it decides what a box you cannot compute yet returns, and it is easier
   to design the empty state than to retrofit one.

## Non-negotiables (the rules an agent must never miss)

- **Follow [CODING_STANDARDS.md](CODING_STANDARDS.md) exactly** — naming,
  functions, modules, errors, tests. Match the surrounding code.
- **TypeScript strict; no `any`, no `!` to silence the compiler.**
- **Never `fetch`/query the backend directly from an app** — go through the shared
  `@qyne/core` client. Don't reimplement shared logic; put it in `packages/*`.
- **Never log or expose PII / payloads / secrets / tokens.** Authorize
  server-side from the JWT `sub`, never from a request body.
- **A figure you cannot compute yet says so; it never shows a number.** Return
  `null` with a declared reason and a ticket id, or mark it
  `// FORMULA NEEDED(TICKET-ID): why`. Never a sample value, never a zero
  standing in for an absent one, never a bare null. A number on a health screen
  is a claim and is indistinguishable from a measurement once rendered. See
  [ADR-0006](https://github.com/qyne-tech/qyne-service/blob/develop/docs/decisions/0006-one-producer-per-figure.md).
- **Commit with the operator's own org GitHub identity** (the `@qyne.one` account
  configured for the repo) — never invent a bot/AI identity or use an unrelated
  one. **No AI/tool attribution** in commits or PRs.
- **Conventional commits; branch off `develop`; never push to `main`/`develop`
  directly — always a PR.**
- **Do NOT merge PRs.** Open the PR, ensure CI is green, and stop — a human
  maintainer merges.
- **Keep docs in sync** in the same change — README, and backend OpenAPI/Swagger
  annotations.
- **Touch a metric, update its documentation in the same change.** A formula,
  weight, constant, threshold, band, unit, rollup or empty state that moves is
  only half a change until the figure's entry in
  `qyne-web/src/data/metricDocs.ts` moves with it. Both the public `/doc` page
  and the internal `/doc/internal` page render from that data. A stale band
  table does not fail a test; it gets quoted back at us.
- **Verify before "done":** run format, lint, typecheck, test, build; don't claim
  green without running it.
- **Wearable data: accuracy outranks completeness, and a feature must hold for
  every provider, not the one in front of you.** The invariants are in
  [docs/WEARABLE-DATA-INVARIANTS.md](docs/WEARABLE-DATA-INVARIANTS.md); a design
  that satisfies one source and breaks the rest is not done.

## Definition of done

[CODING_STANDARDS.md §12](CODING_STANDARDS.md#12-definition-of-done) plus this
repo's own definition of done.

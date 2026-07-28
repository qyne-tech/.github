# GitHub Copilot — QYNE

Follow the repository's [`AGENTS.md`](../AGENTS.md), which resolves to the QYNE
org [Coding Standards](https://github.com/qyne-tech/.github/blob/main/CODING_STANDARDS.md)
and [Contributing guide](https://github.com/qyne-tech/.github/blob/main/CONTRIBUTING.md),
plus this repo's own `AGENTS.md` rules. Use the same standard as every other agent.

Key rules: TypeScript strict (no `any`); no backend `fetch` from an app (use
`@qyne/core`); never log/expose PII or secrets; authorize server-side from the JWT;
conventional commits authored as `qyne-dev <support@qyne.one>` with no AI
attribution; branch off `develop`; never push to `main`/`develop` and never merge a
PR (a human maintainer merges).

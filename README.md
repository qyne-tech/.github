# .github — QYNE org defaults

Org-wide engineering standards, AI-agent instructions, and GitHub
community-health defaults for [**qyne-tech**](https://github.com/qyne-tech). Files
here are inherited by every repo that doesn't define its own.

## What's here

| File | Purpose |
|---|---|
| [CODING_STANDARDS.md](CODING_STANDARDS.md) | **The** coding standard — naming, functions, modules, errors, tests, security, git, PRs. Single source of truth for humans *and* AI agents. |
| [CONTRIBUTING.md](CONTRIBUTING.md) | Org-wide workflow: branches, commits, protected branches, PR process. The default `CONTRIBUTING` for all repos. |
| [AGENTS.md](AGENTS.md) | Cross-agent entry point (Claude, Codex, Copilot, Cursor, Gemini…). |
| [CLAUDE.md](CLAUDE.md) | `@AGENTS.md` import for Claude Code. |
| [.github/copilot-instructions.md](.github/copilot-instructions.md) | GitHub Copilot entry point. |
| [.github/PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md) | Org-wide default PR template. |
| [profile/README.md](profile/README.md) | The org's public profile page. |

## One standard, every agent

Each repo carries three thin pointer files — `AGENTS.md`, `CLAUDE.md`
(`@AGENTS.md`), and `.github/copilot-instructions.md` — that all resolve to the
standards here plus that repo's stack-specific rules. Change a rule **once**, here.

Repo files must **link** to these standards, never restate them.

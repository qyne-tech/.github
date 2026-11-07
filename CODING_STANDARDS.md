# QYNE Coding Standards

The single source of truth for how we write code across **every** QYNE repo
(`qyne-app`, `qyne-service`, `qyne-web`, `qyne-native`, `qyne-infra`). It applies
to **everyone** — humans and AI coding agents (Claude, Codex, Copilot, Cursor,
Gemini, …) alike. If you are an agent, see [AGENTS.md](AGENTS.md); it points back
here.

**Precedence.** This document is authoritative for universal rules. A repo's own
`AGENTS.md` / `CONTRIBUTING.md` adds **stack-specific** rules (e.g. React hooks,
NestJS modules, the native theme system) and, within its own domain, wins on
conflict. Nothing in a repo should *restate* the universal rules below — it links
here instead, so there is one place to change them.

---

## 1. Language & general principles

- **TypeScript, `strict` on.** No `any` — use `unknown` + narrowing, or a real
  type. No non-null `!` to silence the compiler; prove it or handle it.
- **Clarity over cleverness.** Optimise for the next reader. Prefer boring,
  obvious code to terse tricks.
- **Small, single-purpose units.** A function does one thing; a module owns one
  concern. If you need "and" to describe it, split it.
- **Immutability by default.** `const` over `let`; don't mutate inputs; return new
  values. Reach for `let`/mutation only with a reason.
- **No dead code.** No commented-out blocks, unused exports, or "just in case"
  abstractions. Delete it — git remembers.
- **Match the surrounding code.** Naming, structure, and comment density should be
  indistinguishable from the file you're editing.

## 2. Naming

- **Files:** match the framework's convention already in the repo. React
  components `PascalCase.tsx`; hooks `useThing.ts`; everything else
  `kebab-case.ts` (or the repo's established style — don't mix).
- **Variables & functions:** `camelCase`. Names describe intent, not type
  (`athleteProfile`, not `data` / `obj` / `tmp`).
- **Booleans:** read as a predicate — `isLoading`, `hasWearableData`,
  `canRetry`, `shouldCreateUser`.
- **Functions:** verb-first — `getReadiness`, `computeAge`, `verifyToken`. A
  function that returns a boolean reads as a question — `isValid`, `hasAccess`.
- **Types / interfaces / classes / enums:** `PascalCase`. Don't prefix interfaces
  with `I`.
- **Constants:** `UPPER_SNAKE_CASE` only for true module-level compile-time
  constants (`OTP_LENGTH`, `RESEND_SECONDS`); ordinary `const` bindings stay
  `camelCase`.
- **No abbreviations** unless they're domain-standard (`hr`, `hrv`, `rhr`, `id`,
  `url`, `dto`). No `usr`, `btn`, `cfg`, `e2`.
- **Units in the name** when not obvious — `weightKg`, `timeoutMs`,
  `sleepGoalHours`, `distanceKm`.

## 3. Functions

- **Keep them short** — aim for a screen. A long function is usually several
  functions.
- **Early returns** over nested `if`. Handle the error/edge case first and bail.
- **Few parameters.** More than ~3 → pass an options object with a named type.
- **Pure where possible.** Isolate side effects (I/O, DB, network) from logic so
  the logic is unit-testable without mocks — e.g. a pure `computeReadiness()` /
  `ageFromDateOfBirth()` that a thin service calls.
- **Explicit return types on exported functions.** Inference is fine for locals.
- **No boolean/flag parameters** that change behaviour — split into two functions.

## 4. Modules & imports

- **Feature-oriented folders**, not type-oriented — group by domain
  (`athletes/`, `wearables/`, `readiness/`), not `controllers/`, `services/`
  scattered globally.
- **Shared logic lives in a shared package**, never copy-pasted. In the monorepo,
  cross-app code goes in `packages/*` (`@qyne/core`, `@qyne/tokens`); apps never
  reimplement it. **Never `fetch`/query the backend directly from an app** — go
  through the injected `@qyne/core` client.
- **Dependency inversion:** shared/core code never reads env or platform globals —
  config is injected (see `@qyne/core`'s `AppConfig`). Keep platform-specific code
  (BLE, HealthKit, `window`, `import.meta.env`, `process.env`) out of shared
  packages.
- **Import order:** external deps → internal packages (`@qyne/*`) → relative. No
  deep imports into another module's internals; use its public barrel.

## 5. Comments & documentation

- **Explain *why*, not *what*.** The code says what; comments capture intent,
  trade-offs, and non-obvious constraints (idempotency, ordering, units,
  empty-state shapes).
- **JSDoc every exported/public symbol** — a one-line purpose, plus anything a
  caller can't infer (nullability, side effects, units).
- **No noise** — no `// increment i`, no commented-out code, no changelog
  comments (git is the changelog).
- **A metric change includes its documentation.** Move a formula, weight,
  constant, threshold, band, unit, rollup or empty state, and update that
  figure's entry in `qyne-web/src/data/metricDocs.ts` (or
  `metricPipeline.ts` for provenance) in the **same** PR. Both doc pages render
  from that data, and every entry cites the file it was transcribed from.

## 6. Error handling

- **Never swallow errors.** No empty `catch {}`. Handle it, or let it propagate to
  the framework's global handler (don't hand-roll error logging).
- **Fail with context** — throw typed/framework errors with a clear message; on
  the backend return the correct status (`400` validation, `401`/`403` auth,
  `404` missing, `409` conflict) — never a bare `500` for a foreseeable case.
- **Validate at the boundary.** Untrusted input (request bodies, params) is
  validated before use; align validation with what storage actually accepts (a
  DTO bound must match the DB column, e.g. `numeric(5,2)` → `@Max(999.99)`).

## 7. Testing

- **Unit-test the logic**, especially pure functions and anything with branches or
  math (readiness, age, validation). Colocate as `*.spec.ts` next to the source.
- **Test behaviour and edge cases**, not implementation details — empty/no-data,
  boundaries, error paths.
- **A bug fix ships with a test** that fails before the fix.
- CI must be green: format, lint, typecheck, test, build — all pass.

## 8. Security & data (health + identity — treat as sensitive)

- **No secrets in code, logs, or fixtures.** Config via env / Secrets Manager.
- **Never log PII or payloads** — no phone/email/name/DOB, no measurements, no
  bodies, no tokens. Log ids, counts, durations, outcomes.
- **Authorize server-side.** Derive the user from the verified JWT (`sub`), never
  from a request body. Every non-public route is guarded and scoped to the caller.
- **Don't weaken a security gate** (e.g. prod Swagger off) without an explicit
  reason in the PR.

## 9. Observability (logs and audit)

Two systems with different jobs. **Logs** answer "what is the system doing and
why did it break"; they are disposable. The **audit trail** answers "who did
what to whose data, and did it succeed"; it is evidence. Most changes touch
both, and losing the second one is a compliance hole rather than a debugging
inconvenience.

### Structured, always

- **Log an object, never an interpolated string.** `logger.warn({ msg: 'auth
  rejected', reason: 'expired_token' })`, not `` logger.warn(`auth rejected:
  ${err.message}`) ``. A string is unqueryable, and it bypasses the redactor,
  which only reaches structured fields.
- **Every line carries `msg`** as a short, stable, lower-case event name. Treat
  it as an identifier: alerts and dashboards key off it, so renaming one is a
  breaking change.
- **Reason codes, not prose.** A rejection logs `reason: 'no_bearer_header'`,
  a stable snake-case value from a known set. Prose cannot be counted.
- **Correlation is free.** The request id and caller are stamped automatically.
  Never thread them through a signature, and never interpolate an id that is
  already a field.

### What must be logged

Prohibitions alone do not make a system observable. Each of these needs a line:

- **Every outbound dependency call**: the provider, the operation, `durationMs`,
  and the outcome. When the product is slow, this is what separates our latency
  from a provider's.
- **Every rejection or degradation the caller cannot see**: a skipped
  reconciliation, a dropped notification, a fallback taken, a limit hit.
- **Every batch outcome**: accepted, rejected and retracted counts. Counts, not
  contents.
- **Start-up and shutdown**: the effective configuration at boot (environment,
  log levels, which integrations are enabled) and the signal that ended the
  process. An instance that recycled must be distinguishable from one that
  crashed.

A plain handler needs no logging of its own. The access line and the global
error filter already cover it. Add logs for the interesting middle: decisions,
degradations and batch outcomes.

### Never logged

No bodies, no measurements, no identity (phone, email, name, date of birth,
address), no credentials. Ids, counts, durations, status codes, outcomes and
reason codes are safe. See [§8](#8-security--data-health--identity--treat-as-sensitive).

Health data in a URL path is included: a route whose parameter is itself a
measurement or a date of one is logged as its template, never its value.

### Audit the privileged surface

- **Audit anything that creates, changes, exports or deletes user data, and
  anything that grants or denies access.** Retracting a measurement counts.
  Minting a credential counts. A read counts once anyone can read data that is
  not their own.
- **Record failures and denials, not just successes.** In an incident the
  rejected attempts are usually the interesting ones.
- **Use the typed action vocabulary**, never a free string, so the privileged
  surface stays reviewable in one place. Adding an action is a deliberate act.
- **Commit the audit row with the write.** Inside a transaction, pass the
  transaction handle: an audit entry for a write that never landed is worse than
  no entry.
- **The payload is metadata.** Counts, ids and reasons. Never measurements, and
  never the content of what changed.
- **The trail is append-only.** Nothing in the application may update or delete
  an audit row, and the database role it connects with should not be able to.

### Ready for the log pipeline

The service may be shipping to nothing but stdout today. It must still be
queryable the day a pipeline is attached, which means the shape cannot be an
afterthought:

- **Every line identifies its origin**: service name, environment, and the build
  it came from. Without the build, no regression can be tied to a deploy.
- **Route labels are templates, not values.** `/articles/:id`, never
  `/articles/0fc45698-...`. An unbounded label is useless as a dimension and
  expensive as a metric.
- **The dimension that predicts cost is a field.** A request whose work scales
  with a parameter records that parameter, or its latency cannot be explained.
- **One event per line, JSON in every deployed environment.** Human-readable
  output is for a terminal, not for a log group.

## 10. Git & commits

- **Branch flow** (see [CONTRIBUTING.md](CONTRIBUTING.md) for the full table):
  work off `develop`; `feat/*`, `fix/*`, `chore/*`, `hotfix/*`. Never push to
  `main` or `develop` directly — always via PR.
- **Conventional commits:** `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`,
  `test:`, `ci:`. Subject in the imperative, ≤ 72 chars; body explains *why*.
- **Commit with your own org identity** — the GitHub account you were granted
  access with (your `@qyne.one` account). Set `user.name` / `user.email` to it;
  don't commit under an unrelated personal or other-employer identity.
- **No AI/tool attribution** in commit messages or PR descriptions.
- Small, focused commits; each one builds.

## 11. Pull requests

- **Small and focused** — one concern per PR. Split unrelated changes.
- **Describe the change**: what, why, how verified. Screenshots for UI. Link the
  issue.
- **Green CI is required.** Format, lint, typecheck, test, build.
- **Keep the README current** — if the change makes it wrong, fix it in the same
  PR; if not, say "README: no change needed" in the description.
- **The maintainer merges — never self-merge.** Open the PR and leave it; merging
  is the maintainer's call. Prefer **squash-merge**.
- A release is a PR from `develop` → `main`.

## 12. Definition of done

- CI green (format, lint, typecheck, test, build).
- Input validated; auth server-side; no secrets/PII in code or logs.
- Tests for new logic; a failing-then-passing test for bug fixes.
- Observable in production: logs are structured objects with a stable `msg`,
  every outbound call records its duration and outcome, and every rejection
  carries a reason code ([§9](#9-observability-logs-and-audit)).
- Privileged action audited with a typed action, inside the transaction that
  performs it, with failures and denials recorded too ([§9](#9-observability-logs-and-audit)).
- Docs current — README, and any repo-specific contract docs (OpenAPI/Swagger on
  the backend), updated or explicitly confirmed unaffected.
- Metric documentation current — if the change touches a figure an athlete sees,
  its entry in `qyne-web/src/data/metricDocs.ts` is updated in the same PR, or
  explicitly confirmed unaffected.
- Repo-specific "definition of done" (in that repo's `AGENTS.md`/`CONTRIBUTING.md`)
  also satisfied.

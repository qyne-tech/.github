# Code review and coding standards

The gate you pass through **before writing code, and before reviewing a PR**, in
any QYNE repo. Humans and AI agents alike.

## How to use this, and what it is not

This file does **not** restate the rules. [CODING_STANDARDS.md](CODING_STANDARDS.md)
is authoritative for how code is written, and a repo's own `AGENTS.md` wins
inside its own stack. Forking a rule into two documents is how the two copies
drift, which is the same failure this document spends most of its length warning
about.

So this file does three things, and only three:

1. **Routes** you to the authoritative rule for each check, in the order a review
   should run.
2. **Specifies the review protocol**, which exists nowhere else.
3. **Carries the handful of rules with no other home**, marked :round_pushpin:.

Every rule below names a real incident. None of them is here because it is good
practice in the abstract.

---

## 1. The review protocol :round_pushpin:

### Show findings before posting them

**Draft the review, show it to the maintainer, post it when they say so.** Not
after. A review is a public statement about someone's work, and a wrong one costs
them a day of chasing a defect that was never there.

This rule exists because it has been broken: a second-pass review went out on a
PR while the maintainer was mid-sentence asking to see it first, and a review of
two more PRs was posted on a loose reading of "review these". Neither was
recoverable. GitHub does not allow a submitted review to be deleted, only edited.

Posting without being asked is the exception, not the default, and the exception
needs the maintainer to have said so in that message.

### One finding, one claim

A finding states **what breaks, for whom, under what input**. "This looks fragile"
is not a finding. If you cannot name the input that produces the wrong output,
you have a question, not a defect. Ask it as a question.

### Withdraw loudly

A finding you got wrong is corrected in the same thread, plainly, without
ceremony. Do not quietly drop it. The author has already started investigating.

### Severity, and what each one means

| | meaning | blocks merge |
|---|---|---|
| **Defect** | a named input produces a wrong output, a crash, or a false claim to an athlete | yes |
| **Risk** | correct today, but the next change breaks it silently | usually not |
| **Note** | a reader will be misled, or a convention is broken | no |
| **Question** | you do not understand something and said so | no |

A PR that cannot merge (conflicts, failing CI) gets a **comment**, not a
changes-requested, when the blocker is not the author's fault.

---

## 2. The evidence standard :round_pushpin:

**Verify the claim against the running code before you write it down.**

Read the surrounding context, not the diff hunk. A finding drafted on this
codebase claimed a Recovery ring showed the anchor day's score on every tab,
painted by a tone describing the period average. The code was three lines inside
an `isToday` branch, and the period tabs drew a chart instead. The finding was
wrong, and only reading twenty lines up caught it.

Specific traps here, each one real:

- **Green tests are not a typecheck.** `ts-jest` does not type-check. A merge can
  pass 1,910 tests and fail `tsc`. CI runs them separately; so should you.
- **A green check can be stale.** If base branches have merged since, the check
  describes a tree that no longer exists.
- **A clean `git merge` is not a working merge.** Two branches can each add code
  at the same insertion point, merge without conflict markers, and not compile.
  Build the merged tree before calling a conflict mechanical.
- **Verify in an isolated worktree, not your working copy.** More than one agent
  may be working in the same checkout. An isolated run is what caught a commit
  that had swept up another session's unrelated refactor.
- **Fixture order is load-bearing.** `createFakeDb` serves canned results in call
  order. Adding one query shifts every later one, and three tests failed in ways
  that looked like logic errors and were not.

---

## 3. Before you write code

Run these in order. Each links the rule that governs it.

1. **Read the repo's `AGENTS.md`, then [CODING_STANDARDS.md](CODING_STANDARDS.md).**
2. **Is there already a producer for this figure?** See *One producer* below.
3. **Does a shared package already do this?** Cross-app code lives in
   `packages/*`; apps never reimplement it
   ([CODING_STANDARDS §4](CODING_STANDARDS.md#4-modules--imports)).
4. **Naming, casing, units in the name**
   ([§2](CODING_STANDARDS.md#2-naming)). `weightKg`, not `weight`.
5. **What does this return when it cannot compute?** Design the empty state
   first. Retrofitting one is harder than designing it.
6. **Is this a figure an athlete sees?** Then its documentation entry is part of
   the change. See *Touch a metric, update the documentation* below.
7. **Will it be audited?** Privileged actions go through `AuditService.record()`;
   see the service's `docs/LOGGING.md` §4.

---

## 4. Before you approve

In this order. Stop at the first that fails.

- [ ] **CI green**: format, lint, typecheck, test, build. Not assumed, read.
- [ ] **Does the merged tree build?** Not just the branch.
- [ ] **No PII in logs.** Not phone, email, name, date of birth, measurements,
      bodies or tokens. Ids, counts, durations, outcomes only
      ([§8](CODING_STANDARDS.md#8-security--data-health--identity--treat-as-sensitive)).
- [ ] **Logs are structured.** An object with a stable `msg`, never an
      interpolated string. A string cannot be queried and is not redacted
      ([§9](CODING_STANDARDS.md#9-observability-logs-and-audit)).
- [ ] **Could you debug this in production from the logs alone?** Outbound calls
      record duration and outcome; rejections and fallbacks carry a reason code;
      batch work reports its counts. See below.
- [ ] **Audit rows for privileged actions**, per `docs/LOGGING.md` §4: a typed
      action, inside the transaction, with the denial and failure paths recorded
      and not only the happy one.
- [ ] **Auth derived server-side** from the verified JWT `sub`, never from a body.
- [ ] **No value without a name.** See below.
- [ ] **One producer per figure.** See below.
- [ ] **Empty states are honest**: null with a reason, never a zero or a sample
      that reads as a measurement.
- [ ] **Tests cover the branches**, including the empty and boundary cases, and a
      bug fix has a test that failed before it.
- [ ] **Docs match the code**: Swagger on the backend, and no comment that now
      lies.
- [ ] **Metric documentation updated in this PR** if a figure, weight,
      constant, threshold, band, unit, rollup or empty state moved. See below.
- [ ] **No dead code, no commented-out blocks, no noise comments**
      ([§5](CODING_STANDARDS.md#5-comments--documentation)).

---

## 5. Rules with no other home :round_pushpin:

### No value without a name

A literal that encodes a decision gets a named constant, in the module that owns
the decision, with a comment saying why it has that value. This covers magic
numbers, thresholds, colours, copy, timeouts, and window lengths.

Three ways this has gone wrong here:

- **A colour decided on the client.** A Recovery ring was `color={colors.success}`
  whatever the score, so a recovery of 28 was green on one screen and red on the
  next. The band was already in the payload.
- **A window written out twice.** A baseline was specified as a four-week median,
  but the fetch window was still a hard-coded `8`, so the median was computed
  from eight days and the rule was silently not implemented. The fix was to
  derive the fetch window from the baseline constant so the two cannot disagree.
- **Eleven constants named the same thing.** Eleven separate `BASELINE_DAYS`
  exist across one service, with genuinely different values for different
  purposes. Same name, different meanings, no way to tell them apart at a call
  site. Name a constant for what it configures.

Corollary: **a threshold that decides what an athlete is told belongs on the
server**, not in a screen file. A client that bands a figure itself will
eventually disagree with the server about the same reading. It has: two rules for
the same HRV reading put an athlete exactly on their own baseline in the green
band on one screen and the amber band on another.

### A log line is an interface

Treat `msg` and the field names beside it the way you treat a response field.
A dashboard, an alarm or an incident query keys off them, so renaming one is a
breaking change to something you cannot see from the call site. Pick the name
once, make it specific, and leave it alone.

Two failures to watch for in review:

- **Prose where a value belongs.** `` `WHOOP ${res.status} for ${path}` ``
  cannot be counted, grouped or alarmed on, and it sails past the redactor
  because redaction only reaches structured fields. The same information as
  `{ msg: 'provider call failed', provider: 'whoop', status, operation }` is
  one Logs Insights query away from a rate per provider.
- **An unbounded value used as a label.** A route logged with its uuid in it,
  or an error message interpolated from a dependency, produces a label with no
  fixed set of values. As a log field that is merely noisy; as a metric
  dimension it is expensive and useless.

The test to apply: name the question this line would answer during an incident
at 2am, and check that answering it needs no regular expression.

### Audit the denial, not just the deed

The happy path is the easy half and the one everybody writes. An audit trail
that records only successes cannot answer the question it exists for, because
the interesting evidence in almost every incident is the attempt that was
refused: the password change that was denied, the sign-up that was rejected, the
token that failed to verify, the data that was retracted.

So in review, for every audited action, ask what the failure and denial paths
write. If the answer is nothing, the action is half audited.

The same applies to a privileged action with no audit action at all. Retracting
a measurement, minting a session token and merging two device identities are all
changes to an athlete's record, and each one is the sort of thing a security
review asks to see. A read becomes auditable the moment anyone can read data
that is not their own.

### One producer per figure

A number shown to an athlete is computed in exactly one place. If two paths can
compute it, they will eventually disagree and nobody will know which is right.

This is [ADR-0006](https://github.com/qyne-tech/qyne-service/blob/develop/docs/decisions/0006-one-producer-per-figure.md).
Its second rule is the one most often missed: **a different derivation is a
different figure, and needs a different name.** A percentile against published
norms and a percentile against other members are not the same number and must not
share a field.

In review, ask: can this value be derived anywhere else? If yes, which is
authoritative, and what stops the other one existing?

### Touch a metric, update the documentation

**A change to a figure includes the change to its documentation, in the same
pull request.** Not a follow-up ticket, and not "I'll do the docs after the
formula settles": the formula shipping is what makes the doc wrong.

What counts as touching a metric:

- a formula, a weight, a coefficient, a threshold or any named constant
- a band table: a boundary, a label, a tone, or the basis (personal / generic)
- a unit, a rollup rule, a precedence rule, or an accepted range
- an empty state or the reason string behind it
- which sources supply the figure, or a new figure on a screen

**Where it goes.** Both documentation pages are rendered from data in
`qyne-web`: `src/data/metricDocs.ts` holds one entry per figure (formula,
term-by-term equations, every constant, bands, what it depends on, limits) and
`src/data/metricPipeline.ts` holds the received-vs-computed tables, the ingest
path and the code map. `/doc` is the public page; `/doc/internal` renders the
same data with the file citations and constant names. One edit, no second copy.

Every entry cites the file that owns the formula, so the mover's checklist is a
`grep` for the citation.

**Why this is a merge gate and not a nicety.** The page publishes the bands an
athlete reads their own health against, and a stale band table does not fail a
test. It gets quoted back at us in a support ticket.

Two cases, both real:

- The first draft of that page was written from a recent reading of the code and
  was wrong in **five** places: heart-rate zones documented as cut from reserve
  when they are percent of maximum, four VO2 max bands where the table has five,
  three wrong energy tones, a dead strain implementation cited as the live one,
  and respiratory rate documented as supplied when nothing sends it. Every one
  was caught by opening the file, and none would have been caught by review.
- This repository's own `AGENTS.md` linked ADR-0006 as
  `0006-unimplemented-figures.md`. The file is `0006-one-producer-per-figure.md`
  and has been for months. A doc pointing at nothing is the quiet half of the
  same failure.

**And the honesty clause.** The public page is built from the same data as the
internal one, so a limitation you would not want published is a limitation to
**fix**, not to omit. Softening a figure for the public page is the one edit
that is never allowed.

### SOLID, as it applies here

The acronym is not the point; these four consequences are.

- **Single responsibility.** If describing a function needs the word "and", split
  it. A service that reads a database and also decides a band is two things, and
  the band cannot be unit-tested without a database.
- **Dependency inversion.** Pure logic takes its inputs; it does not reach for a
  clock, an environment variable, or a connection. `computeReadiness` is pure and
  a thin service calls it. Shared packages never read `process.env` or platform
  globals ([§4](CODING_STANDARDS.md#4-modules--imports)).
- **Open to extension.** Adding the eighth provider, or the sixth metric, should
  mean adding data to a table, not an eighth branch in a `switch`. Where you find
  the `switch`, that is the finding.
- **Interface segregation.** A caller that needs a day's figure should not have to
  accept an object carrying the whole screen.

### Common utils, constants and config

Before adding a helper, search for it. This codebase has carried two `median`
implementations, two `niceAxis` implementations with byte-identical bodies, and
three `clamp` helpers.

- Time arithmetic belongs in `common/time`. Hand-rolling `AT TIME ZONE` anywhere
  else is a finding, because the day boundary is one rule.
- Cross-app code belongs in `packages/*`, never copy-pasted into an app.
- Config is injected, never read from the environment inside shared code.
- A constant that two modules need belongs in the one that owns the decision, and
  is imported, not retyped.

### Comments earn their place

Explain **why**, never what. A comment that restates the code is noise; a comment
that is wrong is worse than none.

Both failure modes are live here. `ageYearsAt` was documented "Whole years" and
returned `30.00000000060209`, because it divides by the mean year length. The
value was right for the model that consumes it and wrong to show anybody, and the
comment sent readers the wrong way for a long time.

When you change behaviour, **re-read the comments and the API descriptions around
it.** A route that started answering `404 account_not_found` shipped with a DTO
description still promising "the response is the same whether or not an account
exists", and both are published in the same Swagger document.

---

## 6. Where everything lives

| | |
|---|---|
| How code is written | [CODING_STANDARDS.md](CODING_STANDARDS.md) |
| Branches, commits, PRs, releases | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Agent entry point | [AGENTS.md](AGENTS.md) |
| Provider and metric invariants | [docs/WEARABLE-DATA-INVARIANTS.md](docs/WEARABLE-DATA-INVARIANTS.md) |
| Logging and audit | `qyne-service/docs/LOGGING.md` |
| Decisions already taken | `qyne-service/docs/decisions/` |
| Review cases, app-specific | `qyne-app/docs/CODE_REVIEW.md` |
| What every metric means, publicly | `qyne-web` `/doc` |
| The same, to the file | `qyne-web` `/doc/internal` |
| The content behind both | `qyne-web/src/data/metricDocs.ts`, `metricPipeline.ts` |

---

## 7. Why this is stricter than it looks

Athletes act on these numbers. A crash is obvious and gets fixed in a day. A
recovery score that is quietly fifteen percent wrong changes how somebody trains
for a month, and nobody files a bug.

That asymmetry is why most of this document is about **silent wrongness** rather
than correctness in the usual sense, and why a plausible-looking default is
treated as worse than a visible failure.

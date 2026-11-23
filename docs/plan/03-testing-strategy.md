# Darsi (دَرْسي) — Testing Strategy

> Status: **✅ Approved** · 2026-11-22
> Architecture: [02-architecture.md](02-architecture.md) · Scope: [01-mvp-scope.md](01-mvp-scope.md) · Decisions: [../PLAN.md](../PLAN.md)

## 1. Goals

1. **Every business rule has a test that names it.** Access windows, quotas, lockout, selection and mastery are pure functions (architecture section 3), so they're tested exhaustively and fast.
2. **Tenant isolation is proven, not assumed.** A teacher can't read or change another workspace's data, and a student can't reach another class. A guard test fails if a new use case ships without an isolation test.
3. **The core journeys work in a real browser** on a small Android-sized screen, in Arabic, right to left.
4. **CI is fast and trustworthy:** under about 10 minutes, no real AI calls, no money spent, and no flaky tests left in place.
5. **Tests ship with the feature.** Each feature PR includes its tests; a bug fix starts with a failing test.

## 2. Test layers

| Layer | Tool | What it covers | Runs against | When |
|---|---|---|---|---|
| Static checks | ESLint, TypeScript (strict), Prisma checks | Types, the `domain/` import rule (architecture section 3), schema and migrations in sync | Source code | Every PR |
| **Unit** | **Vitest** (+ **fast-check** for property tests) | `domain/` modules, AI output parsing, prompt building, digit and money helpers | Plain functions; no database, network or clock | Every PR; locally on save |
| **Integration** | **Vitest** | Use cases in `server/<feature>`: authorization, transactions, SQL constraints, races, tenant isolation, AI flow with the fake provider | Real **PostgreSQL** in Docker | Every PR |
| **End to end** | **Playwright** | Core journeys for teacher, student and admin; accessibility; the student page weight budget | Production build, seeded database, fake AI, Clerk test instance | Every PR |
| AI contract (live) | Vitest, run by hand | The OpenRouter provider passes the same contract tests as the fake one | OpenRouter, costs cents | By hand, never in CI |
| Model evaluation | A script | Quality and cost of each candidate model (architecture section 7) | OpenRouter, costs dollars | Once, after the owner agrees |

**No separate component-test layer.** Phase 1 has few client components (the practice screen, forms with live validation, the copy-code button). Their logic lives in pure functions with unit tests, and their behavior in the browser is covered by Playwright. This avoids testing the same thing twice.

## 3. Unit tests: what each domain module must prove

Time (`now`) and randomness (`rng`) are passed in, so every case below is a plain function call with fixed inputs.

| Module | Cases that must have a test |
|---|---|
| `access` | Trial active and expired. A paid period gives access from approval or the window start, whichever is later, until the end of the period's last day in Palestine time (D-19). Expired is read-only. Expired with a pending notice is still read-only (D-16). The reminder appears 14 days before access ends. Changing a semester's dates moves access. Day boundaries are checked around daylight-saving changes, without hard-coding UTC offsets |
| `calendar` | Semesters in a year are in order and don't overlap. Each period's window starts where the previous one ends, across years; the first period starts on its own start date (D-19). The periods a teacher can buy now, hiding periods that overlap one already paid for or pending, each with the date access would start |
| `payment-notice` | Only Pending → Approved or Rejected; a rejection needs a reason; nothing leaves Approved or Rejected |
| `ai-quota` | Trial 10, semester 300, year 600 (from configuration). A paid allowance counts from approval and replaces the trial's (D-19). Failed generations don't count. Unused generations don't carry over. The daily cap counts per day in Palestine time (D-21) |
| `ai-output` | A fully valid response. One bad item out of five: four kept, one counted. Zero valid items: failed. Options 2–4 and distinct, `correctIndex` in range, length limits. Arabic text with diacritics is measured consistently |
| `practice` | Up to 5 questions; only published ones; order is never answered → most recently wrong → oldest attempt. Fewer than 5 available. Shuffling with a seeded `rng` is repeatable, and each option keeps its original index |
| `mastery` | Share of the topic's published questions whose latest attempt is correct. Star thresholds at their exact boundaries. "Needs review" threshold. A topic with no questions. Unpublished questions don't count |
| `pin` | Locked after the 5th failure for 15 minutes; unlocked at exactly 15 minutes; a success resets the counter; tries during a lockout are refused without being counted |
| `join-code` | The alphabet has no look-alike characters. Typed codes match regardless of case, spaces and digit style |
| `digits` | Arabic-Indic (٠–٩) and Persian (۰–۹) digits become 0–9; other characters are untouched; converting twice changes nothing; display formatting |
| `money` | Whole agorot only; ILS formatting |

**Property tests (fast-check)** cover the modules where hand-picked examples miss edge cases. They generate many random inputs and check a rule that must always hold:

- `calendar`: for any valid calendar, period windows never overlap and leave no gaps.
- `practice`: never more than 5 questions, never duplicates, never an unpublished question.
- `ai-output`: any input, even malformed, splits into valid and invalid items without throwing.
- `digits`: conversion is idempotent and only ever changes digits.

**Prompt tests.** The prompt builder only accepts material text, the grade and the subject name, so its type makes sending student data impossible (D-08). A snapshot test catches accidental prompt changes; a deliberate change updates the snapshot and the prompt version together.

## 4. Integration tests (real PostgreSQL)

### Setup

- **Same database as production:** PostgreSQL in Docker, on the same major version as the Neon project (pinned in both places). Migrations run once into a template database.
- **One database per Vitest worker,** copied from the template, so test files run in parallel without sharing data. Tables are emptied between tests.
- **Factories, not shared fixtures:** helpers such as `makeWorkspace()`, `makeClass()` and `makeSubject()` create exactly the data each test needs. The seed script is separate and used only for E2E and the demo.
- **Clerk is replaced by an injected actor:** use cases receive the actor from `server/auth`, so tests pass a teacher, student or admin directly. The real Clerk connection is covered by E2E.
- **The AI is the `FakeProvider`,** with scenarios for valid output, partly invalid output, refusals, errors and timeouts.

### What they cover

| Area | Tests |
|---|---|
| **Tenant isolation** | For every use case that takes an ID from the browser: an actor from workspace B using workspace A's ID gets "not found" (not "forbidden", which would reveal that the ID exists), and nothing changes. Students: a student can't read or answer another class's topics and questions |
| **Isolation guard** | A test lists every exported use case and fails if one has no isolation case. This is what makes goal 2 hold as features are added |
| SQL constraints | One test per constraint written in SQL: one pending notice per workspace; exactly one period on notices and subscriptions; unique `(roundId, questionId)` on attempts; `RESTRICT` on calendar periods in use; `SET NULL` when a material is deleted (its questions and AI usage rows stay); unique `tokenHash`, subject name per class, student name per class, membership per workspace and user |
| Races | Two approvals of the same notice at once create one subscription. Two first requests create one workspace. A double-tapped answer saves one attempt. The allowance and daily cap go at most one over (the documented trade-off) |
| Sessions | Only the token's hash is stored; a PIN reset revokes sessions; expired sessions are refused |
| AI flow | Success saves drafts and uses one generation. Partly invalid: valid items saved, count shown. Zero valid, error or timeout: failed, nothing used. Allowance or daily cap reached: blocked before calling the provider. Global switch off: blocked. The provider receives only material text, grade and subject (D-08) |
| Billing flow | Submitting, approving and rejecting notices; access after approval follows D-19 |
| Demo reset | Restores the demo workspaces and calendar, and leaves other workspaces untouched (D-20) |
| `User` copy | Updated when the name or email from Clerk differs |

## 5. End-to-end tests (Playwright)

### Environment

- **A production build** (`next build` then `next start`), not the dev server, so tests see what users get.
- **A fresh database per CI run,** migrated and seeded. Tests create their own classes inside the test teachers' workspaces and never change the seeded demo data, so tests don't depend on each other.
- **The fake AI provider.**
- **A Clerk development instance** with fixed test users (two teachers and an admin) and Clerk's Playwright helpers (`@clerk/testing`). The sign-up journey creates a throwaway test user and deletes it afterwards. To be confirmed in the auth PR.
- **A small Android phone profile** (about 360×640), Arabic locale, right to left. Chromium only: the users are on Android phones.

### Journeys

| # | Journey |
|---|---|
| 1 | A teacher signs up: workspace created with a 14-day trial; creates a class, a subject, a topic and a material; writes and publishes a question |
| 2 | AI drafts: generate questions, edit one, publish one, discard one; generate and publish a summary; the remaining allowance goes down |
| 3 | Students: add students; the roster shows PINs; a student logs in with the class code typed in Arabic-Indic digits, skips the subject screen (one subject), practises a round with feedback, and sees stars |
| 4 | PIN lockout: 5 wrong PINs lock the student; the teacher resets the PIN and the old session stops working |
| 5 | Billing: an expired workspace is read-only; the teacher submits a payment notice and sees "payment received"; the admin approves and the teacher can edit again; a rejection shows its reason |
| 6 | Admin calendar: create a school year with semesters; overlapping semesters are refused |
| 7 | Class results: the grid for one subject, with the "needs review" flag |
| 8 | Demo buttons: "Try as teacher" signs in while the demo setting is on (D-20) |
| 9 | Isolation smoke test: teacher B opening teacher A's class URL gets a not-found page |

### Extra checks

- **Accessibility:** axe (`@axe-core/playwright`) on the login, practice, teacher dashboard and admin pages; serious or critical problems fail the test.
- **Student page weight:** on student pages, the test adds up the JavaScript downloaded and fails above the ~150 KB budget (architecture section 9). It also fails if any request goes to Clerk, since student pages must never load it.

### Flaky tests

No fixed waits; only Playwright's auto-waiting assertions. CI retries a failed E2E test once and keeps a trace. A test that passes only on retry is fixed before more feature PRs are merged.

## 6. Testing the AI

- **`FakeProvider`** returns fixed responses chosen by a scenario: valid, partly invalid, all invalid, refusal, error, timeout. It powers all automated tests.
- **Contract tests:** one shared suite checks what any `AiProvider` must do (shape of results, how failures are reported). It runs against the fake in CI. The same suite runs against `OpenRouterProvider` by hand (`npm run test:ai-live`, needs a key, costs cents), so the fake stays honest about the real thing.
- **The model evaluation** stays separate from tests (architecture section 7): it measures quality and cost, it doesn't pass or fail.

## 7. CI pipeline (GitHub Actions)

Four jobs run in parallel on every PR, and all must pass before merging:

| Job | Steps |
|---|---|
| Checks | Lint, type check, Prisma schema and migration consistency |
| Unit | Vitest unit tests with coverage |
| Integration | Vitest integration tests with a Postgres service container |
| E2E | Production build, Postgres service container, migrate and seed, Playwright |

- **Target:** under about 10 minutes in total.
- **Coverage:** `domain/` must stay at **90% or more** of lines and branches, enforced. Other folders are reported but not enforced, because a percentage there rewards shallow tests.
- **Secrets:** CI only needs the Clerk development instance keys. Pull requests from forks of the public repo don't get secrets, so their E2E job is skipped; the owner's PRs run everything.
- **Alongside CI:** Vercel builds a preview deployment, and CodeRabbit reviews the PR (repo setup step).

## 8. Conventions

- **Where tests live:** unit tests next to their code (`*.test.ts`), integration tests as `*.int.test.ts`, E2E tests in `e2e/`.
- **Names describe the rule** in plain English, for example "locks the student for 15 minutes after the 5th wrong PIN".
- **Arabic test data** where text matters (names, materials, questions), to catch right-to-left and Unicode problems early.
- **No real student data** and no ministry materials in test data (D-08, D-09).
- **No `Date.now()` or `Math.random()` in `domain/`;** lint enforces it, and tests pass fixed values.

## 9. Not tested in Phase 1, and why

| Not tested | Why | When to revisit |
|---|---|---|
| Visual screenshots | Arabic text rendering differs between machines, so screenshot comparisons flake | If UI regressions appear |
| Load and performance under traffic | Seed data and a demo only | Before the pilot |
| Browsers other than Chromium | Users are on Android phones | If iOS users appear in the pilot |
| Real AI in CI | Costs money and gives different answers each time | Never; covered by the contract tests and the evaluation |
| Clerk's own screens | A vendor's component; only our use of it is tested | — |
| Mutation testing | Useful for `domain/`, but adds CI time | After Phase 1 |

## 10. Effort and cut line

Testing is already inside the ~8-day estimate (scope, "Realism notes"): tests are written in each feature PR. The shared test infrastructure (database per worker, factories, Clerk test setup, CI jobs) takes about half a day in the setup PRs.

If time runs short, cut in this order, and list each cut under "Known limitations" in the README:

1. Property tests (keep the example-based unit tests).
2. Accessibility checks.
3. The student page weight check (do the Lighthouse check by hand instead).
4. E2E journeys 6, 7 and 8 (keep their integration tests).

**Never cut:** `domain/` unit tests, the tenant isolation tests and their guard, the SQL constraint and race tests, and E2E journeys 1, 3 and 5.

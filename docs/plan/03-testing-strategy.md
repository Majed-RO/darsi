# Darsi (دَرْسي) — Testing Strategy

> Status: **✅ Approved (Stage A revision)** · 2026-11-23
> This revision follows the Stage A scope and architecture v3 (PLAN.md D-24 to D-43). The earlier version (approved 2026-11-22) stays in git history.
> Architecture: [02-architecture.md](02-architecture.md) · Scope: [01-mvp-scope.md](01-mvp-scope.md) · Decisions: [../PLAN.md](../PLAN.md)

## 1. Goals

1. **Every answer is checked correctly.** Wrongly marked answers is a Stage A success criterion with a target of zero (scope, "Success criteria"), so every item template is tested with thousands of generated items.
2. **No answer is lost or counted twice.** Sync, its repeats and the server's re-check are proven by tests, not assumed (D-36, D-37).
3. **Every rule has a test that names it.** Mastery, the learning path, the diagnostic, feedback and PIN lockout are pure functions (architecture section 3), so they're tested exhaustively and fast.
4. **Tenant isolation is proven.** A teacher can't reach another workspace, and a student can't read or send another student's data. A guard test fails if a new use case ships without an isolation test.
5. **The core journeys work in a real browser,** on an Android-sized screen, in Arabic, right to left, **online and offline**.
6. **CI is fast and trustworthy:** under about 10 minutes, no AI calls, no money spent, no flaky tests left in place.
7. **Tests ship with the feature.** Each feature PR includes its tests; a bug fix starts with a failing test.

## 2. Test layers

| Layer | Tool | What it covers | Runs against | When |
|---|---|---|---|---|
| Static checks | ESLint, TypeScript (strict), Prisma checks, **content checks** | Types; the import rules (architecture section 3); schema and migrations in sync; every content file matches its schema; no unreviewed text in a published unit (D-08b) | Source code and `content/` | Every PR |
| **Unit** | **Vitest** (+ **fast-check** for property tests) | `domain/` modules; every item template in `content/`; the phone-side store and queues in `client/` (with an in-memory IndexedDB, `fake-indexeddb`); the AI script's prompt builder | Plain functions; no database, network or clock | Every PR; locally on save |
| **Integration** | **Vitest** | Use cases in `server/<feature>` and the sync Route Handler: authorization, the teacher allowlist, transactions, SQL constraints and the `UPDATE` trigger, repeated syncs, the re-check, tenant isolation | Real **PostgreSQL** in Docker | Every PR |
| **End to end** | **Playwright** | Core journeys for the teacher and the student, **including offline**; accessibility; the practice app's weight; number display | Production build with the service worker, seeded synthetic data, Clerk test instance | Every PR |
| Real phone | By hand, with a checklist | Installing the app, airplane mode, persistent storage, digits and minus signs, font | The son's Android phone with Chrome (D-43) | Before the family test, and after any offline change |
| AI script | By hand, after the owner agrees | Drafting content through OpenRouter (architecture section 7) | OpenRouter, costs money | Never in CI |

**No separate component-test layer.** The practice app's logic lives in pure functions with unit tests, and its behavior in the browser is covered by Playwright. This avoids testing the same thing twice.

## 3. Unit tests

### What each domain module must prove

Time (`now`) and randomness (a seeded generator) are passed in, so every case is a plain function call with fixed inputs.

| Module | Cases that must have a test |
|---|---|
| `rng` | The same seed gives the same sequence; different seeds differ; values stay in the requested range |
| `item` | A template turns a seed into the same item every time; a wrong answer matches the right misconception, or none |
| `answer` | Whole numbers: Arabic-Indic, Persian and Western digits; the minus sign typed in each common form; leading `+`; `-0` equals `0`; spaces; empty input is "no answer", not wrong. Multiple choice: the chosen option's original index, whatever the shuffle |
| `digits` | Conversion is idempotent and changes only digits; display uses Arabic-Indic digits; negative numbers are marked for direction isolation |
| `feedback` | First wrong try → misconception message (or the general one) and a second try; hints on request, at most two; second wrong try → worked solution, and the item ends; a right answer ends the item; hints used before an answer are counted |
| `session` | At most 5 items, all from the chosen skill; no two items the same in one session |
| `skill-map` | A cycle or a missing prerequisite is rejected; the recommended next skill is the first one not mastered whose prerequisites are mastered; when everything is mastered there's no recommendation |
| `diagnostic` | The fixed set of items; 2 of 2 right passes a skill; 1 of 2 doesn't; skills it doesn't cover stay "not started"; it can resume after an interruption |
| `mastery` | Exactly 4 of the last 5 is mastered and 3 isn't; only first tries without a hint count; second tries and hinted answers don't; a skill passed in the diagnostic stays mastered until the first wrong attempt after it, then the last-5 rule applies; mastery drops back; the thresholds come from configuration |
| `attempt` | Which attempts count for mastery; an unmatched wrong answer is recognized |
| `pin` | Locked after the 5th failure for 15 minutes; unlocked at exactly 15 minutes; a success resets the counter; tries during a lockout are refused without being counted; a temporary PIN must be replaced, and the new PIN can't equal it (D-45) |
| `app-lock` | Asks on every opening; asks after exactly 10 minutes without use, not before; the time comes from configuration; locked after the 5th wrong PIN for 15 minutes (D-45) |
| `join-code` | No look-alike characters; typed codes match whatever their case, spaces and digit style |

### Content tests (`content/`)

- **Every item template** has a property test: thousands of random seeds, and for each one the template's answer equals an independent calculation written in the test. The generated numbers stay in their declared range, and the question never shows the answer.
- **Every misconception** has an example test: the wrong answer it describes is matched to it.
- **Every hint and worked solution** belongs to a template that exists, and no hint contains the final answer for the example items.
- **The unit's skill map** passes `skill-map`'s checks, and every skill has a lesson and at least one template.
- **The unit bundle** builds the same way twice from the same files, so its content version is stable.

### Phone-side store (`client/`)

With `fake-indexeddb`:
- Answers go into the current student's queue and survive closing and reopening the store.
- After a sync, only the IDs the server stored leave the queue.
- A queue larger than one batch is sent in several batches in one sync, until it's empty. If a later batch fails, the attempts not yet stored stay in the queue.
- The pull cursor is the latest `receivedAt` the phone has from the server, and attempts received twice (from the overlap window) are saved once. When the server's attempt count differs from the phone's, the phone pulls everything once and ends with the same attempts as the server.
- Switching student keeps the first student's queue, and the second student's view never includes it.
- A stored student missing from the class list is removed from the phone (queue, attempts and PIN hash), and the other students stay. The match is by student ID: a deleted "Ahmad" is removed even when a new "Ahmad" with another ID is on the list. A class code that no longer exists removes all of that class's students. A `401` alone never clears a queue (D-50).

### AI script

- The prompt builder's input type accepts only a template definition (and, later, anonymous patterns from more than one student). A type test proves that student data and free text can't be passed in (D-08a, D-26).
- The output schema is tested on stored example responses; no live calls.

### Property tests (fast-check)

- `answer`: normalizing never changes the value of a valid whole number, and normalizing twice changes nothing.
- `mastery`: for any sequence of attempts, the state matches a simple reference implementation written in the test.
- `skill-map`: for any valid map, the recommended skill always has all its prerequisites mastered.
- `digits`: conversion is idempotent and only ever changes digits.
- Every item template, as above.

## 4. Integration tests (real PostgreSQL)

### Setup

- **Same database as production:** PostgreSQL in Docker, on the same major version as the Neon project. Migrations run once into a template database.
- **One database per Vitest worker,** copied from the template, so test files run in parallel. Tables are emptied between tests.
- **Factories, not shared fixtures:** `makeWorkspace()`, `makeClass()`, `makeSubject()`, `makeStudent()`, `makeAttempt()` create exactly the data each test needs.
- **Clerk is replaced by an injected actor,** so tests pass a teacher or a student directly. The real Clerk connection is covered by E2E.

### What they cover

| Area | Tests |
|---|---|
| **Tenant isolation** | For every use case that takes an ID from the browser: an actor from workspace B using workspace A's ID gets "not found", and nothing changes. A student can't send attempts for another student, and a sync never returns another student's attempts |
| **Isolation guard** | A test lists every exported use case and the sync handler, and fails if one has no isolation case |
| **Teacher allowlist** | A Clerk user whose email isn't on the allowlist gets "not allowed", and no `User`, `Workspace` or `Membership` is created. Two simultaneous first requests from the allowed teacher create one workspace |
| **SQL constraints** | The trigger rejects every `UPDATE` on `Attempt`; `tryNumber` only 1 or 2; unique join code; unique content subject per class; unique student name per class; unique membership per workspace and user; unique session token hash |
| **Sync: no duplicates** | The same attempt sent twice is stored once. Two syncs of the same batch at the same moment store each attempt once. A batch above the limit is refused |
| **Sync: re-check** | A right answer is stored as right on both sides. A phone result that differs from the server's is stored with the server's result and flagged. An attempt made with an outdated template version gets `serverCorrect = null` and is flagged. Parameters that don't match the seed are flagged |
| **Sync: pull** | The answer includes the student's attempts the phone doesn't have yet, and only that student's. **Late upload:** phone A pulls an attempt; phone B then uploads an attempt with an earlier `answeredAt`; phone A's next pull receives it. **Slow commit:** a sync transaction held open past the timeout rolls back and stores nothing. **Count check:** the answer carries the student's attempt count on the server |
| **Sync: sessions** | No session, an expired session or a revoked session gets `401` and stores nothing. A request from another origin is refused |
| Sessions | Only the token's hash is stored; a PIN reset revokes sessions and makes the PIN temporary again |
| **Choosing a PIN (D-45)** | Signing in with a temporary PIN creates no session until a new PIN is chosen; a new PIN equal to the temporary one is refused; after it's chosen, the temporary PIN no longer works, and typing it shows the "already replaced" message without counting as a wrong PIN; a reset clears that, so the old temporary PIN is then just wrong (D-51); no use case returns a student's PIN or its hash to a teacher |
| **Deleting a student** | Removes the student's attempts and sessions in one transaction; a later sync with that student's cookie gets `401`, and the class's list no longer includes the student |

## 5. End-to-end tests (Playwright)

### Environment

- **A production build** (`next build` then `next start`), so the service worker and caching are the real ones.
- **A fresh database per CI run,** migrated and seeded with **synthetic** data only. Tests create their own classes and students.
- **No AI.**
- **A Clerk development instance** with fixed test users: two teachers on the test allowlist and one user who isn't. Clerk's Playwright helpers (`@clerk/testing`).
- **An Android phone profile** (about 360×640), Arabic locale, right to left. **Chromium only** (D-43).
- **Offline** is simulated with Playwright's offline mode for the browser context.

### Journeys

| # | Journey |
|---|---|
| 1 | **Teacher setup:** the allowed teacher signs in, creates a class and its Math subject, adds a student and gets a temporary PIN once. A user who isn't on the allowlist is refused |
| 2 | **Student sign-in:** the class code typed in Arabic-Indic digits, the name, the temporary PIN on Darsi's keypad; the student must choose his own PIN; the practice app opens; the temporary PIN no longer works. 5 wrong PINs lock the student; resetting the PIN ends the old session |
| 3 | **Diagnostic:** the student takes it, sees what he already knows, and gets the recommended next skill |
| 4 | **Practice and feedback:** a wrong answer shows its misconception message and allows another try; hints on request; the worked solution after two wrong tries; the progress screen updates; a skill reaches mastery |
| 5 | **Offline (never cut):** sign in; go offline; reload the practice app and it still opens; practise a full session; the waiting count shows; close and reopen the page; go back online; the answers sync once (the server's count equals the number of answers); a second sync adds nothing |
| 6 | **Switching student:** with answers waiting, switch student; the sign-in screen shows the waiting count; the second student never sees the first one's progress; the first student signs in again and the answers sync |
| 7 | **Deleting a student:** with answers waiting for two students on one phone, the teacher deletes both: one is the current student, the other was switched out. The server's data is gone. The current student's next sync leads to the sign-in screen, and once it opens with a connection, neither student nor their answers remain on the phone (D-50) |
| 8 | **Isolation smoke test:** teacher B opening teacher A's class URL gets a not-found page |
| 9 | **Content update:** after a new content version is deployed, the app picks it up on the next online opening, and new attempts carry the new version |
| 10 | **App lock (D-45):** reopening Darsi asks for the PIN, online and offline; 10 minutes without use locks it (with a simulated clock); 5 wrong PINs lock it for 15 minutes; "Switch student" works from the lock screen |

### Extra checks

- **Accessibility:** axe (`@axe-core/playwright`) on the sign-in, diagnostic, practice, progress and teacher pages; serious or critical problems fail the test.
- **Practice app weight:** the test adds up the JavaScript downloaded on the practice app's first load and fails above **~200 KB** compressed (architecture section 10). It also fails if any student page sends a request to Clerk.
- **Number display:** on the practice screen, a negative number is rendered inside its direction-isolated element, with the minus sign first and Arabic-Indic digits.

### Flaky tests

No fixed waits; only Playwright's auto-waiting assertions. CI retries a failed E2E test once and keeps a trace. A test that passes only on retry is fixed before more feature PRs are merged.

## 6. Real-phone checklist

Run on the son's Android phone with Chrome before the family test, and again after any change to offline behavior:

- [ ] Install Darsi on the home screen and open it from there.
- [ ] Sign in, then switch on airplane mode: the app opens, and a full session works.
- [ ] Close the app while offline, reopen it, and check that the waiting answers are still there.
- [ ] Switch airplane mode off: the answers sync, and the count returns to zero.
- [ ] Check that persistent storage was granted (`navigator.storage.persisted()`).
- [ ] Close Darsi and reopen it, online and in airplane mode: it asks for the PIN each time.
- [ ] Negative numbers, Arabic-Indic digits and the font display correctly.
- [ ] Measure the first load on the real connection.

## 7. CI pipeline (GitHub Actions)

Four jobs run in parallel on every PR, and all must pass before merging:

| Job | Steps |
|---|---|
| Checks | Lint, type check, Prisma schema and migration consistency, content schema and review-state checks |
| Unit | Vitest unit tests (domain, content, client store) with coverage |
| Integration | Vitest integration tests with a Postgres service container |
| E2E | Production build, Postgres service container, migrate and seed, Playwright |

- **Target:** under about 10 minutes in total.
- **Coverage:** `domain/` at **90% or more** of lines and branches, enforced. Every item template must have its property test (a check fails otherwise). Other folders are reported but not enforced.
- **Secrets:** CI only needs the Clerk development instance keys. Pull requests from forks don't get secrets, so their E2E job is skipped.
- **Alongside CI:** Vercel builds a preview deployment, and CodeRabbit reviews the PR.

## 8. Conventions

- **Where tests live:** unit tests next to their code (`*.test.ts`), integration tests as `*.int.test.ts`, E2E tests in `e2e/`.
- **Names describe the rule** in plain English, for example "counts only first tries without a hint toward mastery".
- **Arabic test data** where text matters, to catch right-to-left and Unicode problems early.
- **Synthetic data only.** No real student data, never anything from the family test, in tests, fixtures, seeds or screenshots (D-40). No Ministry or UNRWA material (D-09, D-26).
- **No `Date.now()` or `Math.random()` in `domain/` or `content/`;** lint enforces it, and tests pass fixed values and seeds.

## 9. Not tested in Stage A, and why

| Not tested | Why | When to revisit |
|---|---|---|
| Visual screenshots | Arabic text rendering differs between machines, so screenshot comparisons flake | If UI regressions appear |
| Load and performance under traffic | One family | Before Stage C |
| Browsers other than Chromium | The test phone is Android with Chrome (D-43) | If iPhones join a later stage |
| AI in CI | Costs money and gives different answers each time | Never; the script's builder and schemas are unit-tested |
| Clerk's own screens | A vendor's component; only our use of it is tested | — |
| Mutation testing | Useful for `domain/`, but adds CI time | After Stage A |

## 10. Effort and cut line

Testing is inside the scope's estimate: tests are written in each feature PR, and the shared setup (database per worker, factories, Clerk test setup, offline E2E setup, CI jobs) takes about half a day in the setup PRs.

If time runs short, cut in this order, and list each cut under "Known limitations" in the README:

1. Accessibility checks.
2. The automated weight check (do Lighthouse by hand instead).
3. E2E journeys 7, 8 and 9 (keep their integration tests).

**Never cut:** the property test of every item template; `domain/` unit tests; tenant isolation tests and their guard; sync tests (no duplicates, re-check, sessions); the `UPDATE` trigger test; the PIN-choosing tests; E2E journeys 2 to 6 and 10; the real-phone checklist.

# Darsi (دَرْسي) — Implementation Plan (Stage A)

> Status: **✅ Approved (Stage A)** · 2026-11-23
> This plan builds the Stage A prototype. The earlier 9-day plan for the teacher-first MVP (approved 2026-11-22) stays in git history.
> Scope: [01-mvp-scope.md](01-mvp-scope.md) · Architecture: [02-architecture.md](02-architecture.md) · Testing: [03-testing-strategy.md](03-testing-strategy.md) · Repo and workflow: [04-repo-and-workflow.md](04-repo-and-workflow.md)

## 1. How to read this plan

- **A "day" is one focused day of work, not a calendar date.** The days don't need to be consecutive, and pausing between PRs or days costs nothing (section 6).
- **Two kinds of PR:** code PRs (numbered 1, 2, 3…) and content PRs (C1 to C5), which add lessons, templates, hints and misconception messages to `content/`. Both follow the same routine.
- **Every PR becomes a GitHub issue** (repo and workflow, section 12) and is merged on its own, with its tests, before the next one that depends on it.
- **Each PR's tests follow the testing strategy:** unit tests for its `domain/` modules and templates, integration tests for its use cases (including tenant isolation), and the E2E journeys named in its row. Journey numbers refer to [testing strategy, section 5](03-testing-strategy.md#journeys).
- **Risky things come first.** The early checks from the architecture's risk table (section 14) sit in the first PR that can answer them, so a bad surprise shows up early.

## 2. Before Day 1: what you set up

These need your accounts, so only you can do them. None of them costs money.

| What | Why | Needed by |
|---|---|---|
| GitHub repo `darsi` (public), push `main`, then the `main` ruleset (repo and workflow, sections 2 and 7) | The replanning PR and everything after it | Day 1 |
| Vercel account, linked to the repo; functions region Frankfurt (`fra1`) | Hosting and PR previews | Day 1 |
| Two Neon projects in Frankfurt (`eu-central-1`): `darsi-prod` for the family test's data, not linked to Vercel; `darsi-preview`, linked to Vercel through Neon's integration for previews only (D-49) | Database and preview branches | Day 1 |
| ✅ CodeRabbit, installed on the repo (reviews start by hand for now, step 15) | PR reviews | Day 1 |
| Clerk application (development instance) | Teacher sign-in | Day 3 |
| A dedicated OpenRouter key for Darsi, with a credit limit (D-21, D-23) | The content-writing script | PR 14 |
| Your son's Android phone with Chrome, available for short checks (D-43) | Early checks in PRs 3, 4 and 17, and the real-phone checklist | Day 2 |
| ✅ Displayed digits: Arabic-Indic (D-41) | Decided | — |

**The first PR** is the replanning branch, `docs/replan-stage-a` (repo and workflow, section 2). It merges before the bootstrap PRs.

## 3. A day, step by step

Every day follows the same routine, usually twice: once per PR.

| Who | Does what |
|---|---|
| **You** | Choose what to build, make every decision, read every change, try it on a phone, and merge |
| **The AI assistant** (Claude Code in VS Code) | Reads the plan, proposes how to build the PR, writes the code, tests and content drafts, runs the commands, drafts the PR description and fixes review comments. It asks before pushing, opening a PR, or running any paid AI call |
| **CI** (GitHub Actions) | Runs every check and test on each push. Nothing merges unless it's green |
| **CodeRabbit** | Reviews each PR when you ask it to (step 15) and leaves comments |

### Start (about 10 minutes)

1. Open the project in VS Code and start a Claude Code session. Ask "what's next?": the assistant reads `AGENTS.md` (through `CLAUDE.md`), whose "Current status" names the next PR.
2. Get the latest code: `git switch main`, then `git pull`.
3. On GitHub, check that `main` is green and look at any open PRs, including Dependabot's.
4. On the Project board, move the PR's issue to In progress.

### Plan the PR (about 15 minutes)

5. Ask the assistant to plan it, for example "plan PR 9". It reads the PR's row in section 4, its issue, and the architecture sections it touches, then proposes the files, tables, tests and any open question. You agree or adjust before any code is written.
6. If something needs a decision, decide it now. It's logged as the next D-number in PLAN.md, and also becomes an ADR if it shapes the code.
7. Create the branch, for example `git switch -c feat/student-sign-in`.

### Build (most of the day)

8. Build from the inside out, with tests at each layer: domain rules and their unit tests, then use cases and their integration tests, then pages, then the E2E journey. For a content PR: the templates and their property tests, then the texts, then your review.
9. Read every change the assistant makes, and ask "why?" whenever something isn't clear. In an interview you must be able to explain any file.
   - **As it happens:** in Claude Code's default permission mode, each edit opens in VS Code as a diff that you accept or reject.
   - **Before committing:** VS Code's Source Control view lists every changed file with its diff, including files that tools write (scaffolding, lockfiles, generated migrations).
   - **In the browser:** with `pnpm dev` running, pages reload as soon as a file is saved. Offline behavior needs a production build (`pnpm build && pnpm start`).
10. Run the checks locally (start the database first with `docker compose up`):
    - `pnpm lint` and `pnpm typecheck`
    - `pnpm test` (unit, including content) and `pnpm test:int` (integration)
    - `pnpm test:e2e` for the journeys this PR touches

    The exact script names are set in PRs 1 and 2.
11. Look at the new pages yourself, in the browser's phone view, in Arabic, right to left.
12. Commit in small steps with conventional messages and no AI trailer; the pre-commit hook formats the files. The last commit updates `AGENTS.md` "Current status" to name the next PR, so the status reaches `main` with the merge.

### Open the PR (about 10 minutes, then a wait)

13. The assistant asks before pushing, then runs `git push -u origin <branch>`.
14. It opens the PR with `gh pr create`: a conventional title, the template filled in, "Closes #<issue>", and phone screenshots (synthetic data only) for UI changes. You check the description before it's sent.
15. Ask CodeRabbit for its review: comment `@coderabbitai review` on the PR. It doesn't review this repo on its own yet (repo and workflow, section 10). Then wait about 10 minutes for three things: CI, the Vercel preview link, and CodeRabbit's review.

### Review (20–40 minutes)

16. Open the preview link **on a real phone** and try the feature. For offline changes, also try airplane mode.
17. Go through CodeRabbit's comments one by one. For each, either ask the assistant to fix it or reply with the reason it's fine as it is, then resolve the conversation. Pushing a fix runs CI again; comment `@coderabbitai review` again if the fix is large enough to need a fresh review.
18. Read the "Files changed" tab once more, as if you were reviewing a colleague's work.

### Merge (about 5 minutes)

19. When CI is green, CodeRabbit has posted its review, and every conversation is resolved, click **Squash and merge**, and check that the commit title is the PR title.
    - **Before CI exists** (the replanning PR and PR 1), there's no CI to be green: those PRs merge when their local checks pass (PR 1's lint, type check and lint-rule tests), CodeRabbit has posted its review, and every conversation is resolved. From PR 2 on, green CI is required (repo and workflow, section 1).
    - There's **no "Approve" step**: GitHub doesn't let you approve your own PR, so green CI, CodeRabbit's review and resolved conversations are the gate. GitHub doesn't enforce the review itself, so check that it's there.
20. Merging deploys the private Stage A deployment automatically, and "Closes #…" closes the issue. Check that the deployment succeeded.
21. If the plan marks a tag after this PR, run `gh release create <version> --generate-notes`.

### End (about 5 minutes)

22. `git switch main`, then `git pull`, so the next day starts clean.
23. Stopping in the middle of a PR? Commit, push and open it as a **draft PR** with a note of what's left. Nothing is lost, and `main` is untouched.

The routine around the code takes about an hour per PR; the effort estimates assume it.

### Days that differ

- **Content days** (C1 to C5) are mostly writing and reviewing, not coding. You review every text before the PR merges (D-08b).
- **PR 14** ends with the small AI trial. The assistant asks first, because it costs money; you then choose the model.
- **Dependabot PRs** arrive weekly. Merge them when CI is green, or ask the assistant why one fails.

## 4. The plan

### Day 1 — Foundation → tag `v0.1.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 1 | `chore: bootstrap the Next.js app and tooling` | Next.js 16, TypeScript strict, Tailwind v4, shadcn/ui, Prettier, ESLint with the project rules (repo and workflow, section 5), husky and lint-staged, `env.ts` and `.env.example`, Docker Compose Postgres, pinned Node and pnpm, PR and issue templates, `.coderabbit.yaml`, Dependabot, `SECURITY.md`. **ADR 001, 002, 012** | A small test proves each project lint rule fires | 0.5 |
| 2 | `ci: add the CI pipeline, test setup and deployments` | Prisma, Vitest (unit and integration projects, a database per worker, factories, `fake-indexeddb`), Playwright (production build, Android phone profile, offline helper, axe helper), all CI jobs including the content checks, the coverage threshold, Vercel and Neon connected (the integration on `darsi-preview` for Preview only; Production's `DATABASE_URL` to `darsi-prod` by hand, D-49), migrations in the build. **ADR 003** | The pipeline itself; the first preview and production deploys of an empty app; a check that previews connect to `darsi-preview` and its default branch holds seed data only | 0.5 |

### Day 2 — App shell and the offline spike

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 3 | `feat(ui): add the Arabic right-to-left app shell` | `<html lang="ar" dir="rtl">`, IBM Plex Sans Arabic, `messages/ar.ts`, the route groups (teacher, student sign-in, practice app), the web app manifest, the `digits` module and the number display component. **ADR 010, 011** | Unit and property tests for `digits`; E2E smoke test with axe and the number display check. **Early check:** the font, Arabic-Indic digits and "-٥" on the son's phone | 0.5 |
| 4 | `feat(offline): add the service worker and the phone-side store` | **A spike first:** the service worker caches the static practice app, which opens in airplane mode. Then `client/`: the IndexedDB store with per-student queues, and the sync-state indicator (still without a server). **ADR 013** | Unit tests for the store with `fake-indexeddb`; E2E: the practice app opens offline after a first visit. **Early checks:** the service worker library with Next.js 16; `navigator.storage.persist()` on the son's phone | 0.75 |

### Days 3–5 — Sign-in → tag `v0.2.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 5 | `feat(auth): add teacher sign-in with the allowlist` | Clerk in the teacher pages only, in Arabic; the teacher allowlist from configuration; `User`, `Workspace` and `Membership`; `ensureWorkspace()` with its race handling; `server/auth`. **ADR 004** | Integration tests for the allowlist and the first-sign-in race; E2E journey 1, first part. **Early checks:** Clerk's screens in Arabic right to left; whether Clerk can restrict sign-ups on the free plan | 0.5 |
| 6 | `feat(classes): add classes and subjects` | `Class` (grade 1–12) and `Subject` pointing to a content subject and grade (D-22, D-24); the `join-code` module; teacher pages; the **isolation guard** test. **ADR 006, 014** | Unit tests for `join-code`; isolation tests and the guard; constraint tests; E2E journey 8 | 0.5 |
| 7 | `feat(students): add students, PINs and deletion` | `Student`; one-time temporary PINs generated, hashed and shown once (D-45); resetting a PIN back to a temporary one; deleting a student with all their data | Unique name per class; isolation tests; the deletion transaction; E2E journey 1 completed | 0.5 |
| 8 | `feat(students): add class code, name and PIN sign-in` | `StudentSession`; the `pin` module; Darsi's keypad for the PIN; the student choosing his own PIN at the first sign-in (D-45); digit and case normalization; the session cookie; storing the current student and the PIN's hash for the app lock on the phone; "Not you? Switch student" with each student's waiting count. **ADR 005** | Unit tests for `pin`; integration tests for sessions, revocation and choosing a PIN; E2E journey 2 | 1.0 |

### Days 5–9 — Content model and the practice core

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 9 | `feat(content): add the content model and lesson display` | `content/` with its license file (D-39); Zod schemas for units, skills, lessons and templates; the build of the unit bundle and its content version; the review-state check (D-08b); the `skill-map` module; lessons shown in the practice app. **ADR 007** | Unit tests for `skill-map` and the bundle build; the content checks in CI; a small sample unit written for tests | 1.5 |
| 10 | `feat(practice): add item templates and answer checking` | The `rng`, `item`, `answer` and `attempt` modules; the template interface; the property-test helper every template uses | Unit and property tests for each module | 1.0 |
| 11 | `feat(practice): add the practice screen and keypad` | The `session` module; the practice screen with Darsi's keypad and multiple-choice buttons; answers saved as attempts in the current student's queue; the session summary | Unit tests for `session`; E2E journey 4, first part; the practice app's weight check starts here | 1.0 |
| C1 | `content(integers): add the skill map and the first complete skill` | The draft skill map checked against the UNRWA cards' objectives (V-01); the diagnostic plan; **one skill written completely by hand**: lesson, templates, misconceptions, hints and worked solution. **Time it** | Property tests for its templates. **Early check:** the time per skill; adjust the content estimate in section 5 if needed | 0.75 |

### Days 9–11 — Feedback and sync → tag `v0.3.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 12 | `feat(practice): add feedback, hints and worked solutions` | The `feedback` module: misconception messages, a second try, two hint levels, the worked solution after two wrong tries (D-35); unmatched wrong answers are marked (D-29) | Unit tests for `feedback`; E2E journey 4 continued | 1.0 |
| 13 | `feat(sync): add attempts, sync and the server's re-check` | `Attempt` with its constraints and the `UPDATE` trigger; `POST /api/sync` with push and pull; the session, origin and ownership checks; the re-check and flags; the queue flushed on success; a `401` keeps the queue. **ADR 015** | Integration tests for no duplicates, the re-check, sessions and pull; the trigger test; the isolation guard covers the handler | 1.5 |

### Days 12–14 — The AI script and content

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 14 | `feat(ai): add the content-writing script` | `scripts/content-ai/`: the prompt builder that accepts only a template definition (D-08a, D-26), OpenRouter pinned to Anthropic, structured output validated with Zod, drafts written with `reviewed: false`. **ADR 008** | The type test on the builder's input; schema tests on stored example responses | 0.5 |

**After PR 14:** the assistant asks you before running the small AI trial on a few skills (it costs money). You review the drafts as a teacher and choose the model and effort (Q-15). **ADR 009** records the choice and the real cost.

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| C2 | `content(integers): add the prerequisite skills` | Lessons, templates, misconceptions, hints and worked solutions for the prerequisite skills, drafted with the script and reviewed by you | Property tests for every template | 1.0 |
| C3 | `content(integers): add the number line, opposites, comparing and ordering` | The same for these skills | Property tests for every template | 1.0 |

### Days 15–17 — Diagnostic and progress → tag `v0.4.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 15 | `feat(diagnostic): add the diagnostic test and starting point` | The `diagnostic` module; the diagnostic screens with no hints; resuming after an interruption; the summary of what the student already knows and where he starts | Unit tests for `diagnostic`; E2E journey 3 | 1.5 |
| 16 | `feat(progress): add mastery, the learning path and progress` | The `mastery` module (D-34) with its thresholds in configuration; the recommended next skill and free choice with the prerequisite notice; the progress screen with skill states and counts. **ADR 016** | Unit and property tests for `mastery`, at the exact boundaries; E2E journey 4 completed | 1.5 |

### Days 18–21 — Offline completed and the rest of the content

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 17 | `feat(offline): complete limited offline (L1)` | The unit bundle and templates cached; sync on opening, on reconnecting and after each session; persistent storage requested; the content-update flow; the sign-in screen's waiting counts tied to real queues; the `app-lock` module and lock screen (D-45) | Unit tests for `app-lock`; E2E journeys 5, 6, 7, 9 and 10; the real-phone checklist (testing strategy, section 6), first run | 2.0 |
| C4 | `content(integers): add absolute value and addition` | Lessons, templates, misconceptions, hints and worked solutions | Property tests for every template | 1.0 |
| C5 | `content(integers): add subtraction and word problems, and review the unit` | The last skills, then a full read-through of the unit on the phone | Property tests for every template; the unit is marked published only when every text is reviewed | 0.75 |

### Days 21–22 — Ready for the family test → tag `v1.0.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 18 | `feat(report): add the family test's evaluation script` | `scripts/report/`: sessions, attempts, mastery over time, hint use, unmatched wrong answers, server mismatches and sync statistics, written to `private/` | Unit tests for its calculations on synthetic data | 0.5 |
| 19 | `docs: write the README and run the readiness checks` | The README from the outline (repo and workflow, section 14), including "How this was built"; known limitations; the readiness checklist (section 7) | The real-phone checklist again; Lighthouse on the practice app; the cold-start time of the database | 1.0 |

## 5. Total, buffer and cut line

**Total: about 22.25 focused days:** about 17.75 days of development and about 4.5 days of content.

The development figure matches the scope's size table, which now includes the AI content-writing script (PR 14) and the student-chosen PIN and app lock (PRs 8 and 17, D-45). The content figure is checked after C1, which is timed.

**Keep two buffer days of work** in reserve: the early checks in PRs 3, 4 and 5 can each fail, and the content can take longer than planned. Each early check has a fallback in the architecture's risk table, but a fallback costs time.

If time runs short, cut in this order (the scope's cut line, then the testing strategy's), and list each cut under "Known limitations" in the README:

| Order | Cut | PR | Saves |
|---|---|---|---|
| 1 | The end-of-session summary | 11 | ~0.15 day |
| 2 | Per-skill counts on the progress screen (keep the states) | 16 | ~0.15 day |
| 3 | Fewer templates per skill (at least one each) | C2–C5 | ~0.5–1 day |
| 4 | The evaluation script's summary (keep a raw export) | 18 | ~0.25 day |
| 5 | Accessibility checks | 2, 3 | ~0.1 day |
| 6 | The automated weight check (do Lighthouse by hand) | 11 | ~0.1 day |
| 7 | E2E journeys 7, 8 and 9 (keep their integration tests) | 6, 17 | ~0.2 day |

**Never cut:** correct answer checking and the property test of every template; the server's re-check; no lost or duplicated answers (the sync tests and E2E journeys 2 to 6); the `UPDATE` trigger; the student-chosen PIN and the app lock (D-45, E2E journey 10); PIN lockout; tenant isolation and its guard; the privacy rules (D-40); your review of every piece of content; the real-phone checklist.

## 6. Pausing and resuming

Gaps between days are expected. The plan is ordered by what depends on what, not by date, so a pause never breaks it.

- **Pause at a PR boundary when you can.** Merge the PR, or leave it open with a short note on what's left. `main` only changes through green PRs, so the app is never left half-built.
- **Where things stand is always written down:**
  - the GitHub Project board shows which issue is in progress
  - `AGENTS.md` "Current status" is updated inside every PR, naming the next one, so it reaches `main` with the merge. A new AI session starts from that file, so it picks up correctly even after weeks away.
- **When you come back:**
  1. Update `main`, and rebase any open branch onto it.
  2. Merge or close the Dependabot PRs that arrived (weekly and grouped, so only a few).
  3. Check that CI is green before starting new work.
- **Nothing needs attention while you're away:** the database sleeps and wakes on its own, and AI is charged only when the content script runs.

## 7. Readiness checklist (PR 19)

The build is done, and the family test can start, when every item of the scope's definition of done holds:

- [ ] Every in-scope feature (or its cut, listed in the README) merged through its own PR with CI green
- [ ] `domain/` coverage at 90% or more; every template has its property test; E2E journeys 2 to 6 and 10 pass in CI
- [ ] Every text in the integers unit reviewed, and the unit marked published
- [ ] The private deployment runs on the son's Android phone with Chrome; the real-phone checklist passes
- [ ] The teacher allowlist holds only your email; the OpenRouter key has its credit limit
- [ ] ADRs 001–016 exist (ADR 009 with the trial's numbers)
- [ ] The README describes the current stage, with "How this was built"
- [ ] `git log` contains nothing from `private/`, no secrets, no family-test data and no real name or email
- [ ] The success criteria's numbers are confirmed (Q-11)
- [ ] Tag `v1.0.0` and publish its release notes

## 8. After the build

1. **The family test:** two weeks, at least four practice sessions, with help requests noted (scope, "Success criteria").
2. **The evaluation:** run the evaluation script; write a short results note in `private/`, with the limits of a sample of one.
3. **The decision:** you decide whether to start Stage B, and what to change first. It's recorded as a new D-number in PLAN.md.
4. **Later:** the Stage C prerequisites and the move to a private repo are listed in [PLAN.md](../PLAN.md#future-features) and the [runbook](../runbooks/stage-c-move-to-private-repo.md).

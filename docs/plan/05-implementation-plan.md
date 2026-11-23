# Darsi (دَرْسي) — Implementation Plan

> Status: **✅ Approved** · 2026-11-22
> Scope: [01-mvp-scope.md](01-mvp-scope.md) · Architecture: [02-architecture.md](02-architecture.md) · Testing: [03-testing-strategy.md](03-testing-strategy.md) · Repo and workflow: [04-repo-and-workflow.md](04-repo-and-workflow.md)

## 1. How to read this plan

- **A "day" is one focused day of work, not a calendar date.** The 9 days don't need to be consecutive, and pausing between PRs or days costs nothing (section 6). How long it takes in calendar time depends on your availability.
- **Every PR becomes a GitHub issue** (repo and workflow, section 12) and is merged on its own, with its tests, before the next one that depends on it.
- **Each PR's tests follow the testing strategy:** unit tests for its `domain/` modules, integration tests for its use cases (including tenant isolation), and the E2E journeys named in its row. Journey numbers refer to [testing strategy, section 5](03-testing-strategy.md#journeys).
- **Effort is an estimate per PR,** in days. The early checks from the architecture's risk table (section 13) are placed in the first PR that can answer them, so a bad surprise shows up early, not on the last day.

## 2. Before Day 1: what you set up

These need your accounts, so only you can do them. None of them costs money except the domain.

| What | Why | Needed by |
|---|---|---|
| GitHub repo `darsi` (public), then the `main` ruleset | Repo and workflow, sections 2 and 7 | Day 1 |
| Vercel account, linked to the repo; functions region Frankfurt (`fra1`) | Hosting and PR previews (architecture section 10) | Day 1 |
| Neon project in Frankfurt (`eu-central-1`), with Neon's Vercel integration | Database and preview branches | Day 1 |
| CodeRabbit, installed on the repo | PR reviews | Day 1 |
| Clerk application (development instance first) | Teacher and admin sign-in | Day 3 |
| A dedicated OpenRouter key for Darsi, with a credit limit | AI calls and the evaluation (D-21, D-23) | Day 2 |
| **Decide the displayed digits:** Arabic-Indic (٠١٢) or Western (012) | Architecture section 8 says it's decided before the first UI PR | Day 2 |
| Buy the domain | Clerk's production instance needs it (D-20) | Day 9 |

## 3. A day, step by step

Every day follows the same routine, usually twice: once per PR.

| Who | Does what |
|---|---|
| **You** | Choose what to build, make every decision, read every change, try it on your phone, and merge |
| **The AI assistant** (Claude Code in VS Code) | Reads the plan, proposes how to build the PR, writes the code and tests, runs the commands, drafts the PR description and fixes review comments. It asks before pushing or opening a PR |
| **CI** (GitHub Actions) | Runs every check and test on each push. Nothing merges unless it's green |
| **CodeRabbit** | Reviews each PR and leaves comments |

### Start (about 10 minutes)

1. Open the project in VS Code and start a Claude Code session. Ask "what's next?": the assistant reads `CLAUDE.md`, whose "Current status" names the next PR.
2. Get the latest code: `git switch main`, then `git pull`.
3. On GitHub, check that `main` is green and look at any open PRs, including Dependabot's.
4. On the Project board, move the PR's issue to In progress.

### Plan the PR (about 15 minutes)

5. Ask the assistant to plan it, for example "plan PR 6". It reads the PR's row in section 4, its issue, and the architecture sections it touches, then proposes the files, tables, tests and any open question. You agree or adjust before any code is written.
6. If something needs a decision, decide it now. It's logged as the next D-number in PLAN.md, and also becomes an ADR if it shapes the code.
7. Create the branch, for example `git switch -c feat/classes-and-subjects`.

### Build (most of the day)

8. Build from the inside out, with tests at each layer: domain rules and their unit tests, then use cases and their integration tests, then pages, then the E2E journey.
9. Read every change the assistant makes, and ask "why?" whenever something isn't clear. In an interview you must be able to explain any file.
   - **As it happens:** in Claude Code's default permission mode, each edit opens in VS Code as a diff that you accept or reject.
   - **Before committing:** VS Code's Source Control view lists every changed file with its diff, including files that tools write (scaffolding, lockfiles, generated migrations).
   - **In the browser:** with `pnpm dev` running, pages reload as soon as a file is saved.
10. Run the checks locally (start the database first with `docker compose up`):
    - `pnpm lint` and `pnpm typecheck`
    - `pnpm test` (unit) and `pnpm test:int` (integration)
    - `pnpm test:e2e` for the journeys this PR touches

    The exact script names are set in PRs 1 and 2.
11. Look at the new pages yourself, in the browser's phone view, in Arabic, right to left.
12. Commit in small steps with conventional messages and no AI trailer; the pre-commit hook formats the files. The last commit updates `CLAUDE.md` "Current status" to name the next PR, so the status reaches `main` with the merge.

### Open the PR (about 10 minutes, then a wait)

13. The assistant asks before pushing, then runs `git push -u origin <branch>`.
14. It opens the PR with `gh pr create`: a conventional title, the template filled in, "Closes #<issue>", and phone screenshots for UI changes. You check the description before it's sent.
15. Wait about 10 minutes for three things: CI, the Vercel preview link, and CodeRabbit's review.

### Review (20–40 minutes)

16. Open the preview link **on your real phone** and try the feature.
17. Go through CodeRabbit's comments one by one. For each, either ask the assistant to fix it or reply with the reason it's fine as it is, then resolve the conversation. Pushing a fix runs CI again.
18. Read the "Files changed" tab once more, as if you were reviewing a colleague's work.

### Merge (about 5 minutes)

19. When CI is green and every conversation is resolved, click **Squash and merge**, and check that the commit title is the PR title.
    - There's **no "Approve" step**: GitHub doesn't let you approve your own PR, so green CI and resolved conversations are the gate (repo and workflow, section 7).
20. Merging deploys the demo automatically, and "Closes #…" closes the issue, which moves it to Done. Check that the production deployment succeeded.
21. If the plan marks a tag after this PR, run `gh release create <version> --generate-notes`. It creates the tag and writes release notes from the merged PR titles.

### End (about 5 minutes)

22. `git switch main`, then `git pull`, so the next day starts clean.
23. Stopping in the middle of a PR? Commit, push and open it as a **draft PR** with a note of what's left. Nothing is lost, and `main` is untouched.

The routine around the code takes about an hour per PR; the effort estimates assume it.

### Days that differ

- **Day 1** starts with the first commit made directly on `main`, before the ruleset exists (repo and workflow, section 2). Every change after that goes through a PR.
- **Day 2** ends with the evaluation run. The assistant asks first, because it costs money; you then review the outputs over the next days.
- **Dependabot PRs** arrive weekly. Merge them when CI is green, or ask the assistant why one fails.

## 4. The plan

### Day 1 — Foundation → tag `v0.1.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| — | First commit on `main` (not a PR) | `.gitignore` written and checked first, then `LICENSE`, a short `README.md`, `CLAUDE.md` and `docs/` (repo and workflow, section 2) | `git check-ignore private/business` before committing | — |
| 1 | `chore: bootstrap the Next.js app and tooling` | Next.js 16, TypeScript strict, Tailwind v4, shadcn/ui, Prettier, ESLint with the project rules, husky and lint-staged, `env.ts` and `.env.example`, Docker Compose Postgres, pinned Node and pnpm, PR and issue templates, `.coderabbit.yaml`, Dependabot, `SECURITY.md`. **ADR 001, 002, 012** | A small test proves each project lint rule fires | 0.5 |
| 2 | `ci: add the CI pipeline, test setup and deployments` | Prisma, Vitest (unit and integration projects, a database per worker, factories), Playwright (production build, phone profile, axe helper), all CI jobs and the coverage threshold, Vercel and Neon connected, migrations in the build. **ADR 003** | The pipeline itself; the first preview and production deploys of an empty app | 0.5 |

### Day 2 — App shell and AI groundwork

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 3 | `feat(ui): add the Arabic right-to-left app shell` | `<html lang="ar" dir="rtl">`, IBM Plex Sans Arabic, `messages/ar.ts`, the four route groups with their layouts, a landing page, the `digits` and `money` modules. **ADR 010** | Unit and property tests for `digits`, unit tests for `money`; E2E smoke test of the landing page with axe. **Early check:** the font and diacritics on a real low-end Android phone | 0.35 |
| 4 | `feat(ai): add the AI provider, fake provider and evaluation script` | `AiProvider`, `FakeProvider` with its scenarios, `OpenRouterProvider` pinned to Anthropic, version 1 of the prompts, `ai-output` (loose wire schema and strict per-item schema), the evaluation script with ~20 original Arabic sample materials. **ADR 008** | Unit tests for `ai-output` (with property tests) and the prompt builder; the shared contract tests against the fake | 0.6 |

**After PR 4:** I ask you before running the evaluation (it costs about $10–20). The run is also the **early check** that structured outputs, effort and thinking-token counts work through OpenRouter (D-23), and it measures how slow generation is. You review the outputs as a teacher while Days 3–6 are built, and choose the model and effort before PR 13.

### Day 3 — Teacher sign-in and classes

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 5 | `feat(auth): add teacher sign-in and workspace creation` | Clerk in the teacher and admin layouts only, in Arabic; `User`, `Workspace` and `Membership`; `ensureWorkspace()` with its race handling; keeping the `User` copy current through session-token claims; admins from configuration; `server/auth`. **ADR 004** | Integration tests for the first-sign-in race, the copy refresh and the admin check; E2E journey 1, first part (sign-up → trial). **Early checks:** Clerk's screens in Arabic right to left; session-token claims; a quick test that a Clerk sign-in token isn't stopped by Device Trust, so the D-20 demo plan is confirmed now, not on Day 9 | 0.5 |
| 6 | `feat(classes): add classes and subjects` | `Class` and `Subject` (D-22), the `join-code` module, teacher pages for both, and the **isolation guard** test. **ADR 006, 014** | Unit tests for `join-code`; isolation tests and the guard; constraint tests (unique join code, unique subject name per class); E2E journey 9 | 0.5 |

### Day 4 — Content and rosters → tag `v0.2.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 7 | `feat(content): add topics and materials` | `Topic` and `Material` (with `summaryDraft` and `summaryPublished`); create, edit, delete and order | Isolation tests | 0.35 |
| 8 | `feat(content): add multiple-choice questions` | `Question`; teacher-written questions published directly; edit and delete; shared option validation | Isolation tests; the `SET NULL` test when a material is deleted; E2E journey 1 completed | 0.4 |
| 9 | `feat(students): add class rosters and PINs` | `Student`; generating and hashing PINs; resetting a PIN; the printable roster | Unique name per class; isolation tests | 0.4 |

### Day 5 — Student login and practice

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 10 | `feat(students): add class code, name and PIN login` | `StudentSession`; the `pin` module; digit and case normalization of typed codes and PINs; the session cookie; "Not you? Switch student"; sessions revoked on PIN reset. **ADR 005** | Unit tests for `pin`; integration tests for sessions, revocation and student isolation; E2E journey 4 | 0.5 |
| 11 | `feat(practice): add practice rounds with feedback` | The `practice` module; `Attempt` with its unique `(roundId, questionId)`; the answer flow with server-side checking; the subject picker (skipped when there's one subject); the round summary | Unit and property tests for `practice`; the double-tap test; E2E journey 3, first part; the student page weight and no-Clerk checks start here | 0.6 |

### Day 6 — Progress and AI drafts → tags `v0.3.0` and `v0.4.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 12 | `feat(progress): add mastery stars and class results` | The `mastery` module; stars for students; the teacher's results grid, one subject at a time, with "needs review". Tag `v0.3.0` after merging | Unit tests at the exact thresholds; E2E journey 3 completed, and journey 7 | 0.35 |
| 13 | `feat(ai): add AI drafts with teacher review` | `AiGeneration`; the `ai-quota` module (allowances and the daily cap); the global switch; generating questions and summaries; reviewing drafts (edit, publish, discard); the remaining allowance on screen. Uses the model you chose. **ADR 009** (the evaluation results). Tag `v0.4.0` after merging | Unit tests for `ai-quota`; integration tests for the whole AI flow, including the check that only material text, grade and subject reach the provider (D-08); E2E journey 2 | 0.6 |

### Day 7 — Calendar and access rules

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 14 | `feat(calendar): add the school calendar for the admin` | `AcademicYear` and `Semester` (`date` columns); the `calendar` module (validation, access windows from D-19, the periods a teacher can buy); admin pages. **ADR 007, 011** | Unit and property tests for `calendar`; the `RESTRICT` tests; E2E journey 6 | 0.4 |
| 15 | `feat(billing): add access rules and read-only mode` | The `access` module; the access check on every teacher action; trial and expiry banners; the reminder 14 days before access ends | Unit tests for `access`, including daylight-saving boundaries; an integration test that every teacher action is refused when access has ended | 0.4 |

### Day 8 — Payments and admin → tag `v0.5.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 16 | `feat(billing): add payment notices and admin approval` | The `payment-notice` module; `PaymentNotice` and `Subscription` with their SQL constraints; the teacher's subscription page (periods with dates and placeholder prices, transfer instructions from configuration, the notice form, history, "payment received"); the admin's notice list with approve, reject and the pending counter | Constraint tests (one pending notice, exactly one period); the approval race; E2E journey 5 | 0.6 |
| 17 | `feat(admin): add the workspaces list` | Access status and end date per workspace | Integration test that only admins can open it | 0.15 |

### Day 9 — Demo and launch → tag `v1.0.0`

| # | PR | What's in it | Tests and checks | Effort |
|---|---|---|---|---|
| 18 | `feat(demo): add demo sign-in and the nightly reset` | The demo setting; "Try as teacher" and "Try as admin" with Clerk sign-in tokens; the reset route and Vercel Cron job; the full demo seed (two workspaces, grade 3 Math, written from scratch in Arabic, D-09); demo AI allowance. Clerk's production instance on the domain. **ADR 013** | Integration test that the reset leaves other workspaces alone; E2E journey 8 | 0.5 |
| 19 | `docs: write the README and run the launch checks` | The README from the outline (repo and workflow, section 14), including "How this was built" and screenshots; known limitations; the launch checklist (section 7 below) | Lighthouse on the live student pages; the demo on a real low-end phone; the cold-start time of the demo database | 0.4 |

## 5. Total, buffer and cut line

**Total: about 8.5 focused days of work,** over the 9 days above. That's half a day more than the scope's estimate of about 8. The difference is work the estimate didn't spell out: writing ~20 Arabic sample materials for the evaluation, writing the demo content, and splitting billing into two reviewable PRs.

**Keep one buffer day of work** in reserve. The early checks in PRs 3, 4 and 5 can each fail, and each has a fallback in the architecture's risk table, but a fallback costs time.

If time runs short, cut in this order (the scope's cut line, then the testing strategy's), and list each cut under "Known limitations" in the README:

| Order | Cut | PR | Saves |
|---|---|---|---|
| 1 | The "needs review" flag (keep the grid) | 12 | ~0.1 day |
| 2 | The printable roster (PINs are on the class page) | 9 | ~0.15 day |
| 3 | The admin workspaces list | 17 | ~0.15 day |
| 4 | AI summaries (keep AI questions) | 13 | ~0.2 day |
| 5 | Editing materials and teacher-written questions (keep create and delete) | 7, 8 | ~0.2 day |
| 6 | Property tests | 3, 4, 11, 14 | ~0.2 day |
| 7 | Accessibility checks | 2, 3 | ~0.1 day |
| 8 | The automated student page weight check (do Lighthouse by hand) | 11 | ~0.1 day |
| 9 | E2E journeys 6, 7 and 8 (keep their integration tests) | 14, 12, 18 | ~0.2 day |

Cutting everything saves about 1.4 days. **Never cut:** PIN lockout, tenant isolation and its guard, the audit record on payment decisions, teacher review of AI drafts, `domain/` unit tests, the SQL constraint and race tests, and E2E journeys 1, 3 and 5.

## 6. Pausing and resuming

Gaps between days are expected. The plan is ordered by what depends on what, not by date, so a pause never breaks it.

- **Pause at a PR boundary when you can.** Merge the PR, or leave it open with a short note on what's left. `main` only changes through green PRs, so the app is never left half-built.
- **Where things stand is always written down:**
  - the GitHub Project board shows which issue is in progress
  - `CLAUDE.md` "Current status" is updated inside every PR, naming the next one, so it reaches `main` with the merge. A new AI session starts from that file, so it picks up correctly even after weeks away.
- **When you come back:**
  1. Update `main`, and rebase any open branch onto it.
  2. Merge or close the Dependabot PRs that arrived (weekly and grouped, so only a few).
  3. Check that CI is green before starting new work.
- **Nothing needs attention while you're away:** the database sleeps and wakes on its own, the demo resets itself each night once it's live, and AI is only charged when it's used.

## 7. Launch checklist (PR 19)

Phase 1 is done when every item of the scope's definition of done holds:

- [ ] Every in-scope feature (or its cut, listed in the README) merged through its own PR with CI green
- [ ] `domain/` coverage at 90% or more; E2E journeys 1, 3 and 5 pass in CI
- [ ] The demo is live on the domain: the demo buttons work, student logins from the README work, and the nightly reset has run at least once
- [ ] The AI daily cap and the OpenRouter key's credit limit are set (D-21)
- [ ] ADRs 001–014 exist (ADR 009 with the evaluation numbers)
- [ ] The README has the demo, screenshots, architecture overview and "How this was built"
- [ ] `git log` contains nothing from `private/`, no secrets and no real prices
- [ ] Tag `v1.0.0` and publish its release notes

## 8. After Phase 1

The next steps are already written down: the Phase 2 features and pilot blockers in [PLAN.md](../PLAN.md#future-features), and moving to a private repo in the [runbook](../runbooks/phase-2-move-to-private-repo.md).

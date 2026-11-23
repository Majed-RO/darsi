# Darsi (دَرْسي) — Repo Setup, CI, PR and Commit Conventions

> Status: **✅ Approved** · 2026-11-22
> Architecture: [02-architecture.md](02-architecture.md) · Testing: [03-testing-strategy.md](03-testing-strategy.md) · Decisions: [../PLAN.md](../PLAN.md)

Nothing here is set up yet. It happens in the first PRs, after the whole plan (steps 1–6) is approved.

## 1. Goals

1. **Nothing private ever reaches the public repo:** `private/`, secrets, real prices, bank details (D-17, D-18). Several layers stop it, not just one.
2. **`main` is always deployable.** Every change arrives through a PR with green CI; nobody pushes to `main` directly, including the owner.
3. **The history tells the story.** One PR per feature, conventional commit titles, ADRs for decisions, so a reviewer can follow how Darsi was built.
4. **Setup is cheap to repeat** for the Phase 2 private repo ([runbook](../runbooks/phase-2-move-to-private-repo.md)).

## 2. First commit and bootstrap order

The folder isn't a git repository yet. The order matters, because `private/` already exists next to the docs.

1. **Write `.gitignore` first,** with `private/`, `.env*` (except `.env.example`), `node_modules/`, `.next/`, test reports and coverage.
2. **Check before committing:** `git check-ignore private/business` must print the path, and `git status` must not list anything under `private/`.
3. **First commit on `main`:** `.gitignore`, `LICENSE` (FSL-1.1-ALv2, D-17), a short `README.md`, `CLAUDE.md` and `docs/`. This is the only commit made directly on `main`.
4. **Create the public GitHub repo** `darsi`, push, then turn on the `main` rules (section 7) before anything else.
5. **Bootstrap PR:** the Next.js app skeleton, tooling (section 5), CI (section 9), test infrastructure (testing strategy, section 4), PR template, CodeRabbit config, `SECURITY.md` and ADR 001.

## 3. Keeping private things private

| Layer | What it does |
|---|---|
| `.gitignore` | `private/` and `.env*` files are never staged |
| CI guard | A check fails the PR if any tracked file is under `private/` or looks like an env file other than `.env.example` |
| GitHub secret scanning with push protection | Blocks pushes that contain known secret formats (API keys, tokens). Free for public repos |
| `.env.example` | Lists every setting with placeholder values only, so nobody needs a real `.env` to see what's required |
| Placeholders in docs and seed data | Prices, bank and wallet details are placeholders in the repo; real values live in `private/` and in Vercel's environment settings (D-18) |
| PR template checklist | "No secrets, real prices or `private/` content" is ticked on every PR |

## 4. Folder layout

Application code sits at the repo root, next to the configuration files, so the folder names match the architecture's layers exactly (`app/`, `server/`, `domain/`, `ui/`). Next.js supports this layout and a `src/` folder equally; the two are never mixed.

```
darsi/
├─ app/                       routes (thin): Server Components and Server Actions
│  ├─ (public)/               landing page, demo buttons (D-20)
│  ├─ (student)/              class code, name and PIN login, subjects, practice, progress
│  ├─ (teacher)/              classes, subjects, content, AI drafts, results, subscription
│  ├─ (admin)/                payment notices, workspaces, school calendar
│  └─ api/cron/demo-reset/    nightly demo reset (D-20)
├─ server/
│  ├─ auth/                   resolves the actor: teacher, student or admin
│  ├─ ai/                     AiProvider, OpenRouter and fake providers, prompts/
│  ├─ billing/  calendar/  classes/  students/  content/  practice/  admin/  demo/
│  └─ db.ts                   the Prisma client
├─ domain/                    pure rules: access, calendar, payment-notice, ai-quota, ai-output,
│                             practice, mastery, pin, join-code, digits, money
├─ ui/                        shadcn/ui components and shared UI
├─ messages/ar.ts             every piece of UI text (architecture section 8)
├─ env.ts                     settings validated with Zod at startup
├─ public/                    static files such as icons
├─ prisma/                    schema.prisma, migrations/ (including hand-written SQL), seed/
├─ test/                      factories, per-worker database setup, fake actors
├─ e2e/                       Playwright journeys
├─ scripts/ai-eval/           the model evaluation (run by hand, never in CI)
├─ docs/                      PLAN.md, plan/, adr/, runbooks/
├─ .github/                   workflows/, pull_request_template.md, ISSUE_TEMPLATE/
├─ private/                   git-ignored business notes (D-17)
└─ next.config.ts, package.json, tsconfig.json, eslint.config.mjs, vitest.config.ts,
   playwright.config.ts, docker-compose.yml, .env.example and other configuration files
```

Imports use the `@/` alias from the repo root, for example `@/domain/pin`.

## 5. Tooling

| Concern | Choice | Why |
|---|---|---|
| Node.js | **22 LTS**, pinned in `.nvmrc` and `package.json` `engines` | Matches the local machine, supported by Vercel and Next.js 16, and maintained past Phase 1 |
| Package manager | **pnpm**, version pinned with `packageManager` | Fast installs with CI caching; strict about undeclared dependencies |
| Formatting | **Prettier**, with the Tailwind class-sorting plugin | No style debates in reviews |
| Linting | **ESLint** (flat config): Next.js and TypeScript rules, plus project rules | Project rules: `domain/` may not import Prisma, Next.js, Clerk or the AI client; no `Date.now()` or `Math.random()` in `domain/`; no physical `left`/`right` Tailwind classes (use `ms-*`/`me-*`, architecture section 2) |
| Pre-commit hook | **husky** + **lint-staged**: format and lint only the staged files | Catches small problems in seconds. CI is still the real gate |
| Settings | `env.ts` validates every environment variable with Zod at startup | A missing or malformed setting fails at boot, not halfway through a request |
| Local database | `docker compose up` starts PostgreSQL, same major version as Neon | Matches CI and the testing strategy |
| Dependency updates | **Dependabot**: security alerts, plus weekly grouped updates for minor and patch versions | Updates arrive as normal PRs and go through CI |

## 6. Branches and pull requests

- **Trunk-based:** `main` plus short-lived branches, one per PR, deleted after merging.
- **Branch names:** `<type>/<short-description>`, for example `feat/student-pin-login`, `fix/lockout-timer`, `docs/adr-005`.
- **One PR per feature** (team practice in CLAUDE.md). A PR should be reviewable in about 15 minutes; if the non-test code grows past roughly 400 lines, split it.
- **Squash merge only.** The PR title becomes the single commit on `main`, so the history reads as one line per feature.
- **Every PR includes** its tests, any ADR it needs, and doc updates. The PR template asks for:
  - what changed and why, and the linked issue
  - screenshots on a phone-sized screen for UI changes (right to left)
  - how it was tested
  - a checklist: tests added; `CLAUDE.md` "Current status" names the next PR; no secrets, real prices or `private/` content; UI text in `messages/ar.ts`; nothing about students sent to the AI (D-08); ADR added or updated if a decision changed

## 7. Rules on `main` (GitHub ruleset)

- Changes only through pull requests; **no direct pushes, and no bypass for admins**, so the owner follows the same rules.
- Required status checks: the CI jobs in section 9, and the branch must be up to date with `main`.
- All review conversations resolved before merging (CodeRabbit's included).
- Linear history (squash merges only); no force pushes; `main` can't be deleted.
- Required approving reviews: **none**. In a one-person repo GitHub doesn't let you approve your own PR, so the gate is green CI plus resolved CodeRabbit comments.

## 8. Commit and PR title conventions

**Conventional Commits**, in English. Because PRs are squash-merged, the **PR title** is what lands on `main`, so CI checks the PR title's format.

```
<type>(<scope>): <what changed, in the imperative>
```

| Type | Use for |
|---|---|
| `feat` | A user-visible feature |
| `fix` | A bug fix |
| `test` | Tests only |
| `refactor` | Code change with no behavior change |
| `docs` | Docs and ADRs |
| `ci`, `build`, `chore` | Pipeline, build and dependency or tooling work |
| `perf` | Performance |

**Scopes** follow the feature folders: `auth`, `billing`, `calendar`, `classes`, `students`, `content`, `practice`, `progress`, `ai`, `admin`, `demo`, `db`, `ui`.

Examples:
- `feat(students): add PIN lockout after 5 failed attempts`
- `fix(practice): save a double-tapped answer only once`
- `docs(adr): record ADR 005 on student login`

**AI assistance is disclosed once, in the README** ("How this was built", section 14), not in commits. Commits and squash-merge messages don't carry a `Co-Authored-By` line for the AI assistant.

## 9. CI and deployments

**On every PR** (GitHub Actions), all required before merging:

| Job | What it runs |
|---|---|
| PR title | Checks the Conventional Commits format |
| Private files guard | Section 3 |
| Checks | Lint, type check, Prisma schema and migration consistency |
| Unit | Vitest unit tests with coverage (`domain/` at 90%+) |
| Integration | Vitest with a PostgreSQL service container |
| E2E | Production build, migrate and seed, Playwright |
| CodeQL | GitHub's code scanning for security problems; free for public repos |

Jobs run in parallel, with pnpm and Playwright browser caching, targeting under about 10 minutes (testing strategy, section 7).

**Alongside CI:**
- **Vercel** builds a preview deployment for each PR, with its own Neon database branch (architecture section 10).
- **CodeRabbit** reviews the PR (section 10).

**Migrations:** the Vercel build runs `prisma migrate deploy` before `next build`. A preview migrates its own Neon branch; production migrates the demo database when a PR merges. A failing migration fails the deployment, so a broken schema never goes live. Phase 1 only has seed data; before the pilot, migrations must also be backward compatible (add first, remove in a later PR).

**Production:** merging to `main` deploys the demo automatically. The nightly demo reset runs as a Vercel Cron job (D-20).

## 10. Code review with CodeRabbit

CodeRabbit is free for public repos. A `.coderabbit.yaml` file gives it English reviews and project-specific instructions by path:

| Path | What CodeRabbit is told to check |
|---|---|
| `domain/**` | No I/O imports; `now` and randomness passed in; every exported rule has unit tests |
| `server/**` | Every query scoped by the actor's workspace or class; every use case has an isolation test (testing strategy, section 4) |
| `app/**` | Thin routes that call one use case; UI text from `messages/ar.ts`; right-to-left-safe classes; no Clerk in `(student)` routes |
| `server/ai/**` | Nothing about students in prompts (D-08); output checked by `ai-output` |
| `prisma/migrations/**` | Hand-written SQL constraints each have an integration test |

The owner reads every comment, then fixes it or replies with the reason, and resolves the conversation. Merging waits for that and for green CI.

## 11. ADRs

- **Location:** `docs/adr/NNN-short-title.md`, numbered in order (`001-record-architecture-decisions.md`).
- **Template:** Status (Proposed, Accepted, or Superseded by ADR-NNN), Context, Decision, Consequences, and links to the PLAN.md decisions it comes from.
- **When:** ADR 001 in the bootstrap PR; each other ADR in the PR that first implements it (architecture section 12 lists 001–014).
- **Never edited after acceptance,** except to mark it superseded. A change of mind is a new ADR that replaces the old one.

## 12. Issues and tracking

- **One GitHub issue per planned PR,** created from the implementation plan (step 6), each linked from its PR ("Closes #12").
- **A GitHub Project board** with To do, In progress and Done, so progress is visible to reviewers.
- **Labels:** the commit types, plus `phase-2` for ideas parked for later.
- Phase 2 ideas from PLAN.md stay in PLAN.md; they become issues only when Phase 2 starts.

## 13. Releases

- **Tag versions at milestones:** `v0.1.0` at the first deploy, then one per group of features, and `v1.0.0` at the end of Phase 1. The Phase 2 runbook adds the `phase-1` tag.
- **Release notes** are generated by GitHub from the merged PR titles.
- **Why it matters for the license:** under the FSL, each release becomes Apache 2.0 two years after it's published (D-17), so tagged releases give clear dates.

## 14. README outline

The definition of done asks for a professional README (scope, "Phase 1 definition of done"):

1. What Darsi is, in two sentences, with a screenshot
2. **Try the demo:** the link, the "Try as teacher" and "Try as admin" buttons, and the student class code and PINs (D-20)
3. Features in Phase 1, and known limitations (including anything cut)
4. Architecture overview: the layer diagram and a link to the architecture document and ADRs
5. Testing: the layers, how to run them, and the CI badge
6. Running locally: prerequisites, `docker compose up`, `pnpm install`, migrate, seed, `pnpm dev` (with the fake AI by default)
7. Project practices: PRs, conventional commits, CodeRabbit, ADRs
8. **How this was built:** the owner planned the product and made the decisions (the PLAN.md decisions log and the ADRs); an AI assistant was used as a pair programmer under the owner's direction; every change went through tests, CI, CodeRabbit and the owner's review
9. License (FSL-1.1-ALv2) and what it allows

## 15. Effort

About **one day**, spread over the first commit and the bootstrap PR: repo and rules, tooling, CI jobs, test infrastructure, templates and configs. It's already inside the ~8-day estimate ("setup" in the scope's realism notes) and includes the half day of test infrastructure from the testing strategy.

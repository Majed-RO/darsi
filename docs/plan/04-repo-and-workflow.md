# Darsi (دَرْسي) — Repo Setup, CI, PR and Commit Conventions

> Status: **✅ Approved (Stage A revision)** · 2026-11-23
> This revision follows architecture v3 and the Stage A testing strategy. The earlier version (approved 2026-11-22) stays in git history.
> Architecture: [02-architecture.md](02-architecture.md) · Testing: [03-testing-strategy.md](03-testing-strategy.md) · Decisions: [../PLAN.md](../PLAN.md)

## 1. Goals

1. **Nothing private ever reaches the public repo:** `private/`, secrets, real prices, bank details, the family test's data, the owner's real name and email (D-17, D-18, D-40, D-42). Several layers stop it, not just one.
2. **`main` is always deployable.** Every change after the first commit arrives through a PR; nobody pushes to `main` directly, including the owner. Every PR needs green CI once CI exists: the two PRs before it (the replanning PR and PR 1, the app skeleton) merge on the local checks they have, CodeRabbit's review and resolved conversations. The CI gate starts with PR 2, which adds the pipeline, and its checks then become required on `main` (section 7).
3. **The history tells the story.** One PR per feature, conventional commit titles, ADRs for decisions, so a reviewer can follow how Darsi was built.
4. **Setup is cheap to repeat** for the private repo at the start of Stage C ([runbook](../runbooks/stage-c-move-to-private-repo.md)).

## 2. First commit and bootstrap order

1. ✅ **`.gitignore` written first,** with `private/`, `.env*` (except `.env.example`), `node_modules/`, `.next/`, test reports and coverage, and checked with `git check-ignore`.
2. ✅ **Local git identity for this repo:** the owner's GitHub name and GitHub's no-reply email, so the owner's real name and email never appear in commits (D-42). On GitHub, "Keep my email addresses private" and "Block command line pushes that expose my email" are on.
3. ✅ **First commit on `main`:** `.gitignore`, `LICENSE` (FSL-1.1-ALv2, D-17), a short `README.md`, `CLAUDE.md` and `docs/`, as approved on 2026-11-22. This is the only commit made directly on `main`.
4. ✅ **The replanning branch** `docs/replan-stage-a` holds the Stage A revision of the plan, one commit per document.
5. ✅ **Create the public GitHub repo** `darsi`, push `main`, then turn on the `main` rules (section 7). Required status checks are added once the CI PR has merged, because the checks don't exist before it.
6. ✅ **First PR:** push `docs/replan-stage-a` and open it as `docs: replan for Stage A`.
7. **Bootstrap PRs:** the app skeleton and tooling (section 5), then CI (section 9) with the test infrastructure (testing strategy, section 4), PR template, CodeRabbit config, `SECURITY.md` and ADR 001.

## 3. Keeping private things private

| Layer | What it does |
|---|---|
| `.gitignore` | `private/` and `.env*` files are never staged |
| CI guard | A check fails the PR if any tracked file is under `private/` or looks like an env file other than `.env.example` |
| GitHub secret scanning with push protection | Blocks pushes that contain known secret formats (API keys, tokens). Free for public repos |
| GitHub email privacy | Blocks pushes whose commits expose the owner's private email (D-42) |
| `.env.example` | Lists every setting with placeholder values only, including the teacher allowlist |
| Synthetic data only | Seeds, fixtures, test data and README screenshots never contain data from the family test (D-40) |
| Original content only | No Ministry or UNRWA text, examples, exercises or images in `content/` (D-09, D-26) |
| PR template checklist | Ticked on every PR (section 6) |

## 4. Folder layout

Application code sits at the repo root, next to the configuration files, so the folder names match the architecture's layers exactly.

```
darsi/
├─ app/                       routes (thin)
│  ├─ (teacher)/              server-rendered teacher pages: class, subject, students, PINs, deleting a student
│  ├─ (student)/
│  │  ├─ sign-in/             class code, name and PIN; server-rendered, online only
│  │  └─ (practice)/          the static practice app: lessons, diagnostic, practice, progress (offline)
│  └─ api/sync/               the only Route Handler: push and pull of attempts (D-36)
├─ server/
│  ├─ auth/                   resolves the actor: teacher (with the allowlist) or student
│  ├─ classes/  students/  sync/
│  └─ db.ts                   the Prisma client
├─ client/                    browser-only code: IndexedDB store, per-student queues, sync client,
│                             content loading, service worker registration
├─ domain/                    pure rules shared by phone and server: rng, item, answer, digits, feedback,
│                             session, skill-map, diagnostic, mastery, attempt, pin, app-lock, join-code
├─ content/                   core content (D-39): LICENSE.md, then <subject>/<grade>/<unit>/
├─ ui/                        shadcn/ui components and shared UI, including the number display component
├─ messages/ar.ts             every piece of UI text
├─ env.ts                     settings validated with Zod at startup
├─ public/                    static files such as icons and the web app manifest
├─ prisma/                    schema.prisma, migrations/ (including hand-written SQL), seed/ (synthetic)
├─ test/                      factories, per-worker database setup, fake actors
├─ e2e/                       Playwright journeys, including offline
├─ scripts/
│  ├─ content-ai/             the AI content-writing script (run by hand, never in CI)
│  └─ report/                 the family test's evaluation script; its output goes to private/
├─ docs/                      PLAN.md, plan/, adr/, runbooks/
├─ .github/                   workflows/, pull_request_template.md, ISSUE_TEMPLATE/
├─ private/                   git-ignored (D-17)
└─ next.config.ts, package.json, tsconfig.json, eslint.config.mjs, vitest.config.ts,
   playwright.config.ts, docker-compose.yml, .env.example and other configuration files
```

Imports use the `@/` alias from the repo root, for example `@/domain/mastery`.

## 5. Tooling

| Concern | Choice | Why |
|---|---|---|
| Node.js | **22 LTS**, pinned in `.nvmrc` and `package.json` `engines` | Matches the local machine, supported by Vercel and Next.js 16 |
| Package manager | **pnpm**, version pinned with `packageManager` | Fast installs with CI caching; strict about undeclared dependencies |
| Formatting | **Prettier**, with the Tailwind class-sorting plugin | No style debates in reviews |
| Linting | **ESLint** (flat config): Next.js and TypeScript rules, plus project rules | Project rules: `domain/` and `content/` may not import Prisma, Next.js, Clerk, browser APIs or the AI client; no `Date.now()` or `Math.random()` in `domain/` or `content/`; `client/` may not import `server/`; the practice app may not use Server Actions; no physical `left`/`right` Tailwind classes (use `ms-*`/`me-*`) |
| Pre-commit hook | **husky** + **lint-staged**: format and lint only the staged files | Catches small problems in seconds. CI is still the real gate |
| Settings | `env.ts` validates every environment variable with Zod at startup | A missing or malformed setting fails at boot |
| Local database | `docker compose up` starts PostgreSQL, same major version as Neon | Matches CI and the testing strategy |
| Dependency updates | **Dependabot**: security alerts, plus weekly grouped updates for minor and patch versions | Updates arrive as normal PRs and go through CI |

## 6. Branches and pull requests

- **Trunk-based:** `main` plus short-lived branches, one per PR, deleted after merging.
- **Branch names:** `<type>/<short-description>`, for example `feat/student-pin-login`, `fix/sync-duplicate`, `docs/adr-015`.
- **One PR per feature.** A PR should be reviewable in about 15 minutes; if the non-test code grows past roughly 400 lines, split it. Content PRs (lessons and templates for a few skills) follow the same rule.
- **Squash merge only.** The PR title becomes the single commit on `main`.
- **Every PR includes** its tests, any ADR it needs, and doc updates. The PR template asks for:
  - what changed and why, and the linked issue
  - screenshots on a phone-sized screen for UI changes (right to left, synthetic data only)
  - how it was tested, including offline when the practice app changed
  - a checklist: tests added; `AGENTS.md` "Current status" names the next PR; no secrets, real prices or `private/` content; no data from the family test (D-40); no Ministry or UNRWA text (D-26); AI-written content reviewed (D-08b); no student data in AI prompts (D-08a); UI text in `messages/ar.ts`; ADR added or updated if a decision changed

## 7. Rules on `main` (GitHub ruleset)

- Changes only through pull requests; **no direct pushes, and no bypass for admins**, so the owner follows the same rules.
- Required status checks: the CI jobs in section 9, and the branch must be up to date with `main`. Added after the CI PR merges.
- All review conversations resolved before merging (CodeRabbit's included).
- Linear history (squash merges only); no force pushes; `main` can't be deleted.
- Required approving reviews: **none**. In a one-person repo GitHub doesn't let you approve your own PR, so the gate is green CI plus CodeRabbit's review with its comments resolved (section 10).

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
| `content` | Lessons, templates, hints and other files in `content/` |
| `ci`, `build`, `chore` | Pipeline, build and dependency or tooling work |
| `perf` | Performance |

**Scopes** follow the feature folders: `auth`, `classes`, `students`, `practice`, `diagnostic`, `progress`, `offline`, `sync`, `ai`, `db`, `ui`, and for `content` the unit's name (for example `integers`).

Examples:
- `feat(students): add PIN lockout after 5 failed attempts`
- `fix(sync): store a repeated attempt only once`
- `content(integers): add the absolute value skill`
- `docs(adr): record ADR 015 on attempts and sync`

**AI assistance is disclosed once, in the README** ("How this was built", section 14), not in commits. Commits and squash-merge messages don't carry a `Co-Authored-By` line for the AI assistant.

## 9. CI and deployments

**On every PR from PR 2 on** (GitHub Actions), all required before merging. The two PRs before CI exists follow the exception in section 1, goal 2:

| Job | What it runs |
|---|---|
| PR title | Checks the Conventional Commits format |
| Private files guard | Section 3 |
| Checks | Lint, type check, Prisma schema and migration consistency, content schema and review-state checks |
| Unit | Vitest unit tests with coverage (`domain/` at 90%+, a property test for every template) |
| Integration | Vitest with a PostgreSQL service container |
| E2E | Production build, migrate and seed, Playwright (including offline) |
| CodeQL | GitHub's code scanning for security problems; free for public repos |

Jobs run in parallel, with pnpm and Playwright browser caching, targeting under about 10 minutes.

**Alongside CI:**
- **Vercel** builds a preview deployment for each PR, with its own Neon database branch in the `darsi-preview` project, copied from that project's synthetic-only default branch. Production lives in a separate Neon project, `darsi-prod`, that previews can't branch from (D-49).
- **CodeRabbit** reviews the PR (section 10).

**Migrations:** the Vercel build runs `prisma migrate deploy` before `next build`. A failing migration fails the deployment. From Stage A on, the production database holds the family test's data, so migrations must be backward compatible: add first, remove in a later PR.

**Production:** merging to `main` deploys the private Stage A deployment automatically. During the family test, a merge that changes content or the practice app is checked on the real phone (testing strategy, section 6).

## 10. Code review with CodeRabbit

CodeRabbit is free for public repos. A `.coderabbit.yaml` file gives it English reviews and project-specific instructions by path.

**Reviews start by hand for now.** CodeRabbit is installed on the owner's account, so it sees every new repo, but it skips automatic reviews on open-source repos with fewer than 10 stars (seen on the first PR, 2026-11-25). So the owner asks for each review by commenting `@coderabbitai review` on the PR (implementation plan, section 3, step 15). Merging waits for that review too; GitHub doesn't enforce it, so the owner checks that it's there. Once the repo has 10 stars, check whether reviews start on their own; if they do, the step can be dropped.

The instructions by path:

| Path | What CodeRabbit is told to check |
|---|---|
| `domain/**` | No I/O imports; `now` and randomness passed in; every exported rule has unit tests |
| `content/**` | Original text only, nothing copied from Ministry or UNRWA materials; every template has a property test; AI-drafted text is marked reviewed before a unit is published; nothing names math or grade 7 outside its folder |
| `client/**` | No imports from `server/`; each student's queue and data kept separate |
| `app/api/sync/**`, `server/sync/**` | Session, origin and student ownership checked; every attempt validated and re-checked; inserts are idempotent |
| `server/**` | Every query scoped by the actor's workspace or student; every use case has an isolation test |
| `app/**` | Thin routes that call one use case; UI text from `messages/ar.ts`; right-to-left-safe classes; no Clerk in student routes; no Server Actions in the practice app |
| `scripts/content-ai/**` | No student data and no Ministry or UNRWA text in prompts (D-08a, D-26); output validated |
| `prisma/migrations/**` | Hand-written SQL constraints and triggers each have an integration test; migrations are backward compatible |

The owner reads every comment, then fixes it or replies with the reason, and resolves the conversation. Merging waits for that and for green CI.

## 11. ADRs

- **Location:** `docs/adr/NNN-short-title.md`, numbered in order (`001-record-architecture-decisions.md`).
- **Template:** Status (Proposed, Accepted, or Superseded by ADR-NNN), Context, Decision, Consequences, and links to the PLAN.md decisions it comes from.
- **When:** ADR 001 in the first bootstrap PR; each other ADR in the PR that first implements it (architecture section 13 lists 001–016).
- **Never edited after acceptance,** except to mark it superseded. A change of mind is a new ADR that replaces the old one.

## 12. Issues and tracking

- **One GitHub issue per planned PR,** created from the implementation plan, each linked from its PR ("Closes #12").
- **A GitHub Project board** with To do, In progress and Done, so progress is visible to reviewers.
- **Labels:** the commit types, plus `later` for ideas parked for later.
- Ideas on PLAN.md's "Later" list stay there; they become issues only when their stage starts.

## 13. Releases

- **Tag versions at milestones:** `v0.1.0` at the first deploy, then one per group of features, and `v1.0.0` when Stage A's build is done, before the family test.
- **Release notes** are generated by GitHub from the merged PR titles.
- **Why it matters for the license:** under the FSL, each release becomes Apache 2.0 two years after it's published (D-17), so tagged releases give clear dates.

## 14. README outline

1. What Darsi is, in two sentences, with a screenshot (synthetic data)
2. Current stage and what works now, and known limitations (including anything cut)
3. Architecture overview: the layer diagram and a link to the architecture document and ADRs
4. Testing: the layers, how to run them, and the CI badge
5. Running locally: prerequisites, `docker compose up`, `pnpm install`, migrate, seed, `pnpm dev`
6. Project practices: PRs, conventional commits, CodeRabbit, ADRs
7. **How this was built:** the owner planned the product and made the decisions (the PLAN.md decisions log and the ADRs); an AI assistant was used as a pair programmer under the owner's direction; every change went through CodeRabbit and the owner's review, and every change from PR 2 on also went through tests and CI (the two earlier PRs came before CI existed)
8. License: FSL-1.1-ALv2 for the code, and what it allows; the content in `content/` is all rights reserved (D-39)

There's no public demo in Stage A (D-40). A demo section comes back with the public demo (D-20).

## 15. Effort

About **one day**, spread over the first commit and the bootstrap PRs: repo and rules, tooling, CI jobs, test infrastructure, templates and configs. It's inside the scope's estimate ("Foundation").

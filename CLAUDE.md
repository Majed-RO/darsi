# CLAUDE.md — Darsi (دَرْسي)

Instructions for AI coding sessions on this project. This file will be public once the repo is published, so keep private details out of it (see "Rules that must not be broken").

## What this project is

Darsi is an Arabic, mobile-first learning and practice platform for elementary schools (grades 1–6) in Palestine, where many schools are out of service.
- **Teachers** publish materials and multiple-choice practice questions.
- An **AI assistant** drafts summaries and quizzes, and the teacher reviews them before students see anything.
- **Students** log in with a class code, their name and a PIN, practise, and see their progress.

It's also a portfolio project, so engineering quality matters more than the number of features.

## Start here

1. [docs/PLAN.md](docs/PLAN.md): background, phases, **decisions log (D-01 to D-23)**, open questions, future features, planning status.
2. [docs/plan/01-mvp-scope.md](docs/plan/01-mvp-scope.md): the **approved** Phase 1 scope (v3, amended by D-19 to D-23).
3. [docs/plan/02-architecture.md](docs/plan/02-architecture.md): the **approved** architecture and data model (v2).
4. [docs/plan/03-testing-strategy.md](docs/plan/03-testing-strategy.md): the **approved** testing strategy.
5. [docs/plan/04-repo-and-workflow.md](docs/plan/04-repo-and-workflow.md): the **approved** repo setup, CI, PR and commit conventions.
6. [docs/plan/05-implementation-plan.md](docs/plan/05-implementation-plan.md): the **approved** implementation plan: PRs by day, the daily routine, pausing and resuming.
7. [docs/runbooks/](docs/runbooks/): procedures for rare operations, such as moving to a private repo at the start of Phase 2.
8. `private/`: business notes (costs and prices). **Never commit it.**

## Current status (2026-11-22)

**Planning is complete (all six steps approved). No code has been written yet.**

| Step | Status |
|---|---|
| 1. Project idea | ✅ Decided |
| 2. MVP scope | ✅ Approved |
| 3. Architecture and data model | ✅ Approved (v2) |
| 4. Testing strategy | ✅ Approved |
| 5. Repo setup, CI, PR and commit conventions | ✅ Approved |
| 6. Implementation plan (small PRs, day by day) | ✅ Approved |

**Next action:** the owner finishes the setup in section 2 of `docs/plan/05-implementation-plan.md` (accounts, and the displayed-digits decision before Day 2). Then **Day 1** starts: the first commit on `main` (`.gitignore` written and checked before anything else, because `private/` sits next to the docs), followed by PR 1.

## Working agreement

- **One step at a time.** Present a planning step, then wait for the owner's reply before moving on.
- **The whole plan (steps 1–6) was approved on 2026-11-22.** Build in the order of `docs/plan/05-implementation-plan.md`, following its daily routine (section 3).
- **Be direct.** If an idea, scope or timeline seems unrealistic, say so and explain the cost.
- When the owner asks to discuss questions one by one, present one question with options and a recommendation, then wait.
- **Record every decision** in the PLAN.md decisions log (the next number is **D-24**) and keep its planning status table current. Step details go in `docs/plan/`. During implementation, technical decisions become ADRs in `docs/adr/` (the list of planned ADRs is in section 12 of the architecture document).
- Plans are written in clear English; the product UI is Arabic.
- **The owner must see every code change:** edit code with the editor's edit tools, never with shell scripts, so each change appears as a diff in VS Code. Files that tools write themselves (scaffolding, lockfiles, generated migrations) are reviewed in the Source Control view instead.
- **The plan's days are focused days of work, not consecutive calendar days;** the owner sometimes pauses. During implementation, every PR updates "Current status" as its last commit (what it completes and the next PR), so the status reaches `main` with the merge and work resumes cleanly after a pause ([implementation plan, section 5](docs/plan/05-implementation-plan.md)).

## Rules that must not be broken

- **`private/` must be in `.gitignore` before the first commit** and must never be pushed. No real prices, bank or wallet account details, contacts or business plans in git (D-17, D-18). The public repo uses placeholder prices.
- **License:** the repo will be public under **FSL-1.1-ALv2**, and the `LICENSE` file is added in the first PR (D-17).
- **Seed and demo content is written from scratch.** No ministry materials in the repo (D-09).
- **No real student data in Phase 1.** Nothing about students is ever sent to the AI (D-08).
- **The owner chooses the AI model** after an evaluation run (architecture section 7). The run costs money, so **ask before running it**. Candidates: Claude Opus 5.5 (default), Sonnet 5.5 and Haiku 5.5, called **through OpenRouter** by their OpenRouter model IDs (D-23).
- **Team practices:** one PR per feature, conventional commits, CI (lint, typecheck, tests) on every PR, review by CodeRabbit.
- **No AI `Co-Authored-By` trailer** in commits or squash-merge messages, as the owner decided. AI assistance is disclosed once, in the README's "How this was built" section ([repo and workflow, sections 8 and 14](docs/plan/04-repo-and-workflow.md)).

## Product essentials

Details are in the scope document and the decisions log.

- Phase 1 has **one teacher per workspace**. Schools with several teachers are Phase 2 (D-13).
- A **class is a group of children** (one login per child); **subjects** inside it carry the teacher and the topics (D-22). Students are not Clerk users: a student is a first name + PIN in one class.
- A **14-day trial** starts automatically (D-07).
- **Subscriptions** are per semester or school year, tied to a school calendar the admin maintains, with no proration (D-14). Each period's access starts where the previous one ends (D-19).
- **Manual payments** (Bank of Palestine, PalPay, JawwalPay) are verified by the platform admin (D-06). The workspace stays read-only while a payment waits (D-16).
- **AI** writes drafts only, and the teacher reviews them (D-08). There's a shared AI allowance per subscription period (D-15), plus a daily cap across all workspaces and a credit limit on the OpenRouter key (D-21, D-23).
- **Public demo:** "Try as teacher/admin" buttons using Clerk sign-in tokens, a nightly reset, our own domain; off in the pilot (D-20).
- **The UI is Arabic, right-to-left only** (D-04).

## Stack (approved with the architecture)

- **App:** Next.js 16 (App Router), React 19, TypeScript strict
- **UI:** Tailwind CSS v4 + shadcn/ui
- **Data and validation:** Prisma + PostgreSQL (Neon), Zod
- **Login:** Clerk for teachers and admin; our own PIN sessions for students
- **AI:** Claude models through OpenRouter (D-23) with structured outputs, behind an `AiProvider` interface with a fake implementation for tests
- **Testing, CI and hosting:** Vitest, Playwright, GitHub Actions, Vercel

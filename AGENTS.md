# AGENTS.md — Darsi (دَرْسي)

Instructions for AI coding sessions on this project. `CLAUDE.md` only imports this file, so edit this one (D-42). It will be public once the repo is published, so keep private details out of it (see "Rules that must not be broken").

## What this project is

Darsi is an Arabic, mobile-first learning and practice platform for students in Palestine, meant to cover several subjects and grades. It starts with one math unit in grade 7, chosen for testing, not as a limit (D-25, D-26). Don't hard-code math or grade 7 into the data or content model (D-24). It follows a limited hybrid model (D-24):
- **Students** take a short diagnostic test, then practise the skills they're missing, including prerequisites from earlier grades, with feedback and hints that help them think. Practice keeps working when the connection drops.
- **Teachers** follow their classes' progress and step in when needed. **Guardians** do the same for their own children, who can also learn on their own (Stage B, D-44). Upper-grade students will later be able to sign up alone as independent learners (D-46).
- **AI** helps write the content, and a person reviews everything before students see it. Students don't use AI in the prototype (D-29, D-30).

Darsi is supplemental: no official grades, and no replacement for official assessment. It's also a portfolio project, so engineering quality matters more than the number of features.

## Start here

1. [docs/PLAN.md](docs/PLAN.md): background, stages, **decisions log (D-01 to D-48)** with the current status of D-01 to D-23, open questions, facts to verify, future features, planning status.
2. [docs/plan/01-mvp-scope.md](docs/plan/01-mvp-scope.md): the **approved** Stage A scope (v4).
3. [docs/plan/02-architecture.md](docs/plan/02-architecture.md): the **approved** architecture and data model (v3).
4. [docs/plan/03-testing-strategy.md](docs/plan/03-testing-strategy.md): the **approved** testing strategy.
5. [docs/plan/04-repo-and-workflow.md](docs/plan/04-repo-and-workflow.md): the **approved** repo setup, CI, PR and commit conventions.
6. [docs/plan/05-implementation-plan.md](docs/plan/05-implementation-plan.md): the **approved** Stage A implementation plan: PRs by day, the daily routine, pausing and resuming.
7. [docs/runbooks/](docs/runbooks/): procedures for rare operations, such as moving to a private repo at the start of Stage C.
8. `private/`: business notes, reference materials (such as the UNRWA learning cards) and planning drafts. **Never commit it.**

**If a step document ever conflicts with a PLAN.md decision, the decision wins. Report the conflict to the owner instead of choosing silently.**

## Current status (2026-11-25)

**The Stage A plan was approved on 2026-11-23 (all six steps). No code has been written yet.**

| Step | Status |
|---|---|
| 1. Project idea and strategy (PLAN.md) | ✅ Revised (D-24 to D-48) |
| 2. Scope (Stage A) | ✅ Approved (v4) |
| 3. Architecture and data model | ✅ Approved (v3) |
| 4. Testing strategy | ✅ Approved (Stage A revision) |
| 5. Repo setup, CI, PR and commit conventions | ✅ Approved (Stage A revision) |
| 6. Implementation plan | ✅ Approved (Stage A) |

**Git:** the public GitHub repo `darsi` exists (2026-11-25), with `main` pushed, squash-only merges and the `main` ruleset on (repo and workflow, section 7; required status checks come after the CI PR). The Stage A revision is the first PR, `docs: replan for Stage A`, from `docs/replan-stage-a`.

**Next action:** CodeRabbit is installed but skips automatic reviews while the repo has fewer than 10 stars, so the owner asks for its review of the replanning PR by commenting `@coderabbitai review` (repo and workflow, section 10), resolves its comments, then merges. The rest of the setup in section 2 of `docs/plan/05-implementation-plan.md` (Vercel, Neon) is also needed before **Day 1**, which starts with PR 1.

## Working agreement

- **One step at a time.** Present a planning step, then wait for the owner's reply before moving on.
- **Discussion with the owner is in Arabic**, with technical terms in English where clearer. **Every file is written in English.** The product UI is Arabic.
- **Be direct.** If an idea, scope or timeline seems unrealistic, say so and explain the cost.
- When the owner asks to discuss questions one by one, present one question with options and a recommendation, then wait.
- **Record every decision** in the PLAN.md decisions log (the next number is **D-49**), with its state (APPROVED, PROPOSED or OPEN), and keep its planning status table current. Never rewrite an older decision: record its new status in "Current status of D-01 to D-23" or in a new decision. Step details go in `docs/plan/`. During implementation, technical decisions become ADRs in `docs/adr/`.
- **The owner must see every code change:** edit code with the editor's edit tools, never with shell scripts, so each change appears as a diff in VS Code. Files that tools write themselves (scaffolding, lockfiles, generated migrations) are reviewed in the Source Control view instead.
- **The plan's days are focused days of work, not consecutive calendar days;** the owner sometimes pauses. During implementation, every PR updates "Current status" as its last commit (what it completes and the next PR), so the status reaches `main` with the merge and work resumes cleanly after a pause.
- **The whole Stage A plan was approved on 2026-11-23.** Build in the order of `docs/plan/05-implementation-plan.md`, following its daily routine (section 3).

## Rules that must not be broken

- **`private/` is in `.gitignore` and must never be committed or pushed.** No real prices, bank or wallet account details, contacts or business plans in git (D-17, D-18). The public repo uses placeholder prices.
- **Commits use this repo's local git identity** (the owner's GitHub name and GitHub's no-reply email). Don't change it, and don't commit with another identity: the owner's real name and email stay out of the public repo (D-42).
- **License:** the code is public under **FSL-1.1-ALv2** (`LICENSE`, D-17). The content folder gets its own license file, **all rights reserved** (D-39).
- **All content is written from scratch.** No Ministry or UNRWA text, examples, exercises or images in the repo. UNRWA's cards are used only for objectives and their order, and are never sent to an AI (D-09, D-26).
- **Only the student opens the student's own space** (D-45). Teachers and guardians see progress only from their own dashboards; they get a one-time temporary PIN, the student chooses the real one, and the app asks for it on every opening and after 10 minutes without use. Never add a way for an adult to sign in as a student.
- **No real student data,** except family testing with the owner's own child under the D-40 rules: first name only; the data stays in Darsi's database and on the student's phone for offline practice (D-36, D-48), never in the repo, logs, screenshots or AI prompts; a private deployment; teacher sign-up restricted to the owner; complete deletion possible.
- **AI and students (D-30):** nothing that identifies a student is ever sent to an AI (D-08a); a person reviews every piece of stored AI-written content before students see it (D-08b); students don't interact with AI at runtime (D-08c). Anonymous, aggregated wrong-answer patterns may be sent while writing content only when they come from more than one student.
- **AI runs cost money.** In Stage A, AI is used only by the content-writing script (D-29). The owner chooses the model after a small trial; **ask before running any paid AI call.** Candidates: Claude Opus 5.5, Sonnet 5.5 and Haiku 5.5, called **through OpenRouter** by their OpenRouter model IDs (D-23).
- **Team practices:** one PR per feature, conventional commits, CI (lint, typecheck, tests) on every PR, review by CodeRabbit.
- **No AI `Co-Authored-By` trailer** in commits or squash-merge messages, as the owner decided. AI assistance is disclosed once, in the README's "How this was built" section ([repo and workflow, sections 8 and 14](docs/plan/04-repo-and-workflow.md)).

## Product essentials

Details are in PLAN.md (D-24 to D-48) and the Stage A scope document.

- **Stages (D-28, D-44):** A, the student prototype tested in the family; B, the supervised prototype for both teachers and guardians, on test accounts and synthetic data; C, a pilot beyond the family, which needs the Stage C prerequisites first. Other ideas are on the "Later" list.
- **First unit (D-27):** integers, from the UNRWA grade 7 cards, in this order: number line and opposites → comparing and ordering → absolute value → addition → subtraction.
- **Content (D-26, D-39):** a skill map with prerequisites, a short diagnostic, original reviewed lessons, item templates that generate questions, and hints and misconception messages; all stored as files in the repo.
- **Answers (D-33, D-41):** a whole number typed on Darsi's own keypad, or multiple choice (comparison and true/false included). Arabic-Indic digits on screen; negatives shown as "-٥".
- **Feedback and mastery (D-34, D-35):** misconception message and another try; two hint levels on request; worked solution after two wrong tries. A skill is mastered at 4 of the last 5 attempts correct on the first try without a hint, or when passed in the diagnostic with no wrong attempt since.
- **Offline (D-36, D-37):** an installable PWA; the unit is cached on the phone; answers are checked there and stored as immutable attempts in a per-student queue; sync goes to one Route Handler and the server checks every attempt again.
- **Sign-in (D-38, D-45):** the owner signs in with Clerk as the teacher (only the owner can sign up). A class is a group of children with subjects inside it (D-22). A student is a first name and a 4-digit PIN in one class, signing in with the class code. The teacher gets a one-time temporary PIN; the student chooses the real one at the first sign-in. An app lock asks for the PIN on every opening and after 10 minutes without use, offline too. The PIN's weakness is accepted until Stage C.
- **Not in the prototype (D-31, D-32):** payments, trials, subscriptions, the public demo, several subjects or grade cohorts, runtime AI.
- **The UI is Arabic, right-to-left only** (D-04).

## Stack (approved with architecture v3)

- **App:** Next.js 16 (App Router), React 19, TypeScript strict
- **UI:** Tailwind CSS v4 + shadcn/ui
- **Data and validation:** Prisma + PostgreSQL (Neon), Zod
- **Offline:** an installable PWA with a service worker and IndexedDB; sync through one Route Handler (D-36)
- **Login:** Clerk for the teacher (the owner); our own class code, name and PIN sessions for students
- **AI:** Claude models through OpenRouter (D-23), used only by the content-writing script in Stage A (D-29)
- **Testing, CI and hosting:** Vitest, Playwright, GitHub Actions, Vercel

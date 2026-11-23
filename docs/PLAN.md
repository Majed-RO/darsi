# Darsi (دَرْسي) — Project Plan

> Living document · last updated 2026-11-23
> This is the entry point for all planning. Details live in the step documents linked below.

## How the plans are organized

| Where | What it holds | Changes when |
|---|---|---|
| `docs/PLAN.md` (this file) | Background, stages, **decisions log**, planning status, open questions, facts to verify | Any decision is made or changed |
| `docs/plan/0N-*.md` | One detailed document per planning step | That step is revised |
| `docs/adr/` *(created in the first PR)* | Architecture Decision Records: one file per decision that shapes the code, with its context and trade-offs | A technical decision is made during implementation |
| `docs/runbooks/` | Step-by-step procedures for operations done rarely, such as moving the repo to private | A procedure is added or changes |
| `private/` | Business notes (costs, pricing, contacts), reference materials such as the UNRWA learning cards, and planning drafts. **Git-ignored from the first commit and never pushed** (D-17) | Business decisions change |
| `AGENTS.md` (repo root) | Instructions for AI coding sessions: where to start, current status, working agreement, rules. `CLAUDE.md` only imports it (D-42) | The planning status or a working rule changes |

Why this split: one big file would grow past a thousand lines and become hard to review. Separate step files can be reviewed and changed one at a time. This file stays short enough to read in two minutes, and technical decisions graduate into ADRs, which reviewers expect in a professional repo.

## Background

Since October 2023, students in Gaza have lost years of schooling. Schools reopened for in-person teaching on 19 September 2026, after three years, mostly in tents and temporary learning points, and the Ministry of Education runs a virtual track for students who can't attend ([Al Jazeera](https://www.aljazeera.com/features/2026/9/20/back-to-school-in-gaza-learning-in-tents-after-years-of-disruption), [Al-Monitor](https://www.al-monitor.com/originals/2026/09/gaza-students-return-person-schooling-first-time-three-years)). UNICEF reports that every school-age child has lost learning and that basic reading and numeracy are wearing away, although the loss hasn't been measured yet ([UNICEF, April 2026](https://www.unicef.org/sop/media/6361/file/Humanitarian%20Situation%20Update%20-%2030%20April%202026%20-%20Education.pdf.pdf)). Connections and electricity are unreliable, and family phones are often shared.

The Ministry publishes materials and quizzes through Moodle, a general-purpose system that is heavy for children and families; its official role in assessment is still to be verified (V-03). UNRWA schools teach with weekly self-learning cards, 16 per semester ([UNRWA, grade 7 math](https://keeplearning.unrwa.org/cat_gaza/gfo-06-mathematics-gfo-07/)).

**Darsi** is an Arabic, mobile-first learning and practice platform, meant to cover several subjects and grades. It starts with one math unit in grade 7 as its first test (D-25, D-26). A short diagnostic test finds what the student already knows; practice then follows the skills the student is missing, including skills from earlier grades, with feedback and hints that help the student think rather than hand out answers. It keeps working through connection drops, and teachers can follow progress and step in (Stage B). Darsi is supplemental: it gives no official grades and doesn't replace official assessment.

## Goals

1. **Product:** a learning platform for several subjects and grades that is educationally valuable and commercially viable for students in Palestine. It starts with math in grade 7, chosen for testing, not as a limit. Who pays is still open (D-32).
2. **Career:** a portfolio project that shows senior engineering practice: unit and E2E tests, CI on every PR, one PR per feature, code review, conventional commits, ADRs, and a professional README.

## Stages

Stages replace the earlier phases (D-28). Stages A and B are two steps of one product, not two products.

| Stage | Goal | Data | Status |
|---|---|---|---|
| **A — Student prototype** | One student can learn and practise one unit fairly independently: lessons, a diagnostic test, practice with feedback and hints, progress, and practice that survives connection drops | Family testing with the owner's son only (D-40) | Planning |
| **B — Teacher-supervised prototype** | Teacher role and access, classes, a progress dashboard, skill gaps, and teacher intervention | Test accounts and synthetic data | After Stage A |
| **C — Pilot beyond the family** | Real students and teachers outside the family | Real data: the Stage C prerequisites come first, and development moves to a private repo (D-17) | After Stage B |
| **Later** | Unscheduled ideas (see "Future features") | — | — |

## Planning status

The plan approved on 2026-11-22 is being revised for Stage A after the strategy review of 2026-11-23 (D-24 to D-43). Each document is rewritten and reviewed in turn.

| Step | Document | Status |
|---|---|---|
| 1. Project idea and strategy | This file | ✅ Revised (2026-11-23) |
| 2. Scope (Stage A) | [plan/01-mvp-scope.md](plan/01-mvp-scope.md) | ✅ Approved (v4) |
| 3. Architecture and data model | [plan/02-architecture.md](plan/02-architecture.md) | ✅ Approved (v3) |
| 4. Testing strategy | [plan/03-testing-strategy.md](plan/03-testing-strategy.md) | ✅ Approved (Stage A revision) |
| 5. Repo setup, CI, PR and commit conventions | [plan/04-repo-and-workflow.md](plan/04-repo-and-workflow.md) | 🔄 Draft, in review |
| 6. Implementation plan (PRs by day) | [plan/05-implementation-plan.md](plan/05-implementation-plan.md) | 🔄 To be rewritten for Stage A |

**Working agreement:** one step at a time, each reviewed before the next. Implementation starts when all six documents are approved again.

## Decisions log

Decisions keep their original wording. When a later decision changes one, the change is recorded in "Current status of D-01 to D-23" below the log, not by rewriting it. From D-24 on, each decision states whether it is **APPROVED**, **PROPOSED** (recommended, waiting for approval) or **OPEN** (not decided); outside facts still to be checked are listed as **VERIFY** items (D-42).

| # | Decision | Why |
|---|---|---|
| D-01 | Build **Darsi**, a learning and practice space for elementary schools | A real need in the region, and a stronger story than a generic SaaS demo |
| D-02 | Start with **grades 1–6**; demo content is **grade 3** | Text-only questions suit grades 3–6; grades 1–2 need images and audio (Phase 2) |
| D-03 | The customer is a **teacher** in Phase 1 and also a **school** from Phase 2 (the workspace); students are end users | B2B subscription model; see D-13 for schools |
| D-04 | **Arabic, right-to-left UI only** in the first stage | Local market first; supporting two languages doubles UI work |
| D-05 | **Two phases:** portfolio MVP on seed data, then a pilot with real students | Real children's data needs privacy, consent and support that don't fit in the MVP timeline |
| D-06 | **Manual payments**: bank transfer or e-wallet (Bank of Palestine, PalPay, JawwalPay), verified by the platform admin | Stripe isn't available locally without Stripe Atlas; manual verification fits a local business at pilot scale |
| D-07 | **14-day trial** starts automatically on sign-up, with no approval step (answers Q-01). Reconsider verifying teachers before the Phase 2 pilot | No waiting for teachers or reviewers; AI abuse is capped by the trial quota; Phase 1 has no real children to protect |
| D-08 | **AI generates drafts only.** A teacher reviews and publishes everything students see. Students never talk to the AI, and no student data is sent to it | Quality and safety for young children; predictable cost |
| D-09 | **Phase 1 needs no ministry permission:** Darsi links to ministry sources, and teachers add their own materials. All demo content in the public repo is written from scratch. Before a ministry importer or the paid pilot, get at least an informal written OK from the ministry (answers Q-06) | Emergency conditions make the ministry likely to welcome help, but a short written OK is cheap insurance once Darsi charges money; the public repo must not republish third-party content |
| D-10 | Students log in **without email**: class code + name + 4-digit PIN | Children aged 6–12 don't have email addresses |
| D-11 | **No chat or discussions, and no leaderboards** | Moderating children's messages is a safety burden; competition isn't a goal |
| D-12 | Plans live in `docs/PLAN.md` plus step documents; technical decisions become ADRs | See "How the plans are organized" |
| D-13 | **School workspaces with several teachers move to Phase 2.** Phase 1 workspaces have one teacher, but the data model separates workspaces from members from day one | Saves about a day in Phase 1; schools become an addition later, not a rewrite |
| D-14 | Subscriptions are sold per **semester** or **school year**, following a **school calendar** the admin maintains each year. No proration: buying mid-semester costs the full price and runs to that semester's end. Prices in **ILS**, placeholders in Phase 1 (answers Q-02) | Fewer payments to verify by hand; teachers are never cut off in the middle of a semester; matches how schools plan |
| D-15 | **AI limits are a shared allowance per subscription period**, not per month: trial 10, semester 300, school year 600 (placeholders in configuration). Failed generations don't count, unused ones don't carry over, and a global switch can turn AI off. Final numbers are set with prices after the architecture step measures real cost and quality (answers Q-04) | Teachers create content in bursts, mostly at the start of a semester; a monthly limit would block them exactly then |
| D-16 | **No temporary access while a payment waits for verification.** An expired workspace stays read-only until the admin approves; students are never affected. A pending-notices counter in the admin navigation shows waiting payments (answers Q-05) | The 14-day reminder already protects teachers who pay on time; temporary access would let fake notices buy free days |
| D-17 | **Phase 1 repo is public under the Functional Source License (FSL-1.1-ALv2).** Anyone can read and learn from the code, but nobody may run it as a competing product; each release becomes Apache 2.0 after 2 years. Bank details, real prices, contacts and business plans never go into git. At the start of Phase 2, development moves to a private copy of the repo and the public repo is archived at its Phase 1 state. The `LICENSE` file is added in the first PR | The code must be readable by reviewers, while the business stays protected; the real advantage is local relationships and operations, not the code |
| D-18 | **Working prices are set** for the Teacher plan (semester and year) and the Phase 2 School plan, based on a cost and break-even estimate. The numbers live only in `private/` and in deployment configuration; the public repo uses placeholders. Prices must be validated with teachers, private schools and sponsors before the pilot. **Constraint for the AI model choice:** a heavy user's AI cost must stay well below the yearly price | Low fixed costs mean low risk; the AI model is the largest variable cost |
| D-19 | **Each period's access starts where the previous period ends.** A semester also covers the break before it, and a school year also covers the summer before it; the first period in the calendar starts on its own start date. A purchase gives access from approval or that start, whichever is later, until the period's end. When a trial and a paid period overlap, the paid AI allowance counts from approval and replaces the trial's. The period picker hides periods that overlap one already paid for or waiting for verification, and shows when access would start (answers Q-07) | With access starting at approval for every period, buying next semester mid-semester also gave the rest of the current one (and next year, almost two school years). This keeps "pay early, no gap", lets teachers prepare during breaks, and gives no time away |
| D-20 | **The public demo uses "Try as teacher" and "Try as admin" buttons, plus a nightly reset.** The buttons sign in to fixed demo accounts with Clerk sign-in tokens; the demo accounts have no password. They work only while a demo setting is on, never in the pilot, and the demo admin is a separate account from the real admin. A daily Vercel Cron job restores the demo workspaces and the seeded calendar. Demo workspaces get a small AI allowance. Students sign in with the class code and PINs from the README. The demo runs on our own domain, because a Clerk production instance can't use `*.vercel.app` (answers Q-08) | Clerk's Device Trust (on by default for new apps) asks for an emailed code when a password sign-in comes from a new device, so shared demo passwords can't work. Shared accounts can be vandalized, and the reset limits any damage to one day. Costs a domain (about $10–15 a year) and about half a day of work |
| D-21 | **Two overall AI spending limits.** A daily generation cap across all workspaces (per day in Palestine time, set in configuration; failed generations don't count). When it's reached, AI buttons are disabled for everyone until the next day with a clear message, and everything else keeps working. A monthly spend limit on the Anthropic workspace is the hard backstop (moved to the OpenRouter key's credit limit by D-23). Demo starting values: 50 generations a day and $20 a month. The pilot sets the cap well above real use (answers Q-09) | Automatic trials (D-07) and the demo buttons (D-20) give anyone AI generations, and per-workspace allowances don't limit the total. The daily cap stops a burst, and the spend limit bounds the month even if the code is wrong |
| D-22 | **A class is a group of children, and subjects belong to it.** `Class` holds the roster, grade and join code; `Subject` (such as Math or Arabic) holds the teacher and the topics. Each child has one login per class, however many subjects or teachers it has. A child's logins in different workspaces (a school, a private tutor, a learning center) stay separate until parent accounts in Phase 2 (answers Q-10) | A class teacher in grades 1–4 usually teaches several subjects to the same children, and from grade 5 several subject teachers share one class. With one subject per class, children would get a new code and PIN per subject, and Phase 2 schools would need a data migration. Linking logins across workspaces needs a parent to own the link, with consent |
| D-23 | **AI calls go through OpenRouter**, with one API key. The candidates stay Claude Opus 5.5, Sonnet 5.5 and Haiku 5.5, called by their OpenRouter model IDs, with requests pinned to Anthropic as the serving provider. The D-21 backstop becomes a credit limit on the OpenRouter key. The evaluation PR confirms that structured outputs, effort and thinking-token counts work through OpenRouter. Adding non-Claude candidates is possible later and would be decided before the evaluation run | The owner already has an OpenRouter key, and it allows switching to other providers' models without a new account. Accepted costs: a fee of about 5.5% on credit purchases, one more service handling teachers' material text (named in the Phase 2 privacy policy), and Claude features that depend on OpenRouter's support. The `AiProvider` interface keeps a direct Anthropic provider possible |
| D-24 | **APPROVED · Limited hybrid learning model.** Students learn and practise on their own; teachers follow their progress and step in when needed; practice adapts to each student's mastery; progress follows the skills a student has mastered, not the grade label alone. The long-term vision serves both independent students and teacher-led classes, across several subjects and grades; math and grade 7 are only the starting point for testing (D-25, D-26). The MVP doesn't build all of it, but the data and content model names the subject and grade instead of assuming math or grade 7, without building other subjects yet (D-31) | Replaces the teacher-first idea (D-01, D-03) after comparing it with an independent, student-first plan. Teachers keep a supervising role without having to write all the content |
| D-25 | **APPROVED · First cohort: grade 7.** Chosen for a practical reason: the owner's son is in grade 7, so the first test is family-based validation with a convenience sample, not market research. It doesn't decide the commercial target and doesn't limit Darsi to grade 7 | Real usage from the first day at no recruiting cost. Every result states the limits of a sample of one |
| D-26 | **APPROVED · Math, one small unit, skills first.** One unit of the Palestinian curriculum, taught as a skill map with a short diagnostic test, remediation of missing prerequisite skills from earlier grades, and reviewed core content supplied by Darsi; teacher additions come later. Never assume that a grade-7 student has the earlier skills. The reference for Stage A is the **UNRWA self-learning cards** the son's school uses (16 weekly cards per semester): skills map to their objectives, and mapping to the Ministry's outcomes comes later (V-01). UNRWA's materials are "all rights reserved", so Darsi uses only their objectives and order, writes every text, example and exercise from scratch, and never sends the cards to an AI (extends D-09) | Students in Gaza have lost years of schooling (UNICEF), so prerequisite gaps are the norm. The content is the product, so it must be original and reviewed |
| D-27 | **APPROVED · The first unit: integers.** From semester 1, unit 1, in this order: the number line and opposites → comparing and ordering → absolute value → addition → subtraction. Multiplication and division come later. The number line appears as a static picture (SVG) in lessons and hints; no answer is given by dragging on a number line | Answers are whole numbers that code can check exactly, sign mistakes are well known, and the unit is small enough to write and review. The son studied it about two months earlier, so Stage A tests review and remediation |
| D-28 | **APPROVED · Stages replace phases.** **Stage A:** the student prototype, tested in the family. **Stage B:** the teacher-supervised prototype (teacher role and access, classes, progress dashboard, skill gaps, teacher intervention), tested with test accounts and synthetic data. **Stage C:** a pilot with students outside the family. Stage C inherits every "start of Phase 2" commitment: moving to a private repo (D-17), privacy notice, guardian consent, deleting and exporting data, and a sign-in security review. Everything else once marked Phase 2 or 3 moves to an unscheduled **Later** list. Reading older text: "Phase 1" means Stages A and B; "Phase 2" means Stage C for its commitments (the private repo, protecting real data) and "Later" for features, as "Current status of D-01 to D-23" shows | Stages A and B are two steps of one product. Tying the old commitments to Stage C keeps every one of them |
| D-29 | **APPROVED · AI tutor: in the vision, controlled in the prototype.** The vision keeps an AI tutor that helps students think through curriculum-grounded explanations, progressive hints, guided problem solving and feedback on misconceptions, and never hands out final answers. There is no unrestricted conversational AI in the prototype. **In Stage A, AI is used only to write content:** it drafts hints, misconception messages and worked solutions, the owner reviews them, and they are stored as static content. Students make no AI calls. Wrong answers that match no known misconception are logged; a high share of them is the evidence for evaluating a runtime "explain my mistake" feature, with accuracy, cost and safety checked first | Sign mistakes with integers are predictable, so reviewed static hints cover most cases at almost no cost, work offline and carry no AI risk for children. The content's shape (hint levels, misconception IDs, worked solutions) stays ready for a runtime tutor |
| D-30 | **APPROVED · D-08 becomes three rules.** **D-08a (permanent):** nothing that identifies a student is ever sent to an AI: no name, ID, class, or one student's sequence of answers. **D-08b (permanent):** a person reviews every piece of stored AI-written content before students see it. **D-08c (until evidence and an evaluation change it, D-29):** students don't interact with AI while using Darsi. Aggregated, anonymous wrong-answer patterns (for example "template X: −8 given 12 times") may be sent while writing content, **only when they come from more than one student**, so never during the family test | D-08 mixed a permanent privacy rule with a temporary product rule. Splitting them keeps privacy strict and leaves room for the AI tutor in the vision |
| D-31 | **APPROVED · Scope discipline.** Not in the prototype: several schools, advanced subscription management, several subjects, several grade cohorts, advanced AI features, complex teacher workflows. Keep the design extensible without building ahead. Respect student privacy and content rights, and never present Darsi as a replacement for official assessment | One developer; the goal is a prototype that can be tested, not a virtual school |
| D-32 | **OPEN · Business model.** B2C (students or parents), B2B (teachers, learning centres, schools) and B2B2C (through an institution or a sponsor) all stay possible; none is assumed. Pricing validation, payment automation and subscription management are deferred, not dropped | Nobody has validated willingness to pay yet, and the family prototype needs none of it |
| D-33 | **APPROVED · Two answer types.** (1) A whole number typed on Darsi's own on-screen keypad (٠–٩, minus, delete), checked by exact match after normalizing digits and minus signs. (2) Multiple choice, with comparison (<, >, =) and true/false as fixed-button variants. No drag-and-drop ordering: ordering skills are tested with questions such as "which number is smallest?" or "which order is right?" | The number pad of many phone keyboards has no minus key, and the keypad guarantees Arabic-Indic digits. Every extra answer type costs a screen, a checker and tests |
| D-34 | **APPROVED · Mastery rule.** A skill is mastered when at least 4 of the student's last 5 attempts on it are correct. An attempt counts as correct only if it was right on the first try without a hint; later tries and hinted answers are recorded but not counted. A skill passed in the diagnostic test (2 of 2 correct) counts as mastered until the first wrong attempt after it; from then on the last-5 rule applies. States: not started, practising, mastered; mastery can drop back. The numbers live in configuration and are tested at their boundaries | Measures what the student knows before any help, tolerates one slip, and skips what the diagnostic already showed. Because attempts are never changed (D-37), mastery can be recomputed if the rule changes |
| D-35 | **APPROVED · Feedback flow.** A wrong answer shows the matching misconception message and allows another try. Two levels of hints are available on request and never reveal the final answer. After two wrong tries the worked solution is shown, and the attempt counts as wrong | Helps the student think instead of handing out answers (D-29) |
| D-36 | **APPROVED · Limited offline (L1) and sync.** An installable PWA caches the app shell and the whole unit on the first visit. Checking answers and mastery run on the phone; attempts are saved in IndexedDB and synced without duplicates; the sync state is always visible ("3 answers waiting", "synced"); persistent storage is requested. Sync starts from the page (on opening, when the connection returns, after each session), without Background Sync. It goes to **one stable Route Handler** (`/api/sync`), the only exception to "mutations are Server Actions", because a cached copy of the app keeps old Server Action IDs after a new deploy. The server checks every attempt again from its template, numbers and content version; if its result differs from the phone's, the server's result is kept and the attempt is flagged for the owner. Not included: opening the app for the first time offline, signing in offline, updating content offline. Full offline support (L2) only if real use shows the need | Losing a connection must never lose a student's answers or progress. Static content and checking on the phone make L1 cheap, and immutable attempts (D-37) keep L2 possible |
| D-37 | **APPROVED · Attempts are immutable records.** Each answer is one record that is never changed, with an ID created on the phone, so a repeated sync stores it once. It holds the student, the skill, the item template and its generated numbers, the answer, the result, the hints used, the content version, the phone's time and the time the server received it. Progress and mastery are computed from these records. The only exception: deleting all of a student's data on request | Sync without duplicates, mastery that can be recomputed, and the evidence D-29 needs |
| D-38 | **APPROVED · Student sign-in from Stage A.** D-10 is brought forward. The owner signs in with Clerk as the teacher (sign-up restricted to the owner's email), creates one class with a Math subject linked to the core unit, and adds his son, who signs in with the class code, his first name and a 4-digit PIN. Stage A has only the teacher screens this needs; the dashboard is Stage B. "Switch student" stays: each student's unsynced answers stay on the phone in their own queue and are sent when that student signs in again with a connection (signing in needs one). The sign-in screen shows how many answers each student has waiting, and the server rejects attempts that don't belong to the signed-in student. **The 4-digit PIN and its lockout stay:** the risk is accepted for Stages A and B (a secret class code, one child, synthetic data), and guessing PINs across a class and locking students out on purpose are reviewed before Stage C | Keeps the approved data model (D-13, D-22), so Stage B needs no remodelling. Signing in again on any phone restores progress from the server |
| D-39 | **APPROVED · Content lives in the repo.** Lesson texts, hints, misconception messages and worked solutions are readable files; item templates are small TypeScript functions that generate questions. CI validates them with Zod and with property tests (thousands of generated items per template, each answer checked). The content folder has its own license file: **all rights reserved**, because the FSL covers software, not teaching content. Revisit when the repo moves private (D-17) | Content changes are reviewed as diffs in PRs, like code. One unit has little commercial value to protect, and an open license could be granted later but never withdrawn |
| D-40 | **APPROVED · Family testing exception.** No real student data, except family testing with the owner's own child, under these rules: first name only (no birth date, school or phone number); the data never leaves the database (not in the repo, logs, README screenshots or any AI prompt); a private deployment, with the public demo (D-20) deferred; teacher sign-up restricted to the owner; a student's data can be deleted completely. A child outside the family needs a privacy notice, written guardian consent and deletion first (Stage C) | The prototype must be tested with a real learner. Writing the exception down makes every tool and reviewer apply the same limits |
| D-41 | **APPROVED · Digits.** Numbers are shown in Arabic-Indic digits (١٢٣), as in the UNRWA cards. Typed input accepts Arabic-Indic, Persian and Western digits. A negative number is shown as "-٥", with its direction isolated explicitly so a right-to-left page never shows it as "٥-" | Matches what the student sees at school; minus signs in right-to-left pages are a common source of display bugs |
| D-42 | **APPROVED · Repository housekeeping.** `AGENTS.md` is the single file of instructions for AI tools, and `CLAUDE.md` only imports it. The ChatGPT planning pack lives in `private/`. Commits use the owner's GitHub name (Majed-RO) with GitHub's no-reply email, and the license names Majed-RO, so the owner's real name stays out of the public repo. Decisions carry a state (APPROVED, PROPOSED, OPEN) and outside facts are VERIFY items | One source of instructions can't drift into two versions. The planning pack contains business details. The owner prefers not to publish his name |
| D-43 | **APPROVED · Test device.** The son uses an Android phone with Chrome, and the connection sometimes drops. E2E tests stay on Chromium with a phone profile, and the family test exercises L1 in real conditions, alongside deliberate offline tests | No WebKit testing is needed for Stage A |

### Current status of D-01 to D-23

| # | Status |
|---|---|
| D-01 | Superseded by D-24 and D-26 |
| D-02 | Superseded by D-25 |
| D-03 | Superseded: who pays is open (D-32) |
| D-04 | Active |
| D-05 | Superseded by D-28 (stages) and D-40 (family testing) |
| D-06 | Deferred until the business model is decided (D-32) |
| D-07 | Deferred (D-32); Stage A has no trial |
| D-08 | Amended by D-30 (D-08a, D-08b, D-08c) |
| D-09 | Active; extended by D-26 for UNRWA materials |
| D-10 | Active; brought forward to Stage A by D-38 |
| D-11 | Active |
| D-12 | Active; instructions file amended by D-42 |
| D-13 | Active for the data model; several teachers per school moves to "Later" (D-28) |
| D-14 | Deferred (D-32) |
| D-15 | Deferred: no AI generation inside the app in Stage A (D-29) |
| D-16 | Deferred (D-32) |
| D-17 | Active; "the start of Phase 2" now means the start of Stage C (D-28) |
| D-18 | Deferred: the working prices are not validated (D-32) |
| D-19 | Deferred (D-32) |
| D-20 | Deferred (D-40) |
| D-21 | The credit limit on the OpenRouter key stays, for writing content; the daily generation cap is deferred (D-29) |
| D-22 | Active; Stage A has one class with one Math subject |
| D-23 | Active, for writing content only (D-29) |

## Future features

### Stage C prerequisites

Before any student from outside the family signs in:

- A privacy notice and written guardian consent
- Deleting and exporting a student's data (deleting already exists from Stage A, D-40)
- A sign-in security review: guessing PINs across a class, and locking students out on purpose (D-38)
- A qualified reviewer for the content (Q-12)
- Moving development to a private repo and archiving the public one (D-17). Step-by-step: [runbooks/phase-2-move-to-private-repo.md](runbooks/phase-2-move-to-private-repo.md)
- Before the first paying customer: terms of service and a refund policy (D-32)

### School workspaces with several teachers (Later)

A school subscribes once and gives access to many teachers.

- **School workspace:** one owner (the school's administrator) and any number of teachers.
- **Owner:** manages the subscription, invites and removes teachers, and sees all classes and results in the school.
- **Teacher:** manages only the subjects they teach (their topics, materials and questions), sees the students of those classes, and uses the AI assistant. Several teachers can share one class, each with their own subject, and every child keeps one login (D-22).
- **Invites:** single-use links that expire after 7 days, shared through WhatsApp or any other channel.
- **Removing a teacher:** their subjects move to the owner, so no student data is lost.
- **Joining a school with existing classes:** a teacher who already has their own workspace can move their classes into the school's workspace, with students, content and practice history. Students keep their class code and PINs, so nothing changes for them. The old workspace keeps its billing and AI usage records as history; their links to moved materials are cleared, so no record points across workspaces. Decide then whether the old workspace is closed or kept.
- **School plan:** its own price, a teacher limit (Q-03) and a larger AI quota, if the business model includes schools (D-32).
- **Ready now:** the data model already separates workspaces from their members (D-13).

### Later

Unscheduled. Each item is decided when its time comes:

- Multiplication and division of integers, then more units and grades
- Teacher additions to the core content, and AI help for teachers to draft them
- A runtime AI tutor, after an evaluation of accuracy, cost and safety (D-29)
- Full offline support (L2): several downloaded units, signing in offline (D-36)
- Images and audio in content (needed for grades 1–2)
- Parent accounts, including linking a child's logins across workspaces (a school, a private tutor, a learning center) (D-22)
- Payments, once the business model is decided (D-32): manual payment notices (D-06), receipt uploads, invoices, alerts when a payment notice arrives (SMS as an option alongside email, Telegram or WhatsApp), extra AI packs
- The public demo with "Try as teacher" and "Try as admin" buttons and a nightly reset (D-20)
- An importer from the Ministry's site or Moodle, after a written OK (D-09)
- An English UI

## Open questions

| # | Question | Default until answered |
|---|---|---|
| Q-01 | ~~Does the trial start automatically, or only after you approve the account?~~ | ✅ Answered: automatic (D-07) |
| Q-02 | ~~Prices, currency and billing periods~~ | ✅ Answered: semester + school year, calendar-based, ILS, placeholder prices (D-14). Real prices before the pilot |
| Q-03 | How many teachers does a School plan include? | Later, with school workspaces (D-28, D-32) |
| Q-04 | ~~AI generation limits~~ | ✅ Answered: shared allowance per period, trial 10 / semester 300 / year 600 as placeholders (D-15) |
| Q-05 | ~~Should a workspace keep editing access while its payment notice waits for verification?~~ | ✅ Answered: no, read-only until approved (D-16) |
| Q-06 | ~~Do you have, or will you request, written permission from the ministry to reuse its content?~~ | ✅ Answered: not needed for Phase 1; informal written OK before the importer or paid pilot (D-09) |
| Q-07 | ~~When does access from a purchase start?~~ | ✅ Answered: where the previous period ends, or at approval if later (D-19) |
| Q-08 | ~~How do reviewers sign in to the public demo?~~ | ✅ Answered: demo buttons with Clerk sign-in tokens, nightly reset, own domain (D-20) |
| Q-09 | ~~Is there an overall cap on AI spending across all workspaces?~~ | ✅ Answered: daily generation cap plus a monthly spend limit on the Anthropic workspace (D-21) |
| Q-10 | ~~How are classes and subjects modeled, given that children often have several subjects and teachers?~~ | ✅ Answered: a class is a group of children with subjects inside it (D-22) |
| Q-11 | What are the Stage A success criteria, and their numbers? | Proposed in the Stage A scope; set before the family test starts |
| Q-12 | Who reviews the math content for accuracy and teaching quality? | The owner reviews in Stage A; a qualified reviewer is needed before Stage C |
| Q-13 | Do other children join the Stage A test? | No. A child outside the family waits for the Stage C prerequisites (D-40) |
| Q-14 | Who pays, and how? | Open (D-32); nothing commercial is built in Stages A and B |
| Q-15 | Which AI model writes the content drafts? | A small trial on a few skills, after the owner agrees to its cost (D-23, D-29) |

## Facts to verify

| # | Fact | Needed before |
|---|---|---|
| V-01 | How the UNRWA cards map to the Ministry's curriculum outcomes | Mapping skills beyond the UNRWA cards |
| V-02 | UNRWA's website says only materials published on its own platform may be used in its schools. Does that stop UNRWA teachers from using Darsi in class? | Stage B or C with UNRWA teachers |
| V-03 | Moodle's official role in assessment, including grade 7 | Any public claim; Darsi stays formative only |
| V-04 | Whether personal bank and wallet accounts may receive business payments, and their fees | Taking any money (D-32) |
| V-05 | Privacy and child-protection duties for a pilot with other children | Stage C |

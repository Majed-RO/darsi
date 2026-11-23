# Darsi (دَرْسي) — Project Plan

> Living document · last updated 2026-11-22
> This is the entry point for all planning. Details live in the step documents linked below.

## How the plans are organized

| Where | What it holds | Changes when |
|---|---|---|
| `docs/PLAN.md` (this file) | Background, phases, **decisions log**, planning status, open questions | Any decision is made or changed |
| `docs/plan/0N-*.md` | One detailed document per planning step | That step is revised |
| `docs/adr/` *(created in the first PR)* | Architecture Decision Records: one file per decision that shapes the code, with its context and trade-offs | A technical decision is made during implementation |
| `docs/runbooks/` | Step-by-step procedures for operations done rarely, such as moving the repo to private | A procedure is added or changes |
| `private/` | Business notes: costs, pricing, contacts. **Git-ignored from the first commit and never pushed** (D-17) | Business decisions change |
| `CLAUDE.md` (repo root) | Instructions for AI coding sessions: where to start, current status, working agreement, rules | The planning status or a working rule changes |

Why this split: one big file would grow past a thousand lines and become hard to review. Separate step files can be reviewed and changed one at a time. This file stays short enough to read in two minutes, and technical decisions graduate into ADRs, which reviewers expect in a professional repo.

## Background

Many schools in Palestine are out of service, so students and parents depend on online learning. The Ministry of Education publishes materials, quizzes and discussions through Moodle. Moodle is a general-purpose system that is heavy and complex for elementary-age children and their families.

**Darsi** is a simple, Arabic, mobile-first learning space for elementary schools (grades 1–6). Teachers publish materials and practice questions, and students practise and track their progress. Teachers get **AI help to draft summaries and quizzes**, which is Darsi's edge over a plain Moodle setup.

## Goals

1. **Product:** a tool that real teachers and schools in Palestine pay for, starting with elementary grades.
2. **Career:** a portfolio project that shows senior engineering practice: unit and E2E tests, CI on every PR, one PR per feature, code review, conventional commits, ADRs, and a professional README.

## Phases

| Phase | Goal | Data | Status |
|---|---|---|---|
| **1 — MVP** | Deployed, demoable product with full engineering practices | Seed data only, no real students | Plan approved (2026-11-22); building next |
| **2 — Pilot** | Teachers using it with real students; add school workspaces with several teachers | Real data, so privacy and consent must be ready | After Phase 1 |
| **3 — Growth** | More grades, images and audio, offline mode, more customers | — | Later |

## Planning status

| Step | Document | Status |
|---|---|---|
| 1. Project idea | This file | ✅ Decided |
| 2. MVP scope (in / deferred) | [plan/01-mvp-scope.md](plan/01-mvp-scope.md) | ✅ Approved (v3) |
| 3. Architecture and data model | [plan/02-architecture.md](plan/02-architecture.md) | ✅ Approved (v2), with D-19 to D-23 |
| 4. Testing strategy | [plan/03-testing-strategy.md](plan/03-testing-strategy.md) | ✅ Approved |
| 5. Repo setup, CI, PR and commit conventions | [plan/04-repo-and-workflow.md](plan/04-repo-and-workflow.md) | ✅ Approved |
| 6. Implementation plan (PRs by day) | [plan/05-implementation-plan.md](plan/05-implementation-plan.md) | ✅ Approved |

**Working agreement:** one step at a time, each reviewed before the next. The full plan was approved on 2026-11-22, so implementation can start.

## Decisions log

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

## Future features

### School workspaces with several teachers (Phase 2)

A school subscribes once and gives access to many teachers.

- **School workspace:** one owner (the school's administrator) and any number of teachers.
- **Owner:** manages the subscription, invites and removes teachers, and sees all classes and results in the school.
- **Teacher:** manages only the subjects they teach (their topics, materials and questions), sees the students of those classes, and uses the AI assistant. Several teachers can share one class, each with their own subject, and every child keeps one login (D-22).
- **Invites:** single-use links that expire after 7 days, shared through WhatsApp or any other channel.
- **Removing a teacher:** their subjects move to the owner, so no student data is lost.
- **Joining a school with existing classes:** a teacher who already has their own workspace can move their classes into the school's workspace, with students, content and practice history. Students keep their class code and PINs, so nothing changes for them. The old workspace keeps its billing and AI usage records as history; their links to moved materials are cleared, so no record points across workspaces. Decide in Phase 2 whether the old workspace is then closed or kept.
- **School plan:** its own price, a teacher limit (Q-03) and a larger AI quota.
- **Ready from Phase 1:** the data model already separates workspaces from their members (D-13).

### Other Phase 2+ features

The full list with reasons is in [plan/01-mvp-scope.md → Deferred](plan/01-mvp-scope.md#deferred). The most important ones:

- Images and audio in questions and materials, needed for grades 1–2
- **Pilot blockers:** privacy policy and parental consent; deleting classes, students and accounts
- Terms of service and refund policy, before the first paying customer
- Receipt uploads on payment notices, and email notifications
- **Alert to you when a payment notice arrives**, with **SMS as an option** alongside email, Telegram or WhatsApp. The SMS provider (a local bulk-SMS service or an international one) is chosen in Phase 2. The same channel can later notify teachers when a payment is approved or access is about to end
- Extra AI generation packs, bought through the manual payment flow
- A child in several workspaces (a school, a private tutor, a learning center) has a separate login in each. Linking them is part of parent accounts; a lighter step is letting a device remember several students, so switching doesn't mean retyping codes (D-22)
- Offline mode
- Before the pilot: decide whether teachers must be verified before their trial starts (D-07)
- **At the start of Phase 2:** copy the repo, with its full history, into a private repo, and archive the public one at its Phase 1 state (D-17). Step-by-step: [runbooks/phase-2-move-to-private-repo.md](runbooks/phase-2-move-to-private-repo.md)

## Open questions

| # | Question | Default until answered |
|---|---|---|
| Q-01 | ~~Does the trial start automatically, or only after you approve the account?~~ | ✅ Answered: automatic (D-07) |
| Q-02 | ~~Prices, currency and billing periods~~ | ✅ Answered: semester + school year, calendar-based, ILS, placeholder prices (D-14). Real prices before the pilot |
| Q-03 | How many teachers does a School plan include? | Moved to Phase 2 with school workspaces |
| Q-04 | ~~AI generation limits~~ | ✅ Answered: shared allowance per period, trial 10 / semester 300 / year 600 as placeholders (D-15) |
| Q-05 | ~~Should a workspace keep editing access while its payment notice waits for verification?~~ | ✅ Answered: no, read-only until approved (D-16) |
| Q-06 | ~~Do you have, or will you request, written permission from the ministry to reuse its content?~~ | ✅ Answered: not needed for Phase 1; informal written OK before the importer or paid pilot (D-09) |
| Q-07 | ~~When does access from a purchase start?~~ | ✅ Answered: where the previous period ends, or at approval if later (D-19) |
| Q-08 | ~~How do reviewers sign in to the public demo?~~ | ✅ Answered: demo buttons with Clerk sign-in tokens, nightly reset, own domain (D-20) |
| Q-09 | ~~Is there an overall cap on AI spending across all workspaces?~~ | ✅ Answered: daily generation cap plus a monthly spend limit on the Anthropic workspace (D-21) |
| Q-10 | ~~How are classes and subjects modeled, given that children often have several subjects and teachers?~~ | ✅ Answered: a class is a group of children with subjects inside it (D-22) |

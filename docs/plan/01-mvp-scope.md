# Darsi (دَرْسي) — Phase 1 MVP Scope

> Status: **✅ Approved (v3)** · 2026-11-19 · amended 2026-11-22 by D-19 (where purchased access starts), D-20 (demo sign-in), D-21 (overall AI limits), D-22 (classes and subjects) and D-23 (AI through OpenRouter)
> Background, decisions, open questions and future features: [../PLAN.md](../PLAN.md)

## Product in one sentence

Darsi is an Arabic, mobile-first learning space for elementary schools (grades 1–6). Teachers publish materials and practice questions, with AI help to draft summaries and quizzes, and students practise and track their progress.

## Roles

| Role | Who | Can do |
|---|---|---|
| **Platform admin** | You, the operator | Verify payments; activate or reject subscriptions |
| **Teacher** | The subscriber, who owns a workspace | Manage the subscription, classes, students, materials and questions; use the AI assistant |
| **Student** | Ages 6–12, the end user | Log in **without email**, read materials, practise, see their own progress |

In Phase 1, every workspace belongs to **one teacher**. **School workspaces with several teachers are a Phase 2 feature** (see [PLAN.md → Future features](../PLAN.md#future-features)). The data model keeps a separate workspace and membership from day one, so schools can be added later without a rewrite.

## Phase 1 goal

A deployed, demoable MVP running on **seed data** that proves the engineering practices: tests, CI, one PR per feature, and ADRs.
**No real student data in Phase 1.**

---

## In scope

### 1. Teacher account and workspace
- Email-based sign-up and sign-in for teachers.
- The first sign-in creates the teacher's workspace, named after the teacher or their school.

### 2. Trial, subscription and manual payment
- Every new workspace starts a **14-day trial** automatically, with no approval step (PLAN.md D-07).
- One plan in Phase 1: the **Teacher plan**, sold for a **semester** or a **school year** (PLAN.md D-14). Prices are in **ILS** and come from configuration. Phase 1 uses placeholder prices.
- **School calendar:** the platform admin defines each school year and its semesters, with start and end dates, and updates them every year.
- **Choosing a period:** the teacher picks from periods that haven't ended yet: the current or next semester, or the current or next school year. The page shows each period's dates, its price, and the date access would start. Periods that overlap one already paid for, or waiting for verification, aren't offered.
- **No proration:** the price is the same whenever the teacher buys. Buying in the middle of a semester gives access until **that semester's end**. The days already passed are not refunded or discounted.
- **Where access starts (PLAN.md D-19):** each period's access starts where the previous period ends. A semester also covers the break before it, and a school year also covers the summer before it. The first period in the calendar starts on its own start date.
- **Payment flow:**
  1. The teacher opens the Subscription page, picks a period, and sees the amount plus transfer instructions for **Bank of Palestine, PalPay and JawwalPay**. Your account details come from configuration.
  2. The teacher transfers the money outside Darsi.
  3. The teacher submits a **payment notice**: method, amount, transfer reference number, transfer date and payer name.
  4. The platform admin checks it against the bank or wallet statement, then **approves** it or **rejects it with a reason** the teacher can see.
  5. Approval gives access from that moment, or from the period's start (as defined above) if that's later, until the **end of the paid period**. Paying for the next period while the current one is still active continues access with no gap. Buying next semester in the middle of a semester gives access from the end of the current semester, not from approval.
- **Subscriptions follow the calendar:** a subscription points to its semester or school year rather than copying the dates. If the admin changes a period's dates (for example, the ministry extends a semester), access for everyone who paid for that period changes with it.
- **Breaks between semesters** belong to the next semester. A teacher who has paid for it, or for the school year, has access during the break, and paying during the break gives access right away, so teachers can prepare before term starts. Otherwise the teacher is read-only during the break, and students keep full access.
- Notice states: **Pending → Approved or Rejected**. A workspace can have only one pending notice at a time.
- Every decision records who made it and when.
- The teacher sees the current status, days left and notice history. A reminder banner appears **14 days before** access ends, leaving time for the transfer and its verification.
- **Access rules**, kept in one unit-tested module:
  - Trial or paid period active: full access.
  - Expired: the teacher can view everything but cannot create, edit or use the AI. **Students keep full access.**
  - Expired with a payment notice waiting: still read-only until the admin approves it (PLAN.md D-16). The teacher sees "Payment received, usually verified within 24 hours."
- Darsi never handles money and does not connect to any bank API.

### 3. Platform admin
- A list of pending payment notices with approve and reject actions, oldest first.
- A **pending notices counter** in the admin navigation, so waiting payments are visible at a glance. An SMS or other alert comes in Phase 2.
- A list of workspaces showing access status and end date.
- A **school calendar** page to create and edit school years and their semesters. Semesters in a year must be in order and must not overlap.
- Only accounts listed in server configuration can open these pages.

### 4. Classes
- A class is **a group of children**, such as "3A" (PLAN.md D-22). Create and edit a class with a name and a grade (1–6).
- Add one or more **subjects** to a class, such as Math and Arabic. Each subject has its own topics. A class teacher who teaches several subjects to the same children adds the children only once.
- Each class gets a short **join code** from an alphabet without easily confused characters (no `0/O`, `1/I/l`).

### 5. Students
- The teacher adds students to a class by **first name**. Names must be unique within a class, so add a last initial if two students share a name. Each child has one PIN per class, however many subjects the class has.
- The system generates a **4-digit PIN** for each student, and the teacher can reset it.
- A **printable roster** shows the class code plus each student's name and PIN, for sharing with parents.

### 6. Student login
1. Enter the class code.
2. Pick your name from the class list.
3. Enter your PIN.
- After **5 failed PIN attempts**, that student is locked out for **15 minutes**.
- The session is long-lived because devices are shared family phones. A **"Not you? Switch student"** button is always visible.
- Known trade-off: anyone with the class code can see the class's first names. This is accepted for first names only and will be recorded in an ADR.

### 7. Topics and materials
- Each subject's content is organized into ordered **topics** (units).
- A **material** has a title, a plain-text body, an optional link to the original source (such as the ministry page) and an optional **summary**.
- Teachers create, edit and delete materials. Students read them.

### 8. Questions
- Multiple choice with **one correct answer** and 2–4 options. True/false is the 2-option case.
- Each question belongs to a topic and has a **source** (written by the teacher, or drafted by AI) and a **status** (Draft or Published).
- Questions the teacher writes are published directly. **Only published questions appear in practice.**

### 9. AI assistant (teachers only)
- **Generate summary:** from a material's text, the AI writes a short, child-friendly Arabic summary for the class's grade. It is saved as a **draft** that the teacher edits, then publishes.
- **Generate questions:** from a material's text, the AI drafts **5 multiple-choice questions**. The teacher reviews each one: edit, publish or discard.
- **Grounded:** the AI works only from the material text the teacher provided, and the prompt includes the grade level.
- **Validated output:** every AI response must match a strict schema (2–4 options, exactly one correct answer, length limits). Invalid items are discarded, and the teacher is told how many.
- **Safety:** students never interact with the AI, and **no student data is ever sent to it**. Only the teacher's material text is sent.
- **Quota:** a shared allowance per subscription period (PLAN.md D-15): **10** for the trial, **300** per semester and **600** per school year. These are placeholders kept in configuration.
  - When a trial and a paid period overlap, the paid allowance starts at approval and replaces the trial's (D-19).
  - One summary or one question set counts as one generation, and generating again counts again.
  - Failed generations don't count, and unused generations don't carry over to the next period.
  - The teacher always sees how many are left. At zero, the AI buttons are disabled with a clear message, and everything else keeps working.
- **Cost safeguards:** a maximum material length sent to the AI (about 6,000 characters), and a global switch that turns all AI features off immediately.
  - **Overall limits (D-21):** a daily generation cap across all workspaces, plus a credit limit on the OpenRouter API key (D-23). When the daily cap is reached, the AI buttons are disabled for everyone until the next day with a clear message, and everything else keeps working.
- **Logging:** every generation is logged with workspace, user, type, result, tokens used and estimated cost.
- **Provider:** Claude models accessed through **OpenRouter** (PLAN.md D-23), with structured outputs. The model choice and cost estimate are settled by the evaluation run (architecture section 7) and recorded in an ADR.
- Tests and CI use a **fake AI provider**. No real AI calls run in CI.

### 10. Practice
- The student picks a subject (skipped when the class has only one), then a topic, and gets a **round of up to 5 questions**.
- Questions are picked in this order: never answered first, then those answered wrong most recently, then the rest (oldest attempt first). Answer options are shuffled.
- One question per screen, with **immediate feedback** that shows the correct answer, and a short summary at the end of the round.
- Every answer is stored as an **Attempt**.

### 11. Progress
- **Mastery** per student per topic = the share of the topic's published questions whose most recent attempt is correct.
- Students see mastery as **0–3 stars**. The thresholds live in one module and are unit-tested.
- The teacher's **class results** page shows a grid of students × topics, one subject at a time. Topics where the class average is below the threshold are flagged **"needs review"**.

### 12. Cross-cutting
- **Arabic UI, right-to-left, mobile-first.** It must stay usable on low-end Android phones and slow connections, with no heavy client-side libraries.
- **Tenant isolation:** a teacher sees only their own workspace, and a student only their own class. Both are enforced on the server and covered by tests.
- **Seed data:** two teacher workspaces, one with an active subscription and one in trial with a pending payment notice for the admin demo. Each has a grade 3 class with a Math subject of 3 topics, about 20 questions, and 5 students with practice history. Two workspaces also let reviewers see tenant isolation working. **All demo content is written from scratch**, so the public repo contains no ministry materials (D-09).
- **Deployed** to a public URL on our own domain (D-20). Reviewers sign in as teacher or admin with "Try as teacher" and "Try as admin" buttons, and as students with the class code and PINs in the README. The demo data resets every night.

---

## Deferred

| Item | Why deferred | When |
|---|---|---|
| **School workspaces with several teachers** (details in [PLAN.md → Future features](../PLAN.md#future-features)) | Adds roles, invites and a second plan; one-teacher workspaces prove the product first | Phase 2 |
| Receipt image upload on payment notices | Needs file storage; the reference number is enough to match statements | Phase 2 |
| Invoices and receipts (PDF) | Not needed to verify payments | Phase 2 |
| Online payments or bank and wallet API integration | No suitable provider yet; manual verification works at pilot scale | When volume justifies it |
| Email notifications (payment approved, renewal reminders) | In-app banners are enough for now | Phase 2 |
| Alert to the platform admin when a payment notice arrives: **SMS as an option**, or email, Telegram or WhatsApp | Needs an external provider, and SMS costs per message | Phase 2 |
| **Images and audio** in questions and materials | Needs file storage and upload flows. **Required for grades 1–2**, who can't read reliably yet | Phase 2, top priority |
| PDF and file uploads; AI generation from PDFs or scanned pages | Same storage work, plus text extraction | Phase 2 |
| AI for students (tutor or chat) | Needs safeguards for children; teachers review everything in Phase 1 | Not before Phase 3 |
| Extra AI generation packs, bought through the same manual payment flow | An upsell; not needed to prove the product | Phase 2 |
| Importing from the ministry site or Moodle (GIFT format) | Get at least an informal written OK from the ministry first (D-09) | Phase 2 or later |
| Offline mode (PWA) | Large effort; an ADR will describe the intended approach | Phase 2–3 |
| More question types (fill-in, matching, ordering) | Multiple choice covers most elementary practice | Phase 2 |
| Spaced repetition | Simple prioritization (section 10) comes first | Later |
| Parent accounts, including linking a child's logins across workspaces (a school, a private tutor, a learning center) | Families share one phone; the teacher shares PINs through the roster. Linking a child across workspaces needs a parent to own the link, with consent | Phase 2 |
| English UI | Arabic market first | Later |
| Deleting classes and students, data export, account deletion | **Pilot blocker**: required before real student data | Phase 2 |
| Privacy policy and parental consent flow | **Pilot blocker**: required before any real student signs in | Phase 2 |
| Terms of service and refund policy | **Required before the first paying customer** | Phase 2 |
| Discussions or chat | Moderating children's messages is a safety burden | Not planned |
| Leaderboards and badges | Competition between young children isn't a goal | Not planned |

## Realism notes

- **Grades 1–2:** text-only questions realistically suit grades **3–6**. The MVP accepts grades 1–6 as data, but the demo content is grade 3.
- **AI quality:** AI drafts in Arabic for young children will sometimes be wrong or too hard. That's exactly why every draft goes through teacher review (D-08).
- **Size:** about **8 focused working days**, including setup, tests, CI and docs. The school calendar added about half a day, and the demo sign-in buttons with the nightly reset (D-20) another half day. Splitting classes and subjects (D-22) adds a few hours, within that estimate. The cut line below brings it to about 7. The implementation plan (step 6) refines this to about 8.5 days plus one buffer day.

## Cut line (if behind schedule)

Cut in this order, and list each cut under "Known limitations" in the README:

1. The "needs review" flag on the results page (keep the grid).
2. The printable roster (the teacher reads PINs from the class page).
3. The admin workspace list (keep the payment notices list).
4. AI summaries (keep AI questions, which are the bigger value).
5. Editing materials and teacher-written questions (keep create and delete; AI drafts stay editable).

**Never cut:** PIN lockout, tenant isolation, the audit record on payment decisions, and teacher review of AI drafts.

## Phase 1 definition of done

- Every in-scope feature is merged through its own PR with CI passing.
- Domain logic (access rules, subscription periods and calendar validation, AI quota, payment notice states, AI output validation, question selection, mastery, PIN lockout) is covered by unit tests, and the core E2E flow passes in CI.
- The app is deployed, and the README explains the demo sign-in, and has screenshots and an architecture overview.
- ADRs exist for the key decisions.
- No real student data and no real payments are processed.

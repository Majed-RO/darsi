# Darsi (دَرْسي) — Stage A Scope

> Status: **✅ Approved (v4)** · 2026-11-23
> v4 replaces the teacher-first Phase 1 scope (v3, approved 2026-11-22), which stays in git history. It follows PLAN.md decisions D-24 to D-43.
> Background, decisions, open questions and future features: [../PLAN.md](../PLAN.md)

## Stage A in one sentence

A grade-7 student learns and practises one unit, integers, on a phone and fairly independently: a short diagnostic test, then lessons and practice at the right level, with feedback and hints, and visible progress, without losing an answer when the connection drops.

## What Stage A must show

Stage A is a **family test with one student** (D-25, D-40). It can show whether the product works for a real learner. It can't show whether Darsi improves learning in general, or whether anyone will pay. It answers four questions:

1. **Independent use:** can the student find his way through the diagnostic test, the lessons and the practice without an adult explaining the app?
2. **Learning signal:** do the hints, misconception messages and worked solutions help him fix his mistakes, and do the skills the diagnostic found missing reach mastery?
3. **Connection drops:** are answers ever lost or counted twice when the connection drops and returns?
4. **Content quality:** is the reviewed content accurate, clear and at the right level?

The numbers that count as success are proposed under "Success criteria" and are set before the test starts (Q-11).

## Roles

| Role | Who | Can do in Stage A |
|---|---|---|
| **Teacher** | The owner only; Clerk sign-up is restricted to his email (D-38, D-40) | Create one class with a Math subject linked to the grade 7 math content, add students, get a one-time temporary PIN for each, reset it, delete a student's data. Never sign in as a student (D-45) |
| **Student** | The owner's son (D-25) | Sign in with the class code, his first name and a PIN he chooses himself; take the diagnostic; read lessons; practise; see his progress. Only he can open his space (D-45) |
| **Content author and reviewer** | The owner, outside the app | Write the content files with help from the AI script, and review everything before it's published (D-29, D-30) |

There's no admin area in Stage A: there are no payments, trials or subscriptions (D-32). The teacher's dashboard, skill-gap analysis and interventions are Stage B.

**Not tied to math or grade 7:** a subject, its grade and its units are data. Adding another subject or grade later means adding content, not changing code (D-24).

---

## In scope

### 1. Teacher setup (the minimum)

- Clerk sign-in. Sign-up is restricted to the owner's email; nobody else can create a teacher account.
- The first sign-in creates the teacher's workspace (D-13).
- Create a class with a name and a grade (1–12) and its join code from an alphabet without look-alike characters. Add one subject to it, linked to a subject and grade in the core content: in Stage A, math for grade 7, with its one unit (D-22, D-38).
- Add students by first name (unique within the class). Darsi generates a **one-time temporary 4-digit PIN** for each one and shows it once; only its hash is stored. The student replaces it with his own PIN at the first sign-in, so the teacher never knows the real one (D-45). Resetting gives a new temporary PIN the same way.
- **Delete a student completely:** the student, the sessions and every attempt (D-37, D-40). The phone removes that student's data the next time its sign-in screen opens with a connection (D-50).
- No other teacher pages in Stage A.

### 2. Student sign-in (D-10, D-38)

1. Enter the class code.
2. Pick a first name from the class list.
3. Enter the PIN on Darsi's keypad.
4. **First sign-in only:** choose a new 4-digit PIN, different from the temporary one, and type it twice. Nobody else sees it (D-45). If the temporary PIN was already replaced, the student sees "This temporary PIN has already been replaced. If you didn't choose a new PIN yourself, tell your teacher" (D-51).

- After **5 wrong PINs**, that student is locked out for **15 minutes**. The 4-digit PIN's weakness is accepted for Stages A and B and reviewed before Stage C.
- **App lock (D-45):** Darsi asks for the student's PIN every time it's opened and after **10 minutes** without use (configurable). It's checked on the phone, so it works offline; 5 wrong PINs lock it for 15 minutes. "Switch student" is available from the lock screen.
- The session is long-lived, because family phones are shared.
- **"Not you? Switch student"** is always visible. Each student's unsynced answers stay on the phone in their own queue and are sent the next time that student signs in with a connection.
- The sign-in screen shows how many answers each student has waiting on this phone ("Ahmad: 7 answers saved on this phone. Sign in to send them").
- Signing in needs a connection. Practising after signing in doesn't.

### 3. Core content: the integers unit (D-26, D-27, D-39)

**Order of topics:** the number line and opposites → comparing and ordering → absolute value → addition → subtraction. Multiplication and division come later.

**Draft skill map.** The final list (about 10–14 skills) is fixed while writing the content, checked against the UNRWA cards' objectives (V-01):

| Kind | Skill (draft) |
|---|---|
| Prerequisite | Compare and order whole numbers |
| Prerequisite | Add and subtract whole numbers |
| Prerequisite | Read positions on a number line with a given step |
| Prerequisite | Continue a number pattern with a constant step |
| Unit | Express real situations with integers (gain and loss, above and below sea level, temperature) |
| Unit | Place and read integers on the number line |
| Unit | Find the opposite of an integer |
| Unit | Compare two integers |
| Unit | Order a set of integers |
| Unit | Find the absolute value of an integer |
| Unit | Add two integers with the same sign |
| Unit | Add two integers with different signs |
| Unit | Subtract integers by adding the opposite |
| Unit | Solve short word problems with adding and subtracting integers |

**Each skill has:**
- its prerequisites in the skill map
- a short lesson: the objective, a short explanation, one or two worked examples, and a quick check question
- one or more **item templates**: small functions that generate questions with random numbers in a set range and compute the answer
- for each template: the expected **misconceptions**, each a wrong-answer pattern with its message (for example, −8 for (−5) − (−3)); **two hint levels** that never reveal the answer; and a **worked solution**

**Content rules:**
- Everything is written from scratch. UNRWA's cards give only the objectives and their order, and are never sent to an AI (D-26).
- AI drafts hints, misconception messages and worked solutions through a local script. The owner reviews every piece before it's published (D-29, D-30).
- Content lives as files in the repo, with a version number, and the content folder has its own "all rights reserved" license (D-39).
- CI checks every file against its schema, and generates thousands of items from each template to check their answers.
- The number line appears as a static picture in lessons and hints (D-27).

### 4. Diagnostic test

- About **12–16 questions, about 10 minutes**: two questions for each of up to 8 skills. They cover the prerequisites and the unit's key skills; the exact set is chosen while writing the content.
- **Two correct answers** for a skill mark it as mastered from the diagnostic (D-34). Skills the diagnostic doesn't cover start as "not started".
- No hints or feedback during the diagnostic; at the end the student sees a short summary of what he already knows and where he'll start.
- Taken once at the start. If it's interrupted, it resumes where it stopped. Retaking it is out of scope.

### 5. Learning path

- **Recommended next skill:** the first skill in the map's order that isn't mastered and whose prerequisites are mastered.
- **Free choice:** the student can open any skill. If its prerequisites aren't mastered, a short notice suggests starting with them, without blocking him (D-24).

### 6. Lessons

- A lesson is shown before a skill's first practice, and can be opened again at any time.
- Plain text with simple number formatting, worked examples and the static number line. No video or audio.

### 7. Practice

- A **session** is a round of up to **5 questions** from one skill, generated on the phone from that skill's templates.
- **Answer types (D-33):**
  - a whole number typed on Darsi's own keypad (٠–٩, minus, delete)
  - multiple choice, with comparison (<, >, =) and true/false as fixed-button variants
- **Feedback (D-35):**
  - a wrong answer shows its misconception message (or a general message if none matches) and allows another try
  - two hint levels on request
  - after two wrong tries, the worked solution is shown
  - a right answer shows "correct" at once
- **What counts for mastery (D-34):** only a first-try right answer without a hint. Later tries and hinted answers are recorded too.
- A short summary at the end of the session.
- Every answer becomes one immutable attempt record (D-37). A wrong answer that matches no known misconception is marked, so content gaps can be found (D-29).

### 8. Progress

- **Skill map** with each skill's state: not started, practising, mastered (D-34). Mastery drops back when later answers no longer meet the rule: the first wrong answer after a skill passed in the diagnostic, or fewer than 4 right in the last 5.
- For each skill: how many questions he answered and how many were right on the first try.
- The recommended next skill.

### 9. Connection drops: limited offline (L1, D-36)

- Darsi installs on the phone's home screen (PWA). The first visit caches the app and the whole unit.
- After signing in, lessons, the diagnostic and practice work without a connection: questions are generated and checked on the phone, and mastery is computed there.
- Answers are saved on the phone (IndexedDB) and synced when the connection returns: when the app opens, when the connection comes back, and after each session.
- The sync state is always visible: "3 answers waiting" or "Synced ✓".
- The server stores each answer once, checks it again, and keeps its own result if the two differ. A difference is flagged for the owner, and the student sees no sudden change.
- Darsi asks the browser to keep its storage permanently.
- **Not included:** opening Darsi for the first time offline, signing in offline, and updating content offline. Full offline support (L2) only if real use shows the need.

### 10. Privacy and data (D-30, D-40)

- First names only. No birth date, school or phone number.
- Student data stays in Darsi's own storage: the database, and the student's phone for offline practice and sync (D-36). It never goes into the repo, logs, screenshots or any AI prompt (D-48).
- No student data is sent to any AI; students don't interact with AI.
- A private deployment. Seed and test data are synthetic, written from scratch in Arabic.

### 11. Cross-cutting

- **Arabic, right to left, mobile first.** Tested on Android with Chrome (D-43); stays light on low-end phones.
- **Digits (D-41):** Arabic-Indic digits on screen; typed input accepts Arabic-Indic, Persian and Western digits; negatives shown as "-٥" with their direction isolated.
- **All UI text in one Arabic messages file.**
- **Tenant isolation:** the teacher sees only his workspace; a student sees only his own data. Enforced on the server and covered by tests.
- **Deployment:** a private deployment on Vercel with a Neon database. The Clerk development instance is enough. No public demo and no domain needed (D-40).

### 12. Data for the evaluation

A local script summarizes the family test from the database: sessions, attempts, mastery over time, hint use, wrong answers with no matching misconception, server mismatches, and sync statistics. Its output stays in `private/`.

---

## Not in Stage A

| Item | Where it goes |
|---|---|
| Teacher dashboard, skill-gap analysis, directing students to skills or practice | Stage B |
| Guardian sign-up, children with their own subjects, choosing subjects from the available content | Stage B (D-44) |
| Teacher additions to the content, and AI help for teachers | Later |
| Runtime AI for students (an "explain my mistake" feature or a tutor) | Later, after an evaluation (D-29) |
| Multiplication and division, other units, subjects and grades | Later (D-27, D-31) |
| Retaking the diagnostic, automatic review of mastered skills, spaced repetition | Later |
| Drag-and-drop ordering, interactive number lines, other answer types | Later (D-33) |
| Opening or signing in offline, several downloaded units (L2) | Later, if use shows the need (D-36) |
| Payments, trials, subscriptions, school calendar, admin area | When the business model is decided (D-32) |
| Public demo with "Try as" buttons and a nightly reset | Later (D-20, D-40) |
| Privacy notice, guardian consent, data export, sign-in security review | Stage C prerequisites |
| Chat, discussions and leaderboards | Not planned (D-11) |

---

## Success criteria (Q-11, proposed)

The test runs for **two weeks**, with at least **four practice sessions**. The owner notes each time his son asks for help with the app. The numbers below are proposals to confirm before the test starts:

| # | Question | Measure | Proposed target |
|---|---|---|---|
| 1 | Independent use | Help requests about using the app, after the first 5 minutes | At most 2 per session, falling over the two weeks |
| 2 | Correct checking | Answers marked wrongly (server mismatches, and cases where the student was right) | **0** |
| 3 | Connection drops | Answers lost or counted twice, across real drops and at least 3 deliberate airplane-mode tests | **0** |
| 4 | Content quality | Errors found in the reviewed content | Each one fixed and counted; no target, reported honestly |
| 5 | Learning signal | Skills the diagnostic found missing that reach mastery | At least half, reported as a description, not as proof |
| 6 | Hints | Hint uses followed by a right answer on the next try | Reported; no target |
| 7 | Voluntary return | Days he practised without being asked | Reported; no target, because a parent's influence can't be separated |

Every result states its limits: one student, the developer's son, and a unit he had already studied.

## Size

About **18 focused days of development**, plus about **4–5 days of content writing and review**. The implementation plan (step 6) refines this.

| Work | Days |
|---|---|
| Foundation: repo, tooling, CI, test setup, deployment | 1 |
| Teacher sign-in (restricted), workspace, class, subject, students, PINs, deleting a student | 1.5 |
| Student sign-in, sessions, lockout, switching students, choosing one's own PIN | 1 |
| Content files: schema, loading, validation, lesson display, digits and minus signs | 2 |
| Item templates, answer checking, the keypad | 2 |
| Feedback, hints and worked solutions | 1 |
| Diagnostic test and starting point | 1.5 |
| Mastery, learning path and progress | 1.5 |
| Attempts, the sync Route Handler and the server's re-check | 1.5 |
| Offline: service worker, caching, per-student queues on the phone, sync status, the app lock | 2.75 |
| The AI content-writing script | 0.5 |
| The evaluation script | 0.5 |
| Real-device checks, README, private deployment | 1 |
| **Total** | **≈ 17.75** |

This is more than earlier estimates. Per-student queues (D-38) and the lesson display were counted more carefully, the AI content-writing script was added when the implementation plan was written, and the student-chosen PIN and the app lock (D-45) add half a day.

## Realism notes

- **Content is the largest hidden cost.** Each skill needs a lesson, templates, misconceptions, hints and a worked solution, all reviewed: about 2–3 hours per skill.
- **Text-only lessons may not be enough to teach a new idea.** Stage A tests review and remediation of a unit the student has already studied.
- **A sample of one.** Stage A shows whether the product works for a real learner, not whether it improves learning in general.

## Cut line (if behind schedule)

Cut in this order, and list each cut under "Known limitations" in the README:

1. The end-of-session summary.
2. Per-skill counts on the progress screen (keep the states).
3. Fewer templates per skill (at least one each).
4. The evaluation script's summary (keep a raw export).

**Never cut:** correct answer checking and the server's re-check; no lost or duplicated answers; the privacy rules (D-40); the student-chosen PIN and the app lock (D-45); PIN lockout; tenant isolation; the owner's review of all content.

## Stage A definition of done

**Building is done when:**
- Every in-scope feature is merged through its own PR with CI passing.
- The domain rules are covered by unit tests: answer checking, templates, mastery, the learning path, the diagnostic, PIN lockout and sync. The E2E journeys pass, including practising offline and syncing afterwards.
- The content has passed the owner's review.
- The private deployment runs on the son's Android phone with Chrome.
- The README describes the current state (there's no public demo).
- No real student data is in the repo.

**Stage A is done when** the two-week family test has run, the success criteria have been reported with their limits, and the owner has decided whether to start Stage B.

# Runbook: Move Darsi to a private repo (start of Stage C)

> Decisions: [PLAN.md → D-17 and D-28](../PLAN.md#decisions-log)
> When: at the **start of Stage C**, before any data from students outside the family or any business-specific code is written.

Stages A and B are developed in a **public** repo under the Functional Source License (FSL). Stage C continues in a **private** copy with the full git history, and the public repo is archived as a read-only snapshot of Stages A and B.

In the commands below, replace:
- `<owner>` with your GitHub username
- `darsi` with the public repo's name
- `darsi-app` with the name you choose for the private repo

---

## 0. Before you start

- [ ] Merge or close every open PR in the public repo.
- [ ] Write down the settings you'll need to recreate in the private repo:
  - branch protection or rulesets on `main`, including the required CI checks
  - GitHub Actions secrets, variables and environments
  - the CodeRabbit installation
  - the hosting or deploy connection
  - any webhooks or other integrations
- [ ] **Check the costs that change when a repo becomes private.** On free plans, several things that are free for public repos are limited or paid for private ones. Check current pricing before you move:

  | Service | Public repo | Private repo |
  |---|---|---|
  | GitHub branch protection and rulesets | Free | Needs **GitHub Pro** (personal account) or **Team** (organization) |
  | GitHub Actions minutes | Unlimited | Monthly limit on the Free plan |
  | CodeRabbit | Free for public repos | Paid plan, or a limited free tier |

## 1. Tag the public state of the repo

```bash
git checkout main && git pull
```

```bash
git tag -a public-snapshot -m "Stages A and B (public, FSL)"
```

```bash
git push origin public-snapshot
```

## 2. Create the private repo

Create it **empty**: no README, license or .gitignore.

```bash
gh repo create <owner>/darsi-app --private
```

> Don't use GitHub's **Fork** button. A fork of a public repo stays public.

## 3. Copy the full history into the private repo

Run this in a temporary folder, outside your working copy:

```bash
git clone --bare https://github.com/<owner>/darsi.git darsi-bare
```

```bash
cd darsi-bare && git push --mirror https://github.com/<owner>/darsi-app.git
```

```bash
cd .. && rm -rf darsi-bare
```

This copies every branch, tag and commit. Pull requests and issues are **not** copied, and they stay readable in the archived public repo.

## 4. Point your working copy at the private repo

```bash
git remote set-url origin https://github.com/<owner>/darsi-app.git
```

```bash
git remote -v
```

Don't keep the public repo as a second remote, so private work can never be pushed there by mistake.

## 5. Set up the private repo

- [ ] Recreate branch protection or rulesets on `main`, with the required CI checks.
- [ ] Add the Actions secrets, variables and environments.
- [ ] Install CodeRabbit on the new repo and choose a plan.
- [ ] Reconnect hosting and deployment to the new repo.
- [ ] Replace `LICENSE` with a proprietary notice ("Copyright © <year> Majed-RO. All rights reserved."), using the same GitHub name as the public license, so the owner's real name stays out (D-42). The FSL is no longer needed in a private repo. The public snapshot keeps it. `content/LICENSE.md` stays "all rights reserved" (D-39).
- [ ] Keep the repo's local git identity (GitHub name and no-reply email) in the working copy.
- [ ] Open a small test PR and confirm that CI, CodeRabbit and the preview deploy all work.

## 6. Archive the public repo

Do the README change **first**, because an archived repo is read-only.

- [ ] Add a notice at the top of the public README: *"This repository is a snapshot of Darsi's Stages A and B. Development continues privately."* Keep the screenshots (synthetic data only) and the architecture overview, and the demo link if a public demo exists, since reviewers still read this repo.
- [ ] Make that change **in the public repo only**, never from your working copy, which now points at the private repo (section 4): either with GitHub's web editor, or in a fresh clone of the public repo. The `main` ruleset blocks direct pushes, so put it on a new branch and open a PR. Change only the README, so the public `LICENSE` (FSL) stays as it is. Merge it, then delete that clone.
- [ ] Archive it:

```bash
gh repo archive <owner>/darsi --yes
```

## 7. Separate the deployments

- [ ] The production deployment for Stage C deploys only from the private repo. Move the Stage A deployment and its database (the family test's data, D-40) to the private repo's hosting, or delete that data if it's no longer needed.
- [ ] If a public demo stays online for reviewers (D-20), it must use **synthetic data and its own database**. It must never point at the production database with real students and teachers.

## 8. Final checks

- [ ] The private repo has every branch and tag: compare `git ls-remote` output for both repos.
- [ ] The public repo shows GitHub's **Archived** banner and is still readable.
- [ ] Links on your CV and portfolio still work. They point to the archived public repo.
- [ ] Record the move in the private repo's PLAN.md decisions log.

<div align="center">

# ☁️ Cloud DevOps Interview Guide

### Topic-wise Questions, Answers, Differences, Scenarios & Command Examples

`Git` · `Linux` · `Networking` · `CI/CD` · `Terraform` · `Docker` · `Kubernetes` · `Python`

</div>

---

## 📑 Table of Contents

| # | Topic | Questions |
|---|-------|-----------|
| 1 | [🔀 Git](#git) | 17 |
| 2 | [🐧 Linux](#linux) | 15 |
| 3 | [🌐 Networking](#networking) | 9 |
| 4 | [⚙️ CI/CD](#cicd) | 15 |
| 5 | [🏗️ Terraform](#terraform) | 2 |
| 6 | [🐳 Docker](#docker) | 12 |
| 7 | [☸️ Kubernetes](#kubernetes) | 29 |
| 8 | [📊 Observability](#observability) | Coming soon |
| 9 | [☁️ AWS](#aws) | Coming soon |
| 10 | [🔷 Azure](#azure) | Coming soon |
| 11 | [🐍 Python](#python) | 3 |

<br>

---

<br>

<a id="git"></a>
# 🔀 TOPIC 1: GIT

> Fork vs clone, fetch vs pull, merge vs rebase, branching strategies, conflicts, tags, history cleanup and secret removal.

<br>

---

### Q1. What is the difference between `git fork` and `git clone`, and when would you use each?

#### 📌 Short Explanation
This question is often asked to check if you understand collaboration workflows in Git — especially open-source and team projects. Many developers confuse fork and clone, so it helps to clarify the purpose and use cases of both.

#### ✅ Answer
- **`git fork`** creates a copy of a repository on your GitHub (or GitLab, etc.) account, letting you propose changes without write access to the original repo.
- **`git clone`** creates a local copy of any Git repository (your own or someone else's) on your machine for development.

#### 📖 Detailed Explanation
When you fork a repository on GitHub, you're telling the platform:
> "I want a separate version of this repository in my own GitHub account."

This is especially useful for contributing to open-source and team projects where you don't have direct write access to the main repository. You fork the repo, make changes in your fork, and then create a pull request to propose those changes to the original project.

On the other hand, `git clone` downloads a repository (forked or original) to your local development machine. This is what actually gives you the codebase to work with.

Typical usage of both:
1. Fork the repo on GitHub (creates a copy under your GitHub username).
2. Clone your fork locally:

```bash
git clone https://github.com/your-username/the-repo.git
```

> **So:** Fork = GitHub-level action, Clone = Local machine-level action.

<br>

---

### Q2. Explain a scenario where you used `git fork` instead of `git clone`. Why was forking necessary?

#### ✅ Answer
I used `git fork` when I contributed to a DevOps project in my org on GitHub. Since I didn't have write access to the original repository, I forked it into my GitHub account, made changes, and then created a pull request from my fork to the upstream repo.

#### 📖 Detailed Explanation
The original repository belonged to an organization. I wanted to fix a bug in their Helm chart setup for Kubernetes deployments. Because I didn't have contributor rights to push directly, I used the **Fork** button on GitHub to create a personal copy under my own username.

From there:

1. Clone my fork to my local system:
   ```bash
   git clone https://github.com/my-username/devops-helm-project.git
   ```
2. Create a new branch, make the fix, commit the changes.
3. Push the branch to my fork:
   ```bash
   git push origin bugfix-helm-values
   ```
4. Submit a pull request to the original repository.

Using `git clone` directly on the upstream repo wouldn't have helped because I couldn't push changes or open a PR without a fork. Forking gave me independence and write access on my own terms, while still contributing back to the main project.

<br>

---

### Q3. What is the difference between `git fetch` and `git pull`, and when would you use each?

#### 📌 Short Explanation
This question checks if you understand how Git handles remote updates. Many developers use `git pull` out of habit but don't realize that it's a combination of two actions — which `git fetch` separates for more control.

#### ✅ Answer
- **`git fetch`** retrieves the latest changes from the remote repository **without** merging them into your current branch.
- **`git pull`** does the same as fetch but also **automatically merges** the changes into your current branch.

#### 📖 Detailed Explanation
When you run `git fetch`, Git contacts the remote (like GitHub) and downloads any changes (new commits, branches, tags) — but does not apply them to your working directory.

```bash
git fetch origin
```

Useful when:
- You want to see what others have pushed
- You're preparing for a manual merge or rebase
- You want to avoid surprise changes to your working branch

With `git pull`, you do this **plus** merge the changes into your current branch in one step:

```bash
git pull origin main
```

That's shorthand for:

```bash
git fetch origin
git merge origin/main
```

While `git pull` is faster, it can cause unintended merges if you're not ready. That's why many teams prefer doing `fetch` first, reviewing the changes, and then merging or rebasing manually.

> **Summary:**
> Use `git fetch` when you want control.
> Use `git pull` when you're ready to sync changes directly.

<br>

---

### Q4. Which command do you use more often — `git fetch` or `git pull`, and why?

#### 📌 Short Explanation
This question explores your Git workflow habits and whether you prefer a manual or automated approach to syncing changes from a remote repository.

#### ✅ Answer
I mostly use `git pull` because it streamlines my workflow by fetching and merging remote changes in one step. It's convenient for staying up to date quickly, especially when collaborating in fast-moving branches.

#### 📖 Detailed Explanation
`git pull` is essentially a shortcut that performs both a `git fetch` and a `git merge`. Instead of running two separate commands, I prefer:

```bash
git pull origin main
```

This makes my routine faster and keeps my local branch synchronized with the remote without extra steps. It's particularly useful when:
- I'm working alone or in a small team where merge conflicts are rare
- I'm contributing to a feature branch that others aren't modifying
- I want to frequently pull in the latest changes to test or deploy updates

I stay cautious by committing or stashing local changes before pulling to avoid conflicts or interrupted workflows. And if I suspect major upstream changes or want a closer look, I temporarily switch to `git fetch`.

For day-to-day development, especially in active branches, `git pull` keeps things fast and simple — and that makes it my go-to.

<br>

---

### Q5. What is the difference between `git rebase` and `git merge`? When would you use each?

#### 📌 Short Explanation
This question evaluates your understanding of how Git manages branch history and collaboration. Both commands integrate changes from one branch to another — but they do it in very different ways.

#### ✅ Answer
- **`git merge`** integrates changes by creating a new **merge commit**, preserving the history of both branches.
- **`git rebase`** moves your branch on top of another, **rewriting commit history** to create a linear sequence.

#### 📖 Detailed Explanation
Say you have two branches: `main` and `feature` (branched off earlier from `main`).

**🔹 Using `git merge`:**

```bash
git checkout feature
git merge main
```

This pulls changes from `main` into `feature` and creates a merge commit:

```text
A---B---C (main)
     \
      D---E---F (feature)
               \
                G (merge commit)
```

| ✅ Pros | ❌ Cons |
|--------|--------|
| Preserves full history and context | History becomes messy with many merge commits |
| Safer in teams: no history rewriting | Harder to trace linear commit flow |
| Good for long-lived shared branches | |

**🔹 Using `git rebase`:**

```bash
git checkout feature
git rebase main
```

This re-applies your commits on top of the latest `main`:

```text
A---B---C (main)
         \
          D'---E'---F' (rebased feature)
```

| ✅ Pros | ❌ Cons |
|--------|--------|
| Clean, linear history | Rewrites commit history |
| Easier `git log` and `git bisect` | Risky if already pushed and others based work on it |
| Preferred before merging short-lived branches into main | Not ideal for shared/public branches |

**🔹 When to Use What**

| Use `merge` when... | Use `rebase` when... |
|---------------------|----------------------|
| You're collaborating on shared branches | You're working alone or before a PR merge |
| You want to preserve commit context | You want a clean, linear history |
| History safety is a concern | You're cleaning up before pushing |

> **Short answer:**
> `git merge` integrates changes by creating a new merge commit, preserving the history of both branches.
> `git rebase` moves your branch on top of another, rewriting commit history to create a linear sequence.
>
> **Summary:** Use merge to combine, use rebase to simplify.

<br>

---

### Q6. Explain the Git branching strategy you used in your company. Align it with the open-source branching strategy followed by Kubernetes.

#### 📌 Short Explanation
This question explores how you organize your Git workflow in a collaborative environment — especially in large codebases. Kubernetes, like many open-source projects, uses a clean and scalable branching strategy.

#### ✅ Answer
In my company, we followed a well-structured Git branching model similar to the Kubernetes project's workflow, centered around four key branches:

| Branch | Purpose |
|--------|---------|
| `main` | The default and stable development branch |
| `feature/*` | All new features and enhancements |
| `release/*` | Preparing and testing production releases |
| `hotfix/*` | Urgent bug fixes or patches to production |

This helped us maintain stability while enabling parallel development and quick recovery from issues.

#### 📖 Detailed Explanation

**🔹 `main` branch**
- Equivalent to `master`/`main` in Kubernetes.
- Represents the latest development state — stable but evolving.
- All feature branches are branched off from here.
- Protected with branch rules and mandatory code reviews.

> Developers do not commit directly to `main`. All changes go through pull requests.

**🔹 `feature/*` branches**
- Used for individual features or enhancements.
- Named like `feature/login-api` or `feature/cleanup-metrics`.
- Created from `main`.
- Developers work independently and raise PRs when done.

> We squash commits before merging to keep the history clean.

**🔹 `release/*` branches**
- Cut from `main` when preparing for a release, e.g., `release/1.4`.
- Only allows bug fixes, performance improvements, and docs.
- CI pipelines run regression tests and validations here.
- Used for staging deployments and QA approvals.

> Kubernetes also creates release branches (e.g., `release-1.28`) to stabilize features after code freeze.

**🔹 `hotfix/*` branches**
- Created from the latest release tag or `main`, based on urgency.
- Used to fix critical bugs directly on production without waiting for the next release cycle.
- After fixing and testing, changes are merged back to both `main` and the relevant `release/*` branch.

> This ensures the fix is available in both the short term and future releases.

**🔹 Benefits of this Strategy**
- Supports parallel development and safe releases
- Keeps `main` clean and always deployable
- Makes it easy to trace features and bug fixes
- Aligns well with CI/CD automation and changelog generation

> By following this strategy, we maintained agility without compromising stability — critical in both enterprise and open-source scale environments like Kubernetes.

<br>

---

### Q7. Explain three challenges you faced while using Git in your work experience.

#### 📌 Short Explanation
This question evaluates real-world Git usage and how well you've handled common pain points like collaboration, history management, and contribution workflows.

#### ✅ Answer

**1️⃣ Merge Conflicts During Pulls**
I used to rely heavily on `git pull` without checking what changes others had pushed. This led to merge conflicts, especially in fast-moving branches. Eventually, I switched to `git fetch` followed by a manual merge or rebase, which gave me more control over how I integrated changes.

**2️⃣ Messy Commit History with Frequent Merges**
Early in my career, I used `git merge` frequently on long-lived feature branches. It cluttered the history with multiple merge commits. I learned to use `git rebase` (before pushing) to create a clean, linear history — especially before opening pull requests.

**3️⃣ Confusion Between Fork and Clone in Open-Source Work**
When I first started contributing to open source, I cloned repositories directly and couldn't push my changes. I realized I should have used `git fork` to create my own copy on GitHub. After forking, I could push to my own version and submit pull requests to the original repository.

#### 📖 Detailed Explanation
These challenges reflect how Git is powerful but not always beginner-friendly:
- **Merge conflicts** are common in collaborative teams. Using `git fetch` and reviewing changes before merging helped avoid surprise conflicts.
- **Messy commit history** makes debugging and code reviews painful. Rebasing local branches before pushing made history easier for teammates to follow.
- **Forking confusion** taught me GitHub's collaboration model. Knowing when to fork vs. clone was key to contributing effectively.

<br>

---

### Q8. Explain a recent challenge you faced with Git and how you addressed it.

#### 📌 Short Explanation
This question assesses your experience with Git at scale — especially around collaborative processes and governance.

#### ✅ Answer
I implemented a consistent Git branching strategy across **100+ repositories** used by multiple teams. Each team followed their own style — some had `main/dev`, others `master/feature`, and a few used no long-lived branches at all. This inconsistency made CI/CD pipelines error-prone, releases chaotic, and collaboration difficult.

We standardized on a lightweight **trunk-based** strategy with well-defined rules around `main`, `release/*`, and `feature/*` branches — and rolled it out in phases.

#### 📖 Detailed Explanation
Problems caused by the lack of a unified approach:
- CI pipelines failing due to missing expected branches like `main` or `release`
- Some teams rebased public branches, which broke collaborators' work
- Merge conflicts were common in integration environments
- Releases were delayed due to confusion about which branch was production-ready

**Step 1 — Analyze the current state**
- Audited all repositories using automation (GitHub/GitLab APIs).
- Documented the branching models and naming conventions each team used.

**Step 2 — Design a unified strategy**
- Proposed a trunk-based development model:
  - `main` → always production-ready
  - `release/x.y` → stabilization and hotfixes
  - `feature/*` → short-lived, rebased before merge
- Outlined rules for using merge, rebase, and protection policies.

**Step 3 — Rollout with tooling & education**
- Created templates (starter repos) with the correct branch structure.
- Set up default branch protections and PR requirements using GitHub Actions and branch policies.
- Ran onboarding sessions and created a lightweight Git handbook.

**Step 4 — Iterate with feedback**
- Incorporated feedback from platform, dev, and QA teams.
- Allowed temporary exceptions during migration.

**🎯 Result**
- 95%+ repos aligned within 2 months
- CI/CD pipeline reliability improved significantly
- Teams were clearer on how and when to branch or merge
- Easier onboarding of new developers and automation of release workflows

<br>

---

### Q9. How do you handle merge conflicts in Git?

#### 📌 Short Explanation
Merge conflicts are a common part of collaborative Git workflows. This question tests how calmly and effectively you resolve conflicts and whether you understand why they occur.

#### ✅ Answer
When I encounter a merge conflict, I pause to understand which files are affected, examine the conflicting changes, and manually resolve them using a visual diff tool or editor. Once resolved, I mark the conflicts as resolved, stage the changes, and complete the merge or rebase.

#### 📖 Detailed Explanation
Merge conflicts usually happen when:
- Two branches modify the same lines in a file
- One branch deletes a file the other modifies
- A rebase applies commits that overlap with existing changes

**Step 1 — Identify the conflict**

```text
Auto-merging app.py
CONFLICT (content): Merge conflict in app.py
```

**Step 2 — Open and resolve the conflict**

Conflicted files contain markers like:

```text
<<<<<<< HEAD
print("Hello from main")
=======
print("Hello from feature-branch")
>>>>>>> feature-branch
```

Edit the file to reflect the correct final code. Helpful tools:
- `git diff` to understand the changes
- Visual Studio Code or GitKraken for visual resolution

**Step 3 — Mark as resolved**

```bash
git add app.py
git commit              # if using merge
# or
git rebase --continue   # if using rebase
```

> Merge conflicts aren't errors — they're just Git asking you to make a decision. Handling them well is part of being a collaborative engineer.

<br>

---

### Q10. What are `ours` and `theirs` strategies in Git merges? How and when are they used?

#### 📌 Short Explanation
This checks whether you understand how to control merge behavior during conflicts — especially in tricky cases like rollbacks, overrides, or integration branches.

#### ✅ Answer
- **`ours`** favors your current branch's changes, even if the other branch has different content.
- **`theirs`** isn't directly available as a merge strategy but can be used during conflict resolution (rebase or manual merge) to accept incoming changes over yours.

#### 📖 Detailed Explanation

**🔹 `ours` strategy (as a Git merge strategy)**

> "Even though we're merging, if there's a conflict — pick my current branch's version."

```bash
git merge -s ours feature-branch
```

⚠️ This does **not** merge and keep both sets of changes. It pretends to merge but keeps only your current branch's content, marking the merge as done.

**Use cases:**
- Rolling back a hotfix and keeping the current stable state
- Merging a long-dead branch just to close it while keeping your branch's state intact

**🔹 `theirs` (used manually during conflicts)**

There is no `-s theirs` strategy at the command-line merge level. However, when resolving a conflict manually you can choose the incoming branch's changes:

```bash
git checkout --theirs conflicted_file.txt
git add conflicted_file.txt
```

> "For this conflicted file, discard my local version and use the version from the branch I'm merging in."

**Use cases:**
- The incoming changes are definitely correct
- During `git rebase` when resolving repetitive or well-understood conflicts

**🔹 Summary Table**

| Strategy | Behavior | Use When... |
|----------|----------|-------------|
| `ours` | Keeps **current branch's** content | You want to keep your version, discard the incoming one |
| `theirs` | Keeps **incoming branch's** content | You want to override with the other branch's changes |

<br>

---

### Q11. Have you ever used Git tags? If yes, why?

#### 📌 Short Explanation
This checks if you're familiar with versioning and release practices in Git. Tags mark stable points in history — especially important in CI/CD pipelines and production deployments.

#### ✅ Answer
Yes, I've used Git tags primarily to mark release versions of our applications. It helps track which commit corresponds to a production deployment, and makes it easier to roll back or audit changes when needed.

#### 📖 Detailed Explanation
In one of my recent projects, every stable build that passed all stages in our CI/CD pipeline was tagged with a version number — like `v1.0.3`.

I used annotated tags to add context:

```bash
git tag -a v1.0.3 -m "Release version 1.0.3 with critical bug fixes"
git push origin v1.0.3
```

These tags were picked up by Jenkins and used as part of the deployment name — so we always knew what version was running in production.

**🔹 Why Git Tags Are Useful**
- 📍 **Marking release points** — indicate stable or milestone commits
- ⏪ **Rollback support** — check out a tag to return to a known good state
- 🏗️ **Versioned builds** — many CI systems trigger builds based on tags
- 🔖 **Consistent releases** — tags act like bookmarks for deployments or patch notes

> **Summary:** Tags improve visibility, traceability, and control in software releases — they're lightweight, powerful, and essential in production workflows.

<br>

---

### Q12. How do you combine multiple commits into a single commit in Git?

#### 📌 Short Explanation
This question evaluates your understanding of Git history manipulation — especially commit hygiene, squashing, and preparing a clean pull request.

#### ✅ Answer
I use **interactive rebase** to squash multiple commits into a single, meaningful commit before pushing. This keeps the Git history clean and easy to read.

#### 📖 Detailed Explanation
Say I made 4 commits while working on a single feature:

```bash
git log --oneline
```
```text
abc123 Fix typo
def456 Add input validation
ghi789 Update error message
jkl012 Initial work on login form
```

Before raising a PR, I combine them into one commit, e.g. *"Add login form with validation and error handling"*.

**Step 1 — Start interactive rebase**

```bash
git rebase -i HEAD~4
```

**Step 2 — Change all but the first `pick` to `squash` (or `s`)**

```text
pick   jkl012 Initial work on login form
squash ghi789 Update error message
squash def456 Add input validation
squash abc123 Fix typo
```

**Step 3 — Write the new commit message, save and exit**

**Step 4 — Push**

```bash
git push origin branch-name --force
```

> ⚠️ Force-push only if the commits were already pushed to remote. Otherwise a normal push is fine.

**🔹 Why This Is Useful**
- Keeps Git history clean and easier to understand
- Combines related changes into one atomic commit
- Helpful for reviewers and future debugging
- Commonly used before merging a feature branch into `main`

<br>

---

### Q13. Explain 10 Git commands that you use on a day-to-day basis. What are they used for?

#### 📌 Short Explanation
This question evaluates your hands-on comfort with Git — whether you only follow the basics or use Git effectively for version control and collaboration.

#### ✅ Answer

| # | Command | Example | Purpose |
|---|---------|---------|---------|
| 1 | `git clone` | `git clone https://github.com/org/repo.git` | Create a local copy of a remote repo |
| 2 | `git status` | `git status` | Show what's staged, unstaged, and untracked |
| 3 | `git add` | `git add file.py` / `git add .` | Stage changes before committing |
| 4 | `git commit` | `git commit -m "Fix login bug"` | Save a snapshot of staged changes |
| 5 | `git push` | `git push origin feature-branch` | Upload local commits to the remote |
| 6 | `git pull` | `git pull origin main` | Fetch and merge remote changes |
| 7 | `git fetch` | `git fetch origin` | Download remote changes without merging |
| 8 | `git branch` | `git branch` / `git branch feature/login` | List or create branches |
| 9 | `git checkout` | `git checkout main` / `git checkout -b hotfix/typo` | Switch (or create and switch) branches |
| 10 | `git rebase` | `git rebase main` | Re-apply commits on top of another branch for clean history |

<br>

---

### Q14. I want to ignore pushing changes to a specific file in Git. How can I do it?

#### 📌 Short Explanation
This tests your understanding of how Git tracks files, how `.gitignore` works, and how to prevent accidental pushes of sensitive or local configuration files.

#### ✅ Answer
To ignore future changes to a **tracked** file, use the `assume-unchanged` flag. This tells Git to stop checking the file for changes, even though it's still in the repo.

```bash
git update-index --assume-unchanged path/to/your/file
```

#### 📖 Detailed Explanation
There are two main scenarios:

**🔹 1. The file is already tracked, and you want Git to stop tracking changes**

```bash
git update-index --assume-unchanged file.txt
```

The file stays in the repo, but Git acts like it hasn't changed — useful for config files that differ by environment.

To undo and start tracking again:

```bash
git update-index --no-assume-unchanged file.txt
```

Common use cases:
- `.env` files with local credentials
- `settings.json` or editor-specific config
- Scripts that are tweaked temporarily

**🔹 2. The file should never be tracked**

Add it to `.gitignore` **before** committing it:

```text
# .gitignore
.env
*.log
```

This works only for untracked files. If it's already committed once, untrack it first:

```bash
git rm --cached file.txt
echo file.txt >> .gitignore
git commit -m "Stop tracking file.txt"
```

> ⚠️ **Note:** `assume-unchanged` is a local-only flag. It won't prevent others from seeing changes or pushing the file. It's a lightweight trick, not a security mechanism.

> **Summary:**
> Use `.gitignore` for new/untracked files.
> Use `assume-unchanged` for tracked files you don't want to accidentally push.

<br>

---

### Q15. What is the purpose of the `.git` folder in a Git repository?

#### 📌 Short Explanation
This checks your foundational understanding of how Git works internally. The `.git` directory is the backbone of any Git repository.

#### ✅ Answer
The `.git` folder contains all the metadata, configuration, and object database Git needs to manage version control. It is what transforms an ordinary directory into a Git repository.

#### 📖 Detailed Explanation
When you run `git init`, Git creates a hidden `.git/` directory in the project root.

**🔹 Key Contents of `.git/`**

| Item | Purpose |
|------|---------|
| `HEAD` | Reference to the current checked-out branch — "where you are" |
| `config` | Local repo settings: remote URLs, user info, aliases |
| `objects/` | The actual content — all commits, trees, and blobs (compressed) |
| `refs/` | References to all branches and tags (e.g. `refs/heads/main`, `refs/tags/v1.0.0`) |
| `logs/` | Records of ref updates (used for recovery, e.g. `git reflog`) |
| `index` | The staging area — where files go after `git add` and before `git commit` |

**🔹 Why It's Important**
- Without `.git`, your project is no longer a Git repository.
- If you delete or corrupt it, Git can no longer track changes.
- Copying the `.git` folder to another directory essentially clones the repo without a remote.

> ⚠️ **Common Pitfall:** Developers sometimes accidentally delete `.git` when cleaning up, which removes all Git history — not just local changes.

> **Summary:** The `.git` directory is the internal database and control center of your repository.

<br>

---

### Q16. Can you restore a deleted `.git` folder?

#### 📌 Short Explanation
This evaluates your knowledge of Git internals and backup strategies. Accidentally deleting `.git` wipes out version control history — unless you have a fallback.

#### ✅ Answer
**No**, you cannot restore a deleted `.git` folder on your own unless you have:
- A **backup** of the folder (manual or automated), or
- A **remote copy** of the repository (e.g., on GitHub or GitLab)

#### 📖 Detailed Explanation
The `.git/` folder holds all commits, branch info, tags, staging data, and remote config. If you delete it:

```bash
rm -rf .git
```

Your project becomes a regular directory with no version history.

**🔹 Option 1 — You have a remote (like GitHub)**

```bash
# Move existing code out
mkdir backup && mv * backup/

# Clone fresh repo
git clone https://github.com/your/repo.git

# Move your untracked files back in
mv backup/* repo/
```

Then `git add`, commit, and push any uncommitted changes.

**🔹 Option 2 — You have a backup of `.git`**

```bash
mv .git.bak .git
git status
```

Full history and branches are intact.

**🔹 Option 3 — File recovery tools (low chance)**
Tools like `extundelete` or `recuva` might recover parts of it if it was recently deleted and not overwritten — but success is rare and unreliable.

**🔹 Best Practices to Prevent This**
- Push frequently to a remote
- Enable automatic backups or snapshot tools
- Avoid `rm -rf` without double-checking
- Use Git GUIs or IDEs to reduce accidental deletion

> **Summary:** Once `.git` is gone and you have no remote or backup, the project loses its entire Git history. The safest recovery is to clone from the remote and reapply local, uncommitted changes manually.

<br>

---

### Q17. A teammate accidentally committed a Kubernetes Secret (base64 encoded) to Git. What should you do?

#### 📌 Short Explanation
This scenario tests how you respond to a security incident and how well you understand Git history rewriting, sensitive data handling, and Kubernetes secrets management.

#### ✅ Answer
Immediately **rotate** the compromised secret, **remove** it from Git history using `git filter-repo` or BFG, and **enforce** better secret management (e.g., Sealed Secrets or external secret stores).

#### 📖 Detailed Explanation
When a Kubernetes Secret (even base64-encoded) is committed, it becomes visible to:
- Everyone with repo access
- Anyone who forked or cloned the repo before removal
- CI/CD systems that fetch the repo

**🔹 Step 1 — Rotate the compromised secret**

Whether it's an API key, DB password, or token — assume it's compromised.
- Create a new secret value
- Update the Kubernetes Secret:

```bash
kubectl create secret generic my-secret --from-literal=password=newpassword --dry-run=client -o yaml | kubectl apply -f -
```

- Update any workloads consuming the old secret.

**🔹 Step 2 — Remove the secret from Git history**

> Base64 encoding is **not** encryption. Anyone can decode it.

If recently committed:

```bash
git reset HEAD~1
git restore --staged secret.yaml
rm secret.yaml
git commit -m "Remove secret from repo"
```

This only removes it from the latest commit. To fully remove it from entire history, use [`git filter-repo`](https://github.com/newren/git-filter-repo) (preferred over `filter-branch`):

```bash
git filter-repo --path secret.yaml --invert-paths
```

Or use BFG Repo-Cleaner:

```bash
bfg --delete-files secret.yaml
```

Then force-push:

```bash
git push --force
```

> Everyone with clones will need to re-clone or follow special instructions to realign their history.

**🔹 Step 3 — Prevent it from happening again**
- Add sensitive files to `.gitignore`:
  ```text
  secrets.yaml
  *.key
  ```
- Use [`git-secrets`](https://github.com/awslabs/git-secrets) or [`pre-commit`](https://pre-commit.com/) to scan commits for secrets.
- Educate team members: Kubernetes Secrets are not encrypted by default in Git, even though they look scrambled (base64).

**🔹 Why This Matters**
Hardcoded secrets in Git are one of the most common security missteps. Even private repos can be compromised.

> **Summary:** Rotate → Remove → Recommit safely → Enforce policies.

<br>

---

<br>

<a id="linux"></a>
# 🐧 TOPIC 2: LINUX

> Daily commands, disk & CPU troubleshooting, SSH/EC2 access recovery, log management, shell scripting and automation.

<br>

---

### Q1. What are 10 Linux commands you use daily? (Excluding basic ones like `ls` and `cd`)

#### 📌 Short Explanation
This question assesses your hands-on Linux experience, focusing on diagnostic, file manipulation, service management, and automation-related commands beyond basic navigation.

#### ✅ Answer

| # | Command | Example | Purpose |
|---|---------|---------|---------|
| 1 | `tail -f` | `tail -f /var/log/nginx/error.log` | Monitor log files in real time — great for live debugging |
| 2 | `grep` | `grep -i "timeout" /var/log/app.log` | Search files, logs, or output for patterns |
| 3 | `systemctl` | `systemctl restart nginx` | Start, stop, restart, check status of systemd services |
| 4 | `journalctl` | `journalctl -u docker.service -f` | View logs for systemd-managed services (Docker, Kubelet) |
| 5 | `ps aux \| grep` | `ps aux \| grep nginx` | List running processes; find rogue or heavy ones |
| 6 | `df -h` / `du -sh` | `df -h` · `du -sh *` | Check free disk space / folder sizes |
| 7 | `chmod` / `chown` | `chmod +x deploy.sh` · `chown ubuntu:ubuntu script.sh` | Manage permissions and ownership (common in CI/CD) |
| 8 | `find` | `find /var/log -name "*.log" -mtime +7` | Locate files by name, date, type — great for cleanup/audits |
| 9 | `curl` | `curl -I http://localhost:8080` | Test web endpoints, APIs, service availability |
| 10 | `rsync` | `rsync -avz /app/ user@server:/backup/` | Efficient file sync and backup — better than `scp` for large dirs |

<br>

---

### Q2. Can you restore a lost PEM file? If not, how can you still access the EC2 instance?

#### 📌 Short Explanation
This evaluates your knowledge of secure SSH access, key-based authentication, and how to regain access to EC2 instances when your only login method (PEM file) is lost.

#### ✅ Answer
**No**, you cannot restore a lost PEM file — it's not stored on AWS and is not recoverable. However, you can regain access by creating a new key pair, attaching it to the instance via a temporary EC2 **rescue process**, and restoring SSH access.

#### 📖 Detailed Explanation
PEM files (private keys) are never retrievable from AWS after initial creation. Here is how to regain access:

**Step 1 — Create a new key pair**

```bash
aws ec2 create-key-pair --key-name new-key --query 'KeyMaterial' --output text > new-key.pem
chmod 400 new-key.pem
```

**Step 2 — Stop the affected instance**

```bash
aws ec2 stop-instances --instance-ids i-xxxxxxxxxxxxxxx
```

**Step 3 — Detach the root EBS volume** from the stopped instance
- EC2 → Volumes → find the volume (usually `/dev/xvda`) → Detach

**Step 4 — Attach the volume to a temporary (working) instance**
- Attach it as a secondary volume (e.g., `/dev/sdf`)

**Step 5 — SSH into the temporary instance and mount the volume**

```bash
sudo mkdir /mnt/recovery
sudo mount /dev/xvdf1 /mnt/recovery
sudo nano /mnt/recovery/home/ec2-user/.ssh/authorized_keys
```

Add the **public key** from your new key pair.

**Step 6 — Detach from the rescue instance**
- Unmount, detach, and re-attach to the original instance as `/dev/xvda`

**Step 7 — Start the original instance and SSH with the new key**

```bash
ssh -i new-key.pem ec2-user@<public-ip>
```

**🔹 Prevention Tips**
- Always back up PEM files in a secure location (password manager)
- Create a secondary user with a different key for backup access
- Use EC2 Instance Connect for temporary browser-based access (Amazon Linux 2+ with enabled roles)

> **Summary:** You cannot restore a PEM file, but you can regain access by editing `authorized_keys` on the root volume through a temporary rescue EC2 instance.

<br>

---

### Q3. `/var` is almost 90% full. What will be your next steps?

#### 📌 Short Explanation
This checks your troubleshooting and disk management skills. `/var` holds logs, spools, caches, and runtime data — issues here can break system processes or silently fill up disks.

#### ✅ Answer
Identify what's consuming space inside `/var`, clean up unnecessary files (rotated logs, caches, orphaned packages), and put alerts or log rotation in place to avoid recurrence.

#### 📖 Detailed Explanation

**Step 1 — Inspect disk usage under `/var`**

```bash
sudo du -sh /var/* | sort -hr | head -10
```
Usually the culprit is `/var/log`, `/var/cache`, or `/var/lib/docker`.

**Step 2 — Clean log files**

```bash
sudo journalctl --vacuum-size=200M
sudo rm -rf /var/log/*.gz /var/log/*.[0-9]
sudo truncate -s 0 /var/log/syslog      # truncate a large log
```

**Step 3 — Clear package cache**

```bash
sudo apt clean          # Debian/Ubuntu
sudo yum clean all      # RHEL/CentOS
```

**Step 4 — Check Docker artifacts**

```bash
docker system df        # See what's taking space
docker system prune -a  # Remove unused containers/images
```
> ⚠️ Prune removes *unused* images and volumes — be cautious on production systems.

**Step 5 — Move or archive data**
- Archive old logs to `/home` or S3
- Use `logrotate` to compress and limit logs: `sudo nano /etc/logrotate.conf`

**Step 6 — Set up alerts and monitoring**
- Install `ncdu`, `duf`, or set up Prometheus/Grafana alerts for disk thresholds
- Automate cleanup with cron or systemd timers

**🔹 Why `/var` Fills Up**
- Verbose logging (failed cron jobs, app debug logs)
- Docker images/layers
- Orphaned cache files
- Email spools or crash dumps

> **Summary:** Quickly inspect, clean, and automate monitoring. Ensure critical services like journald, docker, and package managers are not starved of space.

<br>

---

### Q4. Linux server is slow due to high CPU utilization. How will you fix it?

#### 📌 Short Explanation
This assesses your ability to diagnose performance issues, identify root causes, and reduce CPU load on a production server.

#### ✅ Answer
Identify which processes consume the most CPU using `top`, `htop`, or `pidstat`, analyze whether it's a misbehaving application, runaway process, or scheduled job, then take corrective action — kill the process, adjust resource limits, or scale the workload.

#### 📖 Detailed Explanation

**Step 1 — Check load average**

```bash
uptime
```
```text
14:02:03 up 3 days, 4:55, 2 users, load average: 6.02, 4.33, 2.89
```
A load average consistently higher than the number of CPU cores indicates over-utilization.

**Step 2 — Identify CPU-heavy processes**

```bash
top -o %CPU
htop
```

**Step 3 — Drill down with `ps` or `pidstat`**

```bash
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head
pidstat -u 1 5
```

**Step 4 — Investigate the cause**
- Is it a specific app (Java, Python, Node.js)?
- A cron job or batch script?
- A misconfigured service stuck in a loop?
- A known bug (e.g., zombie processes)?

**Step 5 — Take corrective action**

```bash
kill -9 <pid>
systemctl restart <service>
```
- Scale the application or move workloads
- Limit usage with `nice`, `cpulimit`, or cgroups
- Tune app performance (DB queries, memory leaks)

**Step 6 — Check logs**

```bash
journalctl -xe
tail -f /var/log/syslog
```
Look for app crashes, high retry loops, configuration issues.

**Step 7 — Preventive measures**
- Set CPU/memory limits in containerized apps
- Use Prometheus + Grafana
- Alert on high CPU (e.g., >80% for 5 mins)
- Refactor long-running or expensive tasks

**🔹 Real-Life Examples**
- A cron script looping due to a bad condition
- A Java app stuck in infinite recursion
- Docker containers running unbounded scraping jobs
- Antivirus/audit daemon consuming CPU after log floods

> **Summary:** Use `top`, `htop`, `ps`, and `pidstat` to identify heavy processes. Fix the root cause and add monitoring.

<br>

---

### Q5. Application deployed on NGINX returns "Connection Refused". How will you fix it?

#### 📌 Short Explanation
This evaluates your ability to troubleshoot reverse proxy setups and network-level issues. "Connection Refused" often means NGINX or the upstream service isn't reachable at all.

#### ✅ Answer
First check whether NGINX is running and listening on the correct port, then verify the backend application is up and accessible. It could be an NGINX misconfiguration or the app not listening on the expected socket/port.

#### 📖 Detailed Explanation

**Step 1 — Reproduce the error**

```bash
curl -I http://localhost
# curl: (7) Failed to connect to localhost port 80: Connection refused
```
This confirms the server refused the TCP handshake — not a 4xx/5xx error.

**Step 2 — Check if NGINX is running**

```bash
sudo systemctl status nginx
sudo systemctl restart nginx
sudo journalctl -u nginx -xe
```

**Step 3 — Is NGINX listening on the expected port?**

```bash
sudo netstat -tulnp | grep nginx
ss -tuln | grep :80
```
No output? NGINX isn't listening on that port.

**Step 4 — Check NGINX configuration**

```bash
sudo nginx -t
```
```nginx
server {
    listen 80;
    location / {
        proxy_pass http://localhost:5000;   # Is your app running here?
    }
}
```

**Step 5 — Verify the application backend**

```bash
sudo netstat -tulnp | grep 5000
curl http://localhost:5000
```

**Step 6 — Check firewall / security groups**

```bash
sudo ufw status
sudo iptables -L
```
Verify AWS Security Groups, GCP firewall rules, ufw/iptables.

**Step 7 — SELinux / AppArmor restrictions**

```bash
sudo getenforce
```
If `Enforcing`, update policies or temporarily disable for testing.

**🔹 Common Real-Life Causes**
- App crashed or not listening on the correct port
- Wrong `proxy_pass` value
- Port blocked by firewall
- NGINX not restarted after config change
- App takes too long to start — NGINX proxy fails

> **Summary:** Check NGINX status, verify backend availability, inspect configs, and ensure network rules allow traffic.

<br>

---

### Q6. SSH to an instance stopped working. How will you troubleshoot the issue?

#### 📌 Short Explanation
This checks how well you can debug access issues to a Linux server, particularly EC2 instances or cloud VMs. SSH failures can be network, OS-level, key-based, or IAM-related.

#### ✅ Answer
Follow a step-by-step process to determine whether the issue is networking (security group), the instance itself (crashed SSH service), or credentials (PEM file/key mismatch) — then apply the matching fix.

#### 📖 Detailed Explanation

**Step 1 — Confirm the error message**

```bash
ssh -i my-key.pem ec2-user@<instance-public-ip>
```
Typical errors: `Permission denied (publickey)` · `Connection refused` · `Operation timed out`

**Step 2 — Check instance health & reachability**

```bash
aws ec2 describe-instance-status --instance-id <id>
ping <public-ip>
```
If unreachable → investigate VPC/subnet/NACL routing.

**Step 3 — Verify security group rules**

```text
Type: SSH | Port: 22 | Source: your IP (e.g., 203.0.113.0/32)
```
If using a bastion host, check its connectivity too.

**Step 4 — Check Network ACLs & route tables**
Ensure NACLs aren't blocking traffic and the public subnet has a route to the internet gateway.

**Step 5 — Confirm public IP / Elastic IP**
Elastic IPs don't change; public IPs do if the instance is stopped and started.

**Step 6 — Validate PEM file & user**
- PEM file correct (`chmod 400`)
- Right username:

| OS | Username |
|----|----------|
| Amazon Linux | `ec2-user` |
| Ubuntu | `ubuntu` |
| CentOS | `centos` |

**Step 7 — Try EC2 Instance Connect**
If SSH doesn't work, use EC2 Instance Connect via the AWS Console, then check:

```bash
sudo systemctl status sshd
tail -n 50 /var/log/auth.log     # or /var/log/secure
```

**Step 8 — Rescue mode (advanced)**
Stop the instance → detach root volume → attach to another instance → mount and edit `~/.ssh/authorized_keys` or repair config → reattach and start.

**🔹 Real-Life Causes I've Faced**
- Wrong key pair name used
- Security group accidentally updated
- Wrong username (e.g., `root`)
- Instance rebooted with a new IP, old DNS cached
- `/etc/ssh/sshd_config` edited incorrectly

> **Summary:** Identify error → check network reachability → inspect key/user mismatch → use EC2 Connect if possible → rescue via EBS if needed.

<br>

---

### Q7. How do you find and list the log files older than 7 days in `/var/log`?

#### 📌 Short Explanation
Tests your comfort with Linux file management and log housekeeping — a common DevOps/sysadmin task.

#### ✅ Answer

```bash
find /var/log -type f -mtime +7
```

#### 📖 Detailed Explanation

| Part | Meaning |
|------|---------|
| `find` | Search for files in a directory hierarchy |
| `/var/log` | Target directory |
| `-type f` | Files only (not directories) |
| `-mtime +7` | Modified **more than** 7 days ago (`-7` = newer than 7 days) |

**With size and timestamp:**

```bash
find /var/log -type f -mtime +7 -exec ls -lh {} \;
```

**To delete those files:**

```bash
sudo find /var/log -type f -mtime +7 -delete
```
> ⚠️ Be careful with deletion — review the list first.

<br>

---

### Q8. How do you find and remove log files older than 30 days using `-exec`?

#### 📌 Short Explanation
Evaluates your comfort with `find -exec`, useful when you want to take specific actions on matched files (logging, conditional deletion).

#### ✅ Answer

```bash
sudo find /path/to/folder -type f -name "*.log" -mtime +30 -exec rm -f {} \;
```

#### 📖 Detailed Explanation

| Part | Meaning |
|------|---------|
| `sudo` | Needed if the directory (like `/var/log`) requires root |
| `-type f` | Match only files |
| `-name "*.log"` | Files ending with `.log` |
| `-mtime +30` | Modified over 30 days ago |
| `-exec rm -f {} \;` | Run `rm -f` on each match (`{}` = current file, `\;` ends the command) |

**Example:**

```bash
sudo find /var/log -type f -name "*.log" -mtime +30 -exec rm -f {} \;
```

**💡 Dry run before delete:**

```bash
find /var/log -type f -name "*.log" -mtime +30 -exec ls -lh {} \;
```

> **Summary:** `-exec` gives fine-grained control — useful beyond simple deletion, e.g., archiving or compressing matched files.

<br>

---

### Q9. Write a shell script that compresses logs older than 7 days and deletes logs older than 30 days. Run it daily via cron.

#### 📌 Short Explanation
Your application generates large logs in `/var/log/myapp/` with no log rotation. This tests your ability to manage disk space with compression and retention — and automate it safely with cron.

#### ✅ Answer — `log_cleanup.sh`

```bash
#!/bin/bash

# Directory where logs are stored
LOG_DIR="/var/log/myapp"
LOG_FILE="/var/log/myapp/log_rotation.log"

# Ensure the log directory exists
if [ ! -d "$LOG_DIR" ]; then
  echo "[$(date)] ERROR: Log directory $LOG_DIR does not exist!" >> "$LOG_FILE"
  exit 1
fi

# Compress logs older than 7 days (but newer than 30)
find "$LOG_DIR" -type f -name "*.log" -mtime +7 -mtime -30 ! -name "*.gz" \
  -exec gzip {} \; -exec echo "[$(date)] Compressed: {}" >> "$LOG_FILE" \;

# Delete compressed logs older than 30 days
find "$LOG_DIR" -type f -name "*.gz" -mtime +30 \
  -exec rm -f {} \; -exec echo "[$(date)] Deleted: {}" >> "$LOG_FILE" \;

# Optional: Delete uncompressed logs older than 30 days
find "$LOG_DIR" -type f -name "*.log" -mtime +30 \
  -exec rm -f {} \; -exec echo "[$(date)] Deleted (uncompressed): {}" >> "$LOG_FILE" \;

# Done
echo "[$(date)] Log rotation completed successfully." >> "$LOG_FILE"
```

#### ⏰ Schedule it daily with cron

```bash
chmod +x /path/to/log_cleanup.sh
crontab -e
```
```text
0 2 * * * /path/to/log_cleanup.sh
```

<br>

---

### Q10. Write a shell script to read a CSV of usernames/passwords and create Linux users, forcing a password change on first login.

#### 📌 Short Explanation
Tests your ability to read structured data (CSV) and automate user management — commonly needed for onboarding automation.

#### 📄 Input — `users.csv`

```csv
username,password
alice,Password@123
bob,Secure@456
carol,DevOps@789
```

#### ✅ Answer — `create_users.sh`

```bash
#!/bin/bash

INPUT="users.csv"

# Check if the file exists
if [[ ! -f "$INPUT" ]]; then
  echo "CSV file not found!"
  exit 1
fi

# Skip header and read each line
tail -n +2 "$INPUT" | while IFS=',' read -r username password; do

  # Check if user already exists
  if id "$username" &>/dev/null; then
    echo "User '$username' already exists. Skipping..."
    continue
  fi

  # Create the user
  useradd "$username"

  # Set the password
  echo "${username}:${password}" | chpasswd

  # Force password change on first login
  chage -d 0 "$username"

  echo "User '$username' created successfully."
done
```

#### ▶️ Usage

```bash
chmod +x create_users.sh
sudo ./create_users.sh
```

#### 📖 How It Works
- `IFS=',' read -r ...` parses each line into username and password
- `useradd` creates the user
- `chpasswd` sets the password via `echo "$username:$password"`
- `chage -d 0` forces a password reset at first login

<br>

---

### Q11. Write a shell script that monitors services (nginx, sshd, docker), restarts stopped ones, and prints a formatted report.

#### 📌 Short Explanation
Tests your ability to write robust service-monitoring automation for multiple services — a common DevOps/SRE expectation.

#### ✅ Answer — `multi_service_monitor.sh`

```bash
#!/bin/bash

# List of services to monitor
services=("nginx" "sshd" "docker")

# Report Header
echo "-----------------------------------"
echo "   Service Health Check Report"
echo "-----------------------------------"

# Loop through services
for service in "${services[@]}"; do
  if systemctl is-active --quiet "$service"; then
    echo "$service is ✅ RUNNING"
  else
    echo "$service is ❌ STOPPED"
    echo ""
    echo "Attempting to restart $service..."
    systemctl restart "$service" &> /dev/null

    # Check if restart was successful
    if systemctl is-active --quiet "$service"; then
      echo "$service has been ✅ restarted successfully."
    else
      echo "⚠️ Failed to restart $service. Manual intervention needed."
    fi
  fi
  echo "-----------------------------------"
done
```

#### 🖥️ Example Output (if `docker` is down)

```text
-----------------------------------
   Service Health Check Report
-----------------------------------
nginx is ✅ RUNNING
-----------------------------------
sshd is ✅ RUNNING
-----------------------------------
docker is ❌ STOPPED

Attempting to restart docker...
docker has been ✅ restarted successfully.
-----------------------------------
```

#### ⏰ Optional Cron Usage (every 10 minutes)

```text
*/10 * * * * /path/to/multi_service_monitor.sh >> /var/log/service_check.log 2>&1
```

> **Summary:** A lightweight way to keep essential services alive without external monitoring tools.

<br>

---

### Q12. How do you find and delete files larger than 100MB from a given directory?

#### 📌 Short Explanation
Tests your ability to manage disk space and clean up large files using command-line tools.

#### ✅ Answer

```bash
find /path/to/directory -type f -size +100M -exec rm -f {} \;
```

#### 📖 Detailed Explanation

| Part | Meaning |
|------|---------|
| `-type f` | Files only |
| `-size +100M` | Larger than 100 megabytes |
| `-exec rm -f {} \;` | Delete each matched file |

**Preview without deleting (dry run):**

```bash
find /path/to/directory -type f -size +100M -exec ls -lh {} \;
```

**🔹 Best Practices**
- Always dry-run before deleting anything in production
- Consider logging deletions or compressing instead of deleting
- Automate safely with cron for specific paths, e.g., `/tmp` or `/var/cache`

<br>

---

### Q13. How do you get the list of users who logged into the system today?

#### 📌 Short Explanation
Tests your understanding of Linux log inspection and user activity tracking — useful for auditing, troubleshooting, and monitoring.

#### ✅ Answer

```bash
last | grep "$(date '+%a %b %e')" | awk '{print $1}' | sort | uniq
```

#### 📖 Detailed Explanation

| Part | Meaning |
|------|---------|
| `last` | Displays recent login history from `/var/log/wtmp` |
| `date '+%a %b %e'` | Today's date in the format `last` uses (e.g., `Thu Jun 13`) |
| `grep "$(date ...)"` | Filters only today's entries |
| `awk '{print $1}'` | Extracts usernames |
| `sort \| uniq` | Unique users only |

**Example output:**

```text
ubuntu
admin
deploy
```

**💡 Bonus Tip** — if `wtmp` logs are rotated or missing:

```bash
ls -lh /var/log/wtmp
journalctl --since today | grep 'session opened'
```

<br>

---

### Q14. Your website is not loading. Describe the step-by-step investigation process.

#### 📌 Short Explanation
A high-pressure but common scenario testing your ability to troubleshoot full-stack issues — from DNS and networking to web server and app code.

#### ✅ Answer
Start from external checks and move inward, layer by layer.

#### 📖 Detailed Explanation

**1️⃣ Is the site down for everyone or just me?**
```bash
curl -I https://yourdomain.com
ping yourdomain.com
```
Or use [downforeveryoneorjustme.com](https://downforeveryoneorjustme.com).

**2️⃣ DNS resolution**
```bash
dig yourdomain.com
nslookup yourdomain.com
```
No IP? Check DNS settings in Route53 or your DNS provider.

**3️⃣ Is the domain routing to the correct server?**
```bash
curl -v https://yourdomain.com
```
Compare with the server IP. If misrouted, verify DNS records, load balancer config, or CDN rules.

**4️⃣ Network / firewall check**
```bash
telnet yourdomain.com 443
nc -zv yourdomain.com 80
```
Check security groups, firewalls, NACLs.

**5️⃣ Is the web server running?**
```bash
sudo systemctl status nginx
sudo systemctl status apache2
sudo systemctl restart nginx
```

**6️⃣ Check application logs**
- `/var/log/nginx/error.log`
- `/var/log/httpd/error_log`
- App logs: `app.log`, stderr

**7️⃣ Check disk / memory / CPU**
```bash
df -h
top      # or htop
free -m
```

**8️⃣ Check backend services (DB, cache, etc.)**
- MySQL/Postgres down
- Redis/Memcached connection error
- App server crashes

**9️⃣ SSL certificate issues**
```bash
curl -Iv https://yourdomain.com
```
Look for `SSL certificate problem`. If expired, renew via Let's Encrypt or your CA.

**🔟 Rollback or revert**
If the issue started after a deploy:
```bash
kubectl rollout undo deployment your-deployment
```
or redeploy the old version via CI/CD.

**💡 Bonus Tips**
- Always check uptime monitoring, alerting tools, or dashboards
- Build a runbook for repeated scenarios

> **Summary:** Think in layers — **DNS → Network → Web Server → Application → Infrastructure → Dependencies.**

<br>

---

### Q15. How do you remove the first and last line of a file using `sed`?

#### 📌 Short Explanation
Checks your understanding of line-addressing in `sed`, a powerful stream editor for text manipulation in shell scripting.

#### ✅ Answer

```bash
sed '1d; $d' filename.txt
```

#### 📖 Detailed Explanation

| Part | Meaning |
|------|---------|
| `sed` | Stream editor — processes text line by line |
| `'1d'` | Delete the first line |
| `'$d'` | Delete the last line (`$` = end of file) |

**Example** — if `file.txt` contains:

```text
Line 1
Line 2
Line 3
Line 4
Line 5
```

```bash
sed '1d; $d' file.txt
```

Output:

```text
Line 2
Line 3
Line 4
```

**💡 Save to a new file:**

```bash
sed '1d; $d' file.txt > trimmed.txt
```

<br>

---

<br>

<a id="networking"></a>
# 🌐 TOPIC 3: NETWORKING

> DNS, OSI model, proxies, HTTP errors, curl/DNS troubleshooting, special IPs and VPC subnets.

<br>

---

### Q1. What is DNS? Explain in simple words.

#### 📌 Short Explanation
DNS is like the internet's phonebook. It translates website names into IP addresses that computers use to talk to each other.

#### ✅ Answer
**DNS** stands for **Domain Name System**. When you type `www.google.com` into your browser, your computer doesn't understand the name directly. It asks the DNS system for the matching IP address (like `142.250.64.100`). Once it has the IP, your computer connects to the right server and loads the website.

#### 📖 Detailed Explanation

| Term | Meaning |
|------|---------|
| **Domain name** | Human-friendly name like `amazon.com` |
| **IP address** | Computer-friendly address like `192.0.2.1` |
| **DNS Resolver** | The system that looks up domain names and returns IP addresses |

**Step-by-step process:**
1. You enter `www.example.com` in your browser.
2. Your computer asks the DNS resolver for the IP address.
3. The resolver checks its cache or queries other DNS servers.
4. Once it finds the matching IP, it returns it to your browser.
5. Your browser connects to the IP and loads the website.

**🍕 Example:** Think of DNS like asking for a restaurant's phone number:
- You say: *"I want to call Domino's Pizza."*
- DNS replies: *"Here's the number: 123-456-7890."*
- Now you can call and place your order.

> **Summary:** DNS lets us use easy names like `google.com` instead of hard-to-remember IP addresses.

<br>

---

### Q2. Explain the complete flow of a request from client to server using the OSI Model.

#### 📌 Short Explanation
Checks if you understand how a network request travels across layers — from user input in a browser to a remote server — using the 7 layers of the OSI model.

#### ✅ Answer
The OSI model has 7 layers, and each plays a specific role in transferring data from a client (browser) to a server. Here's what happens when you type `https://example.com` and hit Enter:

#### 📖 Detailed Explanation

| Layer | Name | What happens | Examples |
|-------|------|--------------|----------|
| **7** | Application | Browser creates an HTTP request | HTTP, HTTPS, DNS, FTP, SMTP |
| **6** | Presentation | Data formatting, encryption (SSL/TLS), compression | HTTPS encryption starts here |
| **5** | Session | Opens, maintains, and tears down sessions | Session resume / timeout |
| **4** | Transport | Breaks data into segments, adds ports, ensures delivery | TCP (reliable), UDP (fast); HTTP→80, HTTPS→443 |
| **3** | Network | Adds source/destination IP, chooses the route | IP |
| **2** | Data Link | Converts to frames, adds MAC addresses | Ethernet / Wi-Fi |
| **1** | Physical | Transfers raw bits over hardware | Cables, radio waves, fiber |

**🔹 Then What Happens?**
- At the server side, data flows **upward** from Layer 1 to Layer 7.
- Each layer removes the headers added by the client and processes the payload.
- The server responds, and the response travels back through the same layers in reverse.

**✉️ Analogy — Sending a letter:**
- You write a message *(Application)*
- Put it in an envelope *(Presentation)*
- Mark sender/recipient *(Session)*
- Choose a postal service *(Transport)*
- Address it *(Network)*
- The postman routes it *(Data Link)*
- It travels by trucks/planes *(Physical)*

> **Summary:** The OSI model breaks the process into layers so we can troubleshoot and build systems more effectively.

<br>

---

### Q3. Explain the difference between a Forward Proxy and a Reverse Proxy.

#### 📌 Short Explanation
Both proxies act as intermediaries in network communication, but they sit at different ends of the request flow and serve different purposes.

#### ✅ Answer

| Aspect | Forward Proxy | Reverse Proxy |
|--------|---------------|---------------|
| **Position** | Between **client** and the internet | Between **internet** and the **server** |
| **Represents** | Acts on behalf of the **client** | Acts on behalf of the **server** |
| **Use Cases** | Anonymize clients · Bypass firewalls · Control access (parental control) · Cache outgoing requests | Load balancing · SSL termination · Cache incoming responses · Protect internal servers |
| **Example** | Client → Forward Proxy → Google | User → Reverse Proxy → Internal Web Server |

#### 📖 Detailed Explanation

**🔹 Forward Proxy**
- Used by clients to access external servers.
- The target server only sees the proxy, not the real client.
- Often used in corporate environments or VPNs to filter or restrict internet access.

> 🧳 *Like a travel agent booking your ticket on your behalf — the airline doesn't know who you are.*

**🔹 Reverse Proxy**
- Sits in front of a group of servers and routes client requests to them.
- The client thinks it's talking directly to the server, but it's talking to the proxy.
- Used for load balancing, security, and SSL offloading.

> 🛎️ *Like a receptionist at an office — you talk to them, and they connect you to the right person inside.*

**🔹 Example Scenarios**
- **Forward Proxy:** A school uses a proxy server to prevent students from accessing YouTube.
- **Reverse Proxy:** A company uses NGINX to distribute requests to multiple backend services (`/api`, `/auth`, `/app`).

> **Summary:** A Forward Proxy serves the client and hides them from the server. A Reverse Proxy serves the server and hides it from the client.

<br>

---

### Q4. A user reports that the application is slow. How would you troubleshoot and identify the root cause?

#### 📌 Short Explanation
Tests your ability to troubleshoot performance issues across the full stack — frontend, backend, database, infrastructure, and network.

#### ✅ Answer — Step-by-Step Investigation Approach

**1️⃣ Clarify the scope**
- One user or many? Specific pages/actions/times of day? Which environment?
- *Narrows down whether it's user-specific, global, or intermittent.*

**2️⃣ Check frontend first**
- Browser dev tools (Network, Performance tabs): slow JavaScript? Large images/API calls? High Time to First Byte (TTFB)?
- *If TTFB is high, backend or infra may be the bottleneck.*

**3️⃣ Backend API performance**
- Check response times via APM tools (New Relic, Datadog, Prometheus)
- Identify slow endpoints, latency spikes

**4️⃣ Database slowness**
- Slow queries or locking issues? Use `EXPLAIN` to optimize
- Monitor CPU and disk I/O on the DB server
- Check for missing indexes

**5️⃣ Infrastructure & resource usage**
```bash
top, htop, vmstat, iostat
```
- Check container/pod resource limits (Kubernetes)
- Scale up if near limits (AutoScaling, HPA)

**6️⃣ Monitor logs & alerts**
- Check app and server logs for errors/latency
- Look for recent deployments that correlate with slowness

**7️⃣ Caching & CDN checks**
- Cache missed or expiring too often? CDN serving static content properly?

**8️⃣ Network or DNS latency**
- `ping`, `traceroute`, `mtr`; check DNS lookup times; consider edge latency for global users

**9️⃣ Rollbacks or restarts**
- If slowness began after a release: roll back; restart degraded pods/services

**🔟 After fix: monitor & prevent**
- Add performance alerts (latency, CPU, DB)
- Set SLOs for key endpoints
- Add automated profiling for slow endpoints

> **Summary:** Use a systematic layer-by-layer approach. Focus first on scope, then verify each component with logs, metrics, and tools.

<br>

---

### Q5. `curl` works with an IP address but fails with the domain name. Why, and how do you troubleshoot?

#### 📌 Short Explanation
This points to a **DNS resolution problem** — your system can reach the server IP directly but cannot translate the domain name into an IP.

#### ✅ Answer
If `curl http://<IP>` works but `curl http://example.com` fails, the issue is most likely DNS-related.

#### 📖 Detailed Explanation — Common Causes

**1. DNS not resolving**
```bash
nslookup example.com
dig example.com
```
If these fail, DNS is the root issue.

**2. Wrong or missing DNS configuration**
- Check `/etc/resolv.conf` for a valid nameserver (e.g., `8.8.8.8`).

**3. Firewall or network blocking DNS**
- Port 53 may be blocked. Test with:
```bash
dig example.com @8.8.8.8
```

**4. Domain doesn't exist or typo**
```bash
whois example.com
```

**5. Hosts file override**
```bash
cat /etc/hosts
```
Remove or correct conflicting lines.

**6. Internal DNS only**
- Internal domains (e.g., `myapp.internal.local`) won't resolve externally — connect to the VPN/internal network.

**🔹 How to Fix**

```bash
# Add a DNS server like Google DNS
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null

# Or test temporarily with --resolve
curl --resolve example.com:80:<IP> http://example.com
```

> **Summary:** When curl works with an IP but fails with a domain, it's almost always DNS. Use `dig`, `nslookup`, and check `/etc/resolv.conf` or `/etc/hosts`.

<br>

---

### Q6. Your website is returning a `502 Bad Gateway`. What does it mean, and how do you resolve it?

#### 📌 Short Explanation
A **502 Bad Gateway** means a gateway or proxy (NGINX, HAProxy, AWS ELB) got an invalid response from the upstream server (app server or container).

#### ✅ Answer
A 502 is returned when the reverse proxy or load balancer is unable to reach the backend service or gets a malformed response.

> *"The gatekeeper tried to contact the backend service — but something was wrong or unreachable."*

#### 📖 Detailed Explanation — Common Root Causes

**1️⃣ Backend service is down**
```bash
systemctl status your-app
```

**2️⃣ Wrong upstream configuration in NGINX**
```nginx
proxy_pass http://localhost:5000;
```

**3️⃣ Backend is too slow / times out**
```nginx
proxy_read_timeout 60s;
```

**4️⃣ Firewall or security group blocking**
```bash
nc -zv localhost 5000
```

**5️⃣ Incorrect SSL termination**
- NGINX expects HTTP but backend speaks HTTPS (or vice versa) — fix `proxy_pass` protocol.

**6️⃣ App crashed or out of memory**
```bash
journalctl -u your-app
docker logs your-container
```

**🔹 How to Troubleshoot**

```bash
# 1. Check NGINX error logs
tail -f /var/log/nginx/error.log

# 2. Restart app & NGINX
systemctl restart your-app
systemctl restart nginx

# 3. Check app health endpoint
curl http://localhost:5000/health
```

**🔹 Bonus: Simulate a 502 for testing**

```bash
sudo systemctl stop your-app
```
Reload the site — you'll get a 502.

> **Summary:** Check service status, logs, port connectivity, timeouts, and config mismatches.

<br>

---

### Q7. What is the difference between `0.0.0.0` and `127.0.0.1`?

#### 📌 Short Explanation
Both are special IP addresses, but they serve very different purposes.

#### ✅ Answer

| Address | Meaning | Use Case |
|---------|---------|----------|
| `127.0.0.1` | Loopback address (localhost) | Your computer talks to itself |
| `0.0.0.0` | All IPv4 addresses on the local machine | Servers listen on **all interfaces** |

#### 📖 Detailed Explanation

**🔹 `127.0.0.1` — Loopback (localhost)**
- Refers to your own computer; used for testing, development, inter-process communication.
- Traffic never leaves your machine.

```bash
curl http://127.0.0.1:8080
```

**🔹 `0.0.0.0` — All interfaces (wildcard)**
- Means "listen on all network interfaces".
- Commonly used by servers to accept traffic from any IP address.
- Not routable — you won't `curl 0.0.0.0`, but services bind to it.

```bash
python3 -m http.server --bind 0.0.0.0
```
→ Makes the web server available to other devices on the network.

**🔹 Analogy**
- `127.0.0.1`: *"I'm talking to myself only."*
- `0.0.0.0`: *"I'm open to talk to anyone who connects to me."*

> **Summary:** Use `127.0.0.1` to keep traffic inside your machine. Use `0.0.0.0` when your service must accept traffic from anyone, on any IP your machine has.

<br>

---

### Q8. What is the difference between Public and Private Subnets?

#### 📌 Short Explanation
The key difference lies in internet accessibility:
- **Public subnets** can directly communicate with the internet.
- **Private subnets** cannot, unless they go through a NAT Gateway or similar service.

#### ✅ Answer

| Type | Internet Access | Use Case | Route Table |
|------|-----------------|----------|-------------|
| **Public Subnet** | Yes (via Internet Gateway) | Load balancers, bastion hosts, public APIs | Route to Internet Gateway (IGW) |
| **Private Subnet** | No direct access | Databases, app servers, internal services | No direct IGW route; NAT Gateway optional |

#### 📖 Detailed Explanation

**🔹 Public Subnet**
- Has a route to an Internet Gateway (IGW).
- EC2 instances are reachable from the internet if they have public IPs.
- Common use: web servers, bastion hosts, NAT Gateways.

```text
Destination     Target
0.0.0.0/0       igw-xxxxxxxx
```

**🔹 Private Subnet**
- No direct route to the internet; instances can't be reached directly from the internet.
- Outbound access (e.g., installing packages) goes via a NAT Gateway placed in a public subnet.

```text
Destination     Target
0.0.0.0/0       nat-xxxxxxxx
```

**🔹 Analogy**
- **Public Subnet:** A house with a door that opens directly to the street.
- **Private Subnet:** A room in a gated community — you can go out, but only through controlled paths (NAT).

> **Summary:** Public subnet → internet access via IGW. Private subnet → no direct internet; used for backend services, databases, and sensitive components.

<br>

---

### Q9. You accidentally created a private subnet instead of a public one. How do you fix it?

#### 📌 Short Explanation
A subnet becomes public when it has a route to an Internet Gateway (IGW) **and** instances in it are assigned public IPs. Fixing it involves updating the route table and IP assignment settings.

#### ✅ Answer — Step-by-Step Solution

**1️⃣ Update the route table**
- VPC → Route Tables → identify the subnet's route table → add:

```text
Destination: 0.0.0.0/0
Target:      igw-xxxxxxxx     # Your Internet Gateway ID
```

**2️⃣ Associate the correct route table**
- Route Tables → select it → **Subnet Associations** tab → ensure your subnet is associated.

**3️⃣ Enable auto-assign public IPs**
- VPC → Subnets → [Your Subnet] → Actions → **Modify auto-assign IP settings** → check *Auto-assign IPv4 public IP*.

**4️⃣ Assign a public IP to existing EC2 instances**
- If launched without a public IP: allocate an **Elastic IP**, then attach it to the instance's primary network interface (EC2 → Network Interfaces).

**5️⃣ Ensure the IGW is attached to the VPC**
- VPC → Internet Gateways → confirm attachment to the same VPC.

#### 📖 Example Recap
You deployed a web server (Apache/NGINX) in the subnet and expected it to be publicly accessible, but it wasn't reachable because the subnet lacked:
- A route to the Internet Gateway
- A public IP for the EC2 instance

Adding the missing IGW route and assigning a public IP solved the issue.

> **Summary:** A public subnet needs a route to the internet via an IGW, and EC2 instances need public IPs.

<br>

---

<br>

<a id="cicd"></a>
# ⚙️ TOPIC 4: CI/CD

> Jenkins, Maven, Artifactory, pipeline troubleshooting, Python builds, static code analysis, Argo CD and notifications.

<br>

---

### Q1. What are Jenkins Shared Libraries and how do they work?

#### 📌 Short Explanation
Jenkins Shared Libraries centralize reusable pipeline code (functions, steps, variables) across multiple pipelines. They promote code reuse, maintainability, and consistency in large Jenkins setups.

#### ✅ Answer
A Shared Library is a Git repository (or part of one) containing reusable Groovy code you can include in Jenkins pipelines using the `@Library` annotation.

```text
(root)
├── vars/
│   └── sayHello.groovy
├── src/
│   └── org/example/MyClass.groovy
├── resources/
│   └── templates/config.xml
└── README.md
```

#### 📖 Detailed Explanation

**🔹 Why use Shared Libraries?**
- Avoid repeating logic in every Jenkinsfile
- Encapsulate business logic, deployment steps, or validation code
- Easy updates across all pipelines
- Fewer errors and better collaboration

**🔹 How do they work?**
1. Create a Git repo with a specific structure (`vars/`, `src/`, etc.).
2. Configure the library in Jenkins: **Manage Jenkins → Global Pipeline Libraries** → add by name and Git URL.
3. Load it in your Jenkinsfile: `@Library('my-shared-library') _`
4. Use global functions or classes defined in `vars/` or `src/`.

**🔹 Example**

`vars/sayHello.groovy`
```groovy
def call(String name = 'world') {
    echo "Hello, ${name}!"
}
```

`Jenkinsfile`
```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Greet') {
            steps {
                sayHello('Abhishek')
            }
        }
    }
}
```

**🔹 Benefits**
- DRY (Don't Repeat Yourself)
- Cleaner Jenkinsfiles
- Version-controlled and auditable
- Easier team collaboration

> **Summary:** Shared Libraries modularize and reuse pipeline logic — ideal for large-scale CI/CD where consistency and maintainability matter.

<br>

---

### Q2. Talk about 5 build targets that you use on a day-to-day basis in Maven.

#### 📌 Short Explanation
Maven uses build lifecycle phases (build targets) to automate compilation, packaging, testing, and deployment, executed via `mvn <goal>`.

#### ✅ Answer

| # | Command | Purpose | When I use it |
|---|---------|---------|---------------|
| 1 | `mvn clean` | Deletes the `target/` directory for a clean slate | Before any fresh build, to avoid stale artifacts |
| 2 | `mvn compile` | Compiles source code in `src/main/java` | To verify there are no compilation issues |
| 3 | `mvn test` | Runs unit tests (JUnit / TestNG) | Regularly during development and in CI |
| 4 | `mvn package` | Packages compiled code into `.jar` / `.war` | When the build is stable and I need an artifact |
| 5 | `mvn install` | Installs the artifact into the local repo (`~/.m2/repository`) | When developing shared libraries reused by other local projects |

> **Summary:** `clean`, `compile`, `test`, `package`, and `install` ensure a complete, reliable build pipeline from development to artifact distribution.

<br>

---

### Q3. Which artifact repository do you use for builds?

#### 📌 Short Explanation
An artifact repository stores, publishes, and retrieves build artifacts (`.jar`, `.war`, Docker images, etc.) — crucial for dependency management and versioning in CI/CD.

#### ✅ Answer
We use **JFrog Artifactory** as our primary artifact repository.

#### 📖 Detailed Explanation

**🔹 Why JFrog Artifactory?**
- Supports multiple package formats (Maven, npm, Docker, PyPI, Helm, etc.)
- Integrates with Jenkins & GitHub Actions for publishing artifacts
- Proxies public repositories (Maven Central, Docker Hub) for speed and reliability
- RBAC for access control across teams/projects
- Retention policies to clean up outdated builds automatically

**🔹 Typical Workflow**
1. Developer commits code to GitHub.
2. Jenkins triggers a build and packages the app using Maven.
3. The artifact is pushed to Artifactory using `mvn deploy`.
4. Later stages (deployment) pull the artifact from Artifactory.

**🔹 Alternatives I've worked with**
- **Sonatype Nexus Repository** — lightweight, easy to set up for Maven-only use cases
- **AWS CodeArtifact** — great for AWS-native environments
- **GitHub Packages** — useful for open-source or tight GitHub integration

<br>

---

### Q4. How do you configure Artifactory for your application in Maven?

#### 📌 Short Explanation
To publish or retrieve artifacts from Artifactory with Maven, configure credentials and repository endpoints in `settings.xml` and your project's `pom.xml`.

#### ✅ Answer
1. Define credentials in `settings.xml`.
2. Reference the Artifactory repository in `pom.xml`.

#### 📖 Detailed Explanation

**Step 1 — Update `settings.xml`** (located at `~/.m2/settings.xml`)

```xml
<settings>
  <servers>
    <server>
      <id>artifactory</id>
      <username>my-artifactory-user</username>
      <password>my-artifactory-password</password>
    </server>
  </servers>
</settings>
```
> 🔐 For security, use Maven's `mvn --encrypt-password` and store the encrypted password here.

**Step 2 — Configure `pom.xml`**

```xml
<distributionManagement>
  <repository>
    <id>artifactory</id>
    <name>Company Release Repo</name>
    <url>https://artifactory.example.com/artifactory/libs-release-local</url>
  </repository>
  <snapshotRepository>
    <id>artifactory</id>
    <name>Company Snapshot Repo</name>
    <url>https://artifactory.example.com/artifactory/libs-snapshot-local</url>
  </snapshotRepository>
</distributionManagement>
```
Make sure the `<id>` matches the one in `settings.xml`.

**Deploy:**

```bash
mvn clean deploy
```

**🔹 What This Achieves**
- Pulls dependencies from Artifactory (if configured in `<repositories>`)
- Pushes built artifacts (`.jar`, `.war`) to Artifactory
- Supports release and snapshot repositories

<br>

---

### Q5. Build passed locally but fails in CI — how will you troubleshoot?

#### 📌 Short Explanation
A common CI/CD issue; it usually points to differences in configuration, environment, or dependencies between the developer's machine and the CI environment.

#### ✅ Answer
Troubleshoot in a structured way, focusing on environmental differences and repeatability.

#### 📖 Step-by-Step Troubleshooting

**1️⃣ Check the error log in CI**
- Identify whether it's a compilation error, test failure, dependency issue, or permission problem.

**2️⃣ Reproduce locally in a CI-like environment**
- Use a clean Docker image or new VM with the same JDK/Node/Python versions.
```bash
docker run -it --rm openjdk:17 bash
# Then run your build commands
```

**3️⃣ Compare local and CI environments**
- OS version · language/toolchain versions · environment variables · Maven/Gradle versions · memory/CPU limits

**4️⃣ Verify secrets and tokens**
- Missing Git credentials, API tokens, `.npmrc` / `.pypirc` — ensure they're injected via CI variables or secrets management.

**5️⃣ Dependency issues**
- Local cache (`~/.m2/repository`) may hide missing deps. Compare using `mvn dependency:tree` or `npm ls`.

**6️⃣ File / path issues**
- Case sensitivity, permissions, and missing files often break builds on Linux CI runners.

**🔹 Example**
Build failed in CI with:
```text
error: package javax.annotation does not exist
```
- **Root cause:** Local machine had Java 8; CI used Java 11, which doesn't include that package by default.
- **Fix:** Added `javax.annotation` as an explicit dependency in `pom.xml` and pinned the JDK version in CI to match local.

> **Summary:** Start with CI logs → reproduce in a clean container → compare environments → check dependencies/secrets → make the build reproducible and environment-agnostic.

<br>

---

### Q6. CI pipeline succeeds but the app is broken in prod — what action will you take?

#### 📌 Short Explanation
If CI/CD passes but the app breaks in production, something is missing in the validation phase, or there are environment mismatches between staging/CI and production.

#### ✅ Answer
Treat it as a critical incident: debug, roll back, and add preventive actions.

#### 📖 Step-by-Step Troubleshooting

**1️⃣ Initiate incident response** — notify the team, document the issue; if severe, trigger rollback or redeploy the last working version.

**2️⃣ Check prod logs and monitoring** (ELK, CloudWatch, etc.)
- HTTP status codes (500s, 4xx) · application logs · memory, CPU, DB errors, timeouts

**3️⃣ Compare staging and production**
- Missing env vars, backend services, or feature flags?
- Confirm the artifact promoted to prod is the same one tested in staging.

**4️⃣ Check secrets and external integrations** — rotated tokens, misconfigured API keys, DB credentials.

**5️⃣ Check infrastructure differences** — Kubernetes namespace, load balancer config, Terraform state, older AMIs, custom security groups.

**🔹 Immediate Actions**
- Roll back if feasible (Git tags, Helm chart versions, AMI snapshots)
- Create a postmortem entry and assign RCA
- Patch with a hotfix only after RCA is complete

**🔹 Preventive Measures**
- Automated **smoke tests** after deployment
- Canary or blue-green deployments
- Enforce staging/production parity
- Validate secrets and config before each deployment
- Alert on anomalies right after deployment

**🔹 Real-World Example**
After a successful deployment, the app broke in prod because a staging-only env var (`USE_MOCK_SERVICE=true`) was hardcoded and not overridden in production.
- **Fix:** Added a proper `values-prod.yaml` for Helm, separated environments clearly, added post-deploy smoke tests.

<br>

---

### Q7. Pipeline slows down over time (builds taking more time) — how will you fix it?

#### 📌 Short Explanation
Progressively slower CI is usually caused by accumulated build artifacts, unoptimized steps, lack of caching, or resource saturation on runners/agents.

#### ✅ Answer
Take a metrics-driven approach to isolate the slowdown and optimize the pipeline stages.

#### 📖 Step-by-Step Troubleshooting

**1️⃣ Measure stage durations over time**
- Use CI tool metrics or Prometheus/Grafana; identify which stage is slowing (checkout, dependency resolution, tests, packaging).

**2️⃣ Check dependency management**
- `mvn dependency:analyze`, `npm prune`
- Cache dependencies (`~/.m2`, `node_modules`) between runs.

**3️⃣ Enable layered and incremental builds**
- Avoid cleaning the whole workspace unless necessary
- Gradle `--build-cache` · Bazel native caching · Docker layers used wisely

**4️⃣ Clean up disk and workspace**
```bash
docker system prune -af
```
Runners accumulate gigabytes of artifacts, old images, and volumes.

**5️⃣ Use parallelism and matrix builds**
- `strategy.matrix` in GitHub Actions · `parallel` stages in Jenkins

**6️⃣ Review runner/agent resource utilization**
- Check CPU, memory, disk I/O; use autoscaling runners or faster instance types.

**🔹 Real-World Example**
Build time rose from **7 to 19 minutes** over 6 months.
- **Root cause:** more tests without parallel execution; outdated Docker layers not using cache.
- **Fixes:** parallel test stages · refactored Dockerfile for better caching · weekly cleanup of unused images on runners.

<br>

---

### Q8. A developer pushes a feature branch, but the pipeline doesn't trigger in GitHub Actions. What could be wrong?

#### 📌 Short Explanation
Usually caused by misconfigured trigger rules, file path filters, or workflow scope issues.

#### ✅ Answer
Check the following areas step-by-step.

#### 📖 Step-by-Step Troubleshooting

**1️⃣ Check the `on:` section**

❌ Incorrect (only `main` triggers):
```yaml
on:
  push:
    branches:
      - main
```
✅ Fix — include feature patterns:
```yaml
on:
  push:
    branches:
      - main
      - 'feature/*'
      - 'dev'
```

**2️⃣ Check the `paths` filter**
```yaml
on:
  push:
    paths:
      - 'src/**'
```
If a developer changed files outside the listed paths, the pipeline won't trigger.

**3️⃣ Is the workflow file on the default branch?**
- Workflows in `.github/workflows/` must exist on the default branch (usually `main`) to trigger from other branches.

**4️⃣ Check branch protection or permissions**
- If Actions is disabled or workflow permissions are restricted (Settings → Actions → General), execution may be blocked.

**5️⃣ Verify the YAML file validity**
- Run `act` locally or check GitHub Actions logs — a broken workflow file can silently fail to register triggers.

**🔹 Real-World Fix**
A push to `feature/payment-refactor` didn't run the workflow because `on.push.branches` only had `main` and `develop`. Adding `'feature/*'` fixed it.

<br>

---

### Q9. Your build fails because it can't download a dependency from your artifact repository. What will you do?

#### 📌 Short Explanation
A failed dependency download usually indicates an issue with repository configuration, authentication, connectivity, or artifact availability.

#### ✅ Answer
Debug systematically (Artifactory, Nexus, AWS CodeArtifact, etc.).

#### 📖 Step-by-Step Troubleshooting

**1️⃣ Check the build error message**
- Identify the dependency and the repo URL it tried — note whether it's `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, or a timeout.

**2️⃣ Verify repository configuration** (`pom.xml`, `build.gradle`, `.npmrc`)
```xml
<repository>
  <id>company-repo</id>
  <url>https://artifactory.example.com/artifactory/libs-release</url>
</repository>
```

**3️⃣ Check credentials** (`~/.m2/settings.xml` or environment variables)
```xml
<server>
  <id>company-repo</id>
  <username>${env.ARTIFACT_USER}</username>
  <password>${env.ARTIFACT_PASS}</password>
</server>
```
> 💡 Some tokens (e.g., AWS CodeArtifact) expire and must be refreshed via CLI.

**4️⃣ Test repo connectivity**
```bash
curl -I https://artifactory.example.com/artifactory/libs-release/...
```
Check proxy/firewall/DNS issues, especially in CI environments.

**5️⃣ Check if the artifact exists** — confirm the version exists and the repository hasn't been deleted/archived.

**6️⃣ Try a local build with a clean cache**
```bash
rm -rf ~/.m2/repository/<group>/<artifact>
mvn clean install
```

**🔹 Real-World Example**
`mvn install` failed in Jenkins because the **CodeArtifact token had expired** but Jenkins still used the old token via an environment variable.
- **Fix:** added a Jenkinsfile step to refresh the token before the build:
```bash
aws codeartifact get-authorization-token ...
```

<br>

---

### Q10. Python build fails on CI but works locally — what can be the issue?

#### 📌 Short Explanation
Typically differences in environment, missing dependencies, version mismatches, or missing credentials between local and CI.

#### ✅ Answer
Systematically compare local and CI environments for Python version, packages, permissions, and environment variables.

#### 📖 Step-by-Step Troubleshooting

**1️⃣ Check Python version** — local may be 3.11 while the CI runner defaults to 3.6.
```bash
python --version
```
Fix: pin the version in CI (GitHub Actions):
```yaml
- uses: actions/setup-python@v4
  with:
    python-version: '3.10'
```

**2️⃣ Check dependency installation**
```bash
pip install -r requirements.txt
```
Version pinning differences can break CI.

**3️⃣ Check missing files/modules** — CI starts from a clean environment; ensure `.env`, `config.yaml`, and local modules are available.

**4️⃣ Check missing secrets**
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

**5️⃣ Run tests verbosely**
```bash
pytest -v
```
Helps pinpoint `ImportError`, `ModuleNotFound`, or permission issues.

**6️⃣ Check C extension issues** — packages like `psycopg2` and `lxml` need system libraries.
```bash
sudo apt-get install -y libpq-dev
pip install psycopg2
```

**🔹 Real-World Example**
A Flask app built locally but failed in GitHub Actions. The app used `python-dotenv`, but the `.env` file wasn't available in CI.
- **Fix:** added secrets in GitHub → Repository Settings → Secrets, and injected them with `env:`.

<br>

---

### Q11. Explain the Python Application Build Process in detail.

#### 📌 Short Explanation
Python apps don't go through a heavy compile step, but building still involves packaging, dependency resolution, and distribution — critical for CI/CD and production deployment.

#### ✅ Answer
The Python build process includes:
1. Organizing the project structure
2. Managing dependencies
3. Compiling to bytecode (optional)
4. Creating distributable artifacts (wheel / sdist)
5. Publishing the package (optional)

#### 📖 Detailed Python Build Process

**1️⃣ Organize the project structure**

```text
myapp/
├── myapp/               # Application source code
│   └── __init__.py
├── tests/               # Unit tests
├── pyproject.toml       # Modern metadata and build system
├── requirements.txt     # Dependency list (optional)
└── README.md
```

**2️⃣ Declare dependencies**
```bash
pip install -r requirements.txt
```

**3️⃣ Build the package**
```bash
python3 -m venv venv
source venv/bin/activate
pip install build
python -m build
```
Generates:
- `dist/myapp-0.1.0.tar.gz` (source distribution)
- `dist/myapp-0.1.0-py3-none-any.whl` (wheel binary)

**4️⃣ Run unit tests**
```bash
pytest tests/
```

**5️⃣ Distribute or deploy**
```bash
pip install twine
twine upload dist/*
```
Package into containers, VMs, or PaaS (AWS Lambda, Google Cloud Run).

**🔹 Real-World Example (Flask API)**
- Declared dependencies in `requirements.txt`
- Created `pyproject.toml` for packaging metadata
- Ran `python -m build` in CI to produce a wheel
- Built a Docker image with the wheel inside
- Deployed to Kubernetes using Helm

<br>

---

### Q12. Using Static Code Analysis, what kind of problems can you identify?

#### 📌 Short Explanation
Static code analysis detects issues in source code **without executing it** — an early gate in CI/CD to catch bugs, code smells, and standard violations.

#### ✅ Answer — Types of Problems Identified

| # | Category | Examples | Tools |
|---|----------|----------|-------|
| 1 | **Syntax errors & language misuse** | Missing imports, undeclared variables (`print(x)` where `x` isn't defined) | pylint, flake8 |
| 2 | **Coding standard violations** | PEP8, PSR-12, Google Java Style: long lines, bad indentation, poor naming | pylint, flake8, checkstyle, eslint |
| 3 | **Complexity & maintainability** | Deep nesting, too many return points, long methods | radon, sonarqube, jshint |
| 4 | **Security vulnerabilities** | Hardcoded credentials, SQL injection (`"SELECT * FROM users WHERE id = " + user_input`) | bandit, Brakeman, semgrep |
| 5 | **Dead code & unused variables** | Unused imports, unreachable code | pylint, flake8 |
| 6 | **Incorrect type usage** | Type mismatches in statically typed code | mypy |
| 7 | **Common bugs & anti-patterns** | `==` vs `===` in JS, `=` vs `==`, resource leaks (unclosed files) | eslint, sonarqube |
| 8 | **Duplicate code** | Copy-pasted blocks (violates DRY) | SonarQube, PMD, jscpd |

**🔹 Real-World Example (Python projects)**
- `bandit` detected a hardcoded AWS access key in a config file
- `flake8` flagged missing docstrings and complex nested loops
- `SonarQube` highlighted duplicated logic in two modules

These were caught before staging, saving time and preventing technical debt.

> **Summary:** Static analysis identifies bugs, security risks, code smells, and style issues early — boosting quality, maintainability, and security without running the app.

<br>

---

### Q13. Static Code Analysis slows down the CI pipeline — how will you fix it?

#### 📌 Short Explanation
When static analysis becomes a bottleneck, optimize execution by limiting scope, parallelizing checks, or moving analysis to asynchronous/pre-merge steps.

#### ✅ Answer — Step-by-Step Optimization Strategy

**1️⃣ Run analysis only on changed files**
```bash
git diff --name-only origin/main...HEAD | grep '\.py$' | xargs pylint
```
> Cuts analysis time drastically, especially in large monorepos.

**2️⃣ Run analysis in parallel**
```bash
flake8 --jobs=4
```
Or split checks across CI matrix jobs:
```yaml
strategy:
  matrix:
    part: [backend, frontend]
```

**3️⃣ Shift left: run analysis pre-CI**
```bash
pre-commit install
```
Tools: `pre-commit`, `husky`, `lint-staged`.

**4️⃣ Run heavy checks on a schedule**
- Keep quick linting in PR builds; offload deep security scans (`bandit`, `semgrep`) to nightly workflows.
```yaml
on:
  schedule:
    - cron: '0 2 * * *'   # Runs at 2 AM UTC
```

**5️⃣ Tune rules and severity**
- Don't enable all rules by default; focus on high-impact rules (security, correctness) in CI; move style checks to a lower-priority job.

**6️⃣ Cache tool dependencies**
```yaml
- uses: actions/cache@v3
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
```

**🔹 Real-World Example**
`pylint` across a monorepo took 4–5 minutes per build.
- Limited to `git diff` changes · split frontend/backend linters · added pre-commit hooks
- **Result:** pipeline time reduced by ~70% without removing static checks.

<br>

---

### Q14. App is in `OutOfSync` state in Argo CD, but there are no Git changes — what could be the reason?

#### 📌 Short Explanation
Argo CD shows `OutOfSync` when the **live Kubernetes state differs from the desired Git state** — not only when Git changes.

#### ✅ Answer
Typically caused by manual changes in the cluster or non-Git-managed changes (e.g., autoscaling, label updates).

#### 📖 Common Reasons and Fixes

**1️⃣ Manual changes in the cluster**
Someone edited a deployment/config map/secret via `kubectl edit`, `kubectl patch`, Helm, or the Dashboard.
```bash
argocd app sync <app-name>
```

**2️⃣ Dynamic changes not tracked in Git**
Fields such as annotations or HPA replica counts change at runtime. Use `ignoreDifferences`:
```yaml
spec:
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

**3️⃣ Secrets automatically rotated**
Sealed Secrets, External Secrets, or Vault Agent Injector may mutate secrets. Use `ignoreDifferences` for secret fields or exclude secrets from tracking.

**4️⃣ Argo CD sync window missed**
If auto-sync is enabled but sync windows (time-based restrictions) exist, changes might not apply. Check sync windows or trigger a manual sync.

**5️⃣ CRDs or hooks trigger drift**
Helm post-install hooks or CRDs can modify resources post-sync. Ensure generated resources are tracked; use `skipHooks: true` if safe.

**🔹 Real-World Example**
An app stuck in OutOfSync with no Git changes — a DevOps engineer had manually increased the replica count to test scaling.
- **Resolution:** re-synced from Argo CD to restore Git-desired state; added `ignoreDifferences` for replica drift.

> **Summary:** OutOfSync means live state ≠ Git state. Manual changes, runtime drift, or auto-generated mutations can cause this — fix with sync or exclusions.

<br>

---

### Q15. When a build fails in Jenkins, how will you send an email?

#### 📌 Short Explanation
Jenkins can send automated emails using the **Email Notification** or **Editable Email Notification (Email Extension)** plugin.

#### ✅ Answer
Configure Jenkins to automatically send email alerts on failure using the built-in Email Notification system or the Email Extension Plugin for more control.

#### 📖 Steps to Configure

**1️⃣ Install the Email Extension Plugin**
- Manage Jenkins → Plugin Manager → Available → **Email Extension Plugin**

**2️⃣ Global configuration** (Manage Jenkins → Configure System)
- SMTP server, SSL/TLS, port (465 or 587), default sender address, authentication

```text
SMTP Server: smtp.gmail.com
Use SSL:     true
Port:        465
```

**3️⃣ Configure the project**
- Post-build Actions → **Editable Email Notification**
- Recipient list: `dev-team@example.com`
- Trigger: **Failure — Send email on build failure**

Optional content:
- **Subject:** `$PROJECT_NAME - Build #$BUILD_NUMBER - FAILED!`
- **Body:**
  ```text
  Build failed at $BUILD_URL
  Triggered by: $CAUSE
  ```

**4️⃣ Testing**
- Trigger a dummy failure and confirm the email arrives.

**🔹 Real-World Example**
- Used the Email Extension Plugin
- Configured triggers for `FAILURE`, `UNSTABLE`, and `FIXED`
- Custom HTML templates with links to logs and commit diffs
- Author-specific alerts for pull requests

<br>

---

<br>

<a id="terraform"></a>
# 🏗️ TOPIC 5: TERRAFORM

> Looping constructs and reusable infrastructure modules.

<br>

---

### Q1. What is the difference between `for_each` and `for` in Terraform?

#### 📌 Short Explanation
Both relate to "looping," but operate at different layers: `for_each` is a **meta-argument** that drives how many resource instances Terraform creates, whereas `for` is an **expression** you embed inside variables, locals, or arguments to build or transform collections.

#### ✅ Answer
- **`for_each`** tells Terraform to create **one instance per element** in a map or set (resources, modules, or data blocks).
- **`for`** is a collection expression used to generate or reshape lists/maps; it never creates resources on its own.

#### 📖 Detailed Explanation

| Aspect | `for_each` (meta-argument) | `for` (expression) |
|--------|----------------------------|--------------------|
| **Primary role** | Creates/manages **multiple instances** | Builds or transforms **values** in place |
| **Where used** | Resource, module, data, `dynamic` block header | Any argument, variable default, local, output |
| **Input type** | Map or set of strings | List, map, or set (any iterable) |
| **Outcome** | New resource addresses (`aws_instance.web["a"]`) | A new collection value (no extra resources) |

**Example 1 — `for_each` creating three S3 buckets**

```hcl
resource "aws_s3_bucket" "logs" {
  for_each = {
    us = "us-east-1"
    eu = "eu-west-1"
    ap = "ap-southeast-1"
  }

  bucket = "app-logs-${each.key}"
  region = each.value
}
```
Terraform plans three distinct buckets: `aws_s3_bucket.logs["us"]`, `aws_s3_bucket.logs["eu"]`, `aws_s3_bucket.logs["ap"]`.

**Example 2 — `for` expression shaping data**

```hcl
variable "allowed_cidrs" {
  type    = list(string)
  default = ["10.0.0.0/24", "10.1.0.0/24"]
}

locals {
  cidr_to_rule = {
    for cidr in var.allowed_cidrs :
    cidr => { protocol = "tcp", port = 22 }
  }
}
```
Produces a map:
```json
{
  "10.0.0.0/24": { "protocol": "tcp", "port": 22 },
  "10.1.0.0/24": { "protocol": "tcp", "port": 22 }
}
```

> **Key takeaways**
> - `for_each` → use when you need Terraform to create **N physical resources**.
> - `for` → use when you need a **derived list or map** for arguments or outputs.
> - They complement each other: build a map with `for`, then pass it to `for_each` for scalable resource creation.

<br>

---

### Q2. What are modules in Terraform and why should we use them?

#### 📌 Short Explanation
Modules are the core unit of code organization in Terraform. This question tests your understanding of reusability, abstraction, and clean code practices in Infrastructure as Code.

#### ✅ Answer
Terraform modules are **self-contained packages of Terraform configuration** that can be reused across projects or components. We use them to avoid repetition, enforce consistency, and organize infrastructure into logical components.

#### 📖 Detailed Explanation
A Terraform module is just a folder with `.tf` files — local, on GitHub, or on the Terraform Registry.

Two types:
- **Root module:** the directory where `terraform apply` is run.
- **Child module:** a reusable set of configurations called from a root or another module.

**🔹 Why use modules?**

| Benefit | Description |
|---------|-------------|
| **Reusability** | Define once, use anywhere (e.g., a VPC module used by multiple environments) |
| **Abstraction** | Hide complex logic behind simple variables and outputs |
| **Consistency** | Standardize infrastructure setup across teams |
| **Scalability** | Manage large-scale infrastructure with modular breakdown |
| **Maintainability** | Isolate changes to individual components without affecting the whole setup |

**🔹 Example: Creating an EC2 module**

```text
modules/
└── ec2-instance/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

Calling the module from root:

```hcl
module "my_ec2" {
  source        = "./modules/ec2-instance"
  instance_type = "t2.micro"
  ami_id        = "ami-0abcd1234"
}
```

**🔹 Real-world use case**
> *"We had to provision EC2 instances with the same tags, monitoring setup, and IAM roles across multiple environments. Instead of duplicating this logic, we created a reusable `ec2-instance` module. This allowed dev teams to spin up compliant EC2s with just a few input variables."*

> **Key takeaway:** Terraform modules are like **functions in programming** — they promote clean, DRY code and help build scalable, maintainable infrastructure.

<br>

---

<br>

<a id="docker"></a>
# 🐳 TOPIC 6: DOCKER

> Container troubleshooting, Dockerfile instructions, storage, cleanup, registries, CMD vs ENTRYPOINT and daily commands.

<br>

---

### Q1. Docker container exits immediately — how will you troubleshoot?

#### 📌 Short Explanation
A common scenario where a container runs and stops right away. The interviewer assesses your debugging skills, understanding of container lifecycle, and awareness of entrypoint behavior.

#### ✅ Answer
Inspect the container logs, verify the Dockerfile entrypoint/command, and check whether the container runs a long-lived process. Containers often exit if the main process completes or crashes.

#### 📖 Detailed Explanation
When a container starts, it runs the command defined in `CMD` or `ENTRYPOINT`. If that process ends — successfully or due to an error — the container exits.

**Step-by-step troubleshooting**

**1️⃣ Check logs**
```bash
docker logs <container_id_or_name>
```

**2️⃣ Inspect the Dockerfile**
```dockerfile
CMD ["python", "app.py"]
```
If `app.py` exits immediately, so does the container.

**3️⃣ Run in interactive mode**
```bash
docker run -it <image> /bin/bash
```

**4️⃣ Override CMD/ENTRYPOINT temporarily**
```bash
docker run -it <image> /bin/sh
```

**5️⃣ Check the exit status**
```bash
docker inspect <container_id> --format='{{.State.ExitCode}}'
```
A non-zero exit code often indicates an error.

**🔹 Real-world example**
> *"A Node.js app's Dockerfile `CMD` ran `node index.js`, but the file wasn't copied during the build. Logs showed 'cannot find module'. I fixed it by checking the `COPY` command."*

> **Key takeaway:** A Docker container must run a **long-running foreground process**. If it finishes or crashes, the container exits.

<br>

---

### Q2. What is the purpose of the `EXPOSE` instruction in a Dockerfile?

#### 📌 Short Explanation
Tests your understanding of container networking and how ports are communicated within and outside the container runtime.

#### ✅ Answer
`EXPOSE` indicates which port the containerized application will listen on at runtime. It serves as **documentation** and a signal to tools like Docker Compose, but it does **not** actually publish the port.

#### 📖 Detailed Explanation
To make a port accessible to the host or external clients, you still need `-p` / `--publish`:

```bash
docker run -p 8080:80 myapp
```
- `80` → port inside the container (maybe defined using `EXPOSE`)
- `8080` → port exposed on the host machine

**Example Dockerfile**

```dockerfile
FROM nginx:alpine
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```bash
docker run mynginx                  # NOT accessible externally
docker run -p 8080:80 mynginx       # accessible on host port 8080
```

**🔹 Real-world insight**
> *"We used `EXPOSE` in all our base images so developers and Docker Compose could understand default ports — even though we always mapped them explicitly with `ports:` in Compose."*

> **Key takeaway:** `EXPOSE` is documentation inside the image — it doesn't open ports on its own.

<br>

---

### Q3. Port is not accessible on localhost even after using the `-p` flag. How do you troubleshoot?

#### 📌 Short Explanation
A common container networking issue testing your ability to debug port exposure and host-container networking.

#### ✅ Answer
Check whether the app inside the container is listening on the correct interface and port. Often the app listens on `127.0.0.1` instead of `0.0.0.0` inside the container.

#### 📖 Detailed Explanation
`-p 8080:80` forwards host port 8080 to container port 80. But if the app listens only on `127.0.0.1`, it won't accept forwarded traffic.

**Example scenario**

```bash
docker run -p 8080:5000 myapp
```
```python
app.run(host='127.0.0.1', port=5000)   # ❌ only reachable from inside the container
```

**✅ Fix** — listen on all interfaces:

```python
app.run(host='0.0.0.0', port=5000)
```

**🔹 Additional things to verify**
- `docker ps` — confirm the port is published correctly
- Check the listening process:
  ```bash
  docker exec -it <container> netstat -tulnp
  ```
- Check firewall/security group rules (cloud VMs)
- Make sure no other service on the host is using that port

**🔹 Real-world insight**
> *"Our Flask app worked with `curl localhost` inside the container but was unreachable from the host. Changing `host='127.0.0.1'` to `'0.0.0.0'` solved it instantly."*

> **Key takeaway:** Publishing ports is only part of the setup. The application must listen on `0.0.0.0` for external traffic to reach it.

<br>

---

### Q4. Your application container loses data when it stops and restarts. How do you fix this?

#### 📌 Short Explanation
Checks your understanding of Docker storage and persistence. Containers are ephemeral — data written to the container filesystem is lost on removal unless volumes are used.

#### ✅ Answer
Mount a **Docker volume** or **bind mount** to persist data outside the container.

#### 📖 Detailed Explanation
By default, containers store data in a writable layer that is lost when the container is removed.

**🔹 1. Docker Volumes** (managed in `/var/lib/docker/volumes/`)

```bash
docker volume create mydata
docker run -v mydata:/app/data myapp
```

**🔹 2. Bind Mounts** (host directory mounted into the container)

```bash
docker run -v /host/data:/app/data myapp
```

**🔹 Real-world example**
A container running SQLite stores its `.db` file inside the container — every stop loses the DB.

```bash
docker run -v sqlite_data:/app/db my-sqlite-app
```

**🔹 Verify persistence**

```bash
docker stop myapp
docker rm myapp
docker run -v mydata:/app/data myapp     # data is still there!
```

> **Key takeaway:** Containers are ephemeral by design. Use volumes or bind mounts for databases, logs, or uploaded files.

<br>

---

### Q5. You changed your code, rebuilt the image, but the change isn't reflected. What could be the issue?

#### 📌 Short Explanation
Tests your understanding of image layers, build caching, and volume mounting in development setups.

#### ✅ Answer
Usually either **build caching** or a **volume overriding** the updated code. Verify the image was rebuilt correctly and that no volume is masking the changes.

#### 📖 Detailed Explanation

**🔹 1. Docker build cache**
Docker reuses cached layers if nothing changed in the context, possibly skipping the step that copies new code.

```bash
docker build --no-cache -t myapp:latest .
```

Place `COPY` instructions after dependencies:

```dockerfile
# ❌ BAD: Code copied early, everything cached
COPY . .

# ✅ GOOD: Dependencies first, then app code
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

**🔹 2. Volume overwriting image content**
A volume mounted at `/app` overrides whatever was built into the image at that path:

```bash
docker run -v "$(pwd)":/app myapp
```
Fix: remove the volume mount, or update your local files too.

**🔹 Steps to verify**

```bash
docker run -it myapp:latest cat /app/main.py   # check file contents in the image
docker ps                                       # list running containers
docker images | grep myapp                      # make sure you run the updated image
```

**🔹 Real-world insight**
> *"I rebuilt a Node.js image after updating code, but a `-v "$(pwd)":/app` bind mount kept running old code from my host. Took me a while to realize the mount was masking the update."*

> **Key takeaway:** Check for caching in `docker build` and ensure no local volumes mask your image content.

<br>

---

### Q6. App crashes with "Permission Denied" in a container but works fine on localhost. What could be the issue?

#### 📌 Short Explanation
Tests your knowledge of how file permissions, user IDs, and execution rights differ inside a container versus the host.

#### ✅ Answer
The app or script likely lacks proper execution permissions in the container, or is being run as a non-root user without access. Check file permissions and adjust the Dockerfile.

#### 📖 Detailed Explanation

**🔹 1. Permissions not preserved during `COPY`**
```dockerfile
COPY start.sh /app/start.sh
CMD ["/app/start.sh"]      # ❌ Permission denied
```
✅ Fix:
```dockerfile
RUN chmod +x /app/start.sh
```

**🔹 2. Running as a non-root user**
```dockerfile
USER appuser
```
That user may not have access unless ownership/permissions are set:
```dockerfile
RUN chown -R appuser:appuser /app
```

**🔹 3. Volume mounts with restricted permissions**
```bash
docker run -v $(pwd)/data:/data myapp
```
If `/data` contains files not readable by the container user → "Permission Denied."
```bash
chmod -R 755 data/
```
Or avoid mounting sensitive paths.

**🔹 Debugging steps**
```bash
docker exec -it <container> ls -l /app
docker run -it myapp /bin/sh
```

**🔹 Real-world insight**
> *"A Python script crashed with Permission Denied in CI. We copied it from the host but forgot `chmod +x`. Fixed by adding `RUN chmod +x script.py`."*

> **Key takeaway:** Always check file permissions and user context in your Dockerfile.

<br>

---

### Q7. Your Docker host is running out of disk space. How do you clean up and prevent it?

#### 📌 Short Explanation
Checks your understanding of Docker's local storage usage — images, containers, volumes, and build cache — and how to manage them.

#### ✅ Answer
Use Docker system prune commands to clean up unused containers, images, volumes, and networks. Then investigate large files under Docker's storage directory and implement a cleanup policy.

#### 📖 Detailed Explanation
Docker stores data in `/var/lib/docker` on Linux hosts.

**🔹 Cleanup steps**

**1️⃣ Prune unused Docker resources**
```bash
docker system df          # Overview of space used
docker system prune       # Stopped containers, unused networks, dangling images, build cache
docker system prune -a --volumes
```
> ⚠️ `-a --volumes` deletes **all unused images** and **all unused volumes**.

**2️⃣ Remove unused volumes manually**
```bash
docker volume ls -f dangling=true
docker volume rm $(docker volume ls -qf dangling=true)
docker volume prune
```

**3️⃣ Remove unused images selectively**
```bash
docker images --format "{{.Repository}}:{{.Tag}}\t{{.Size}}"
docker rmi <image-id>
```

**4️⃣ Investigate storage usage**
```bash
sudo du -sh /var/lib/docker/*
```

**🔹 Preventive actions**
- Weekly cron prune: `docker system prune -a --volumes -f`
- Use smaller base images (alpine, distroless)
- Clean intermediate layers:
  ```dockerfile
  RUN apt-get update && apt-get install -y something \
      && rm -rf /var/lib/apt/lists/*
  ```

**🔹 Real-world insight**
> *"A CI runner ran out of disk due to hundreds of dangling images and orphan volumes. We added a weekly `docker system prune` cron job and used multistage builds to reduce image size."*

> **Key takeaway:** Docker doesn't auto-clean. Regularly prune unused resources to avoid downtime.

<br>

---

### Q8. How will you debug a live container without stopping or restarting it?

#### 📌 Short Explanation
Tests your ability to troubleshoot and inspect a running container in real time — a critical skill for production debugging.

#### ✅ Answer
Use `docker exec` or `docker attach` to connect to the running container, then inspect logs, processes, file system, and network configuration.

#### 📖 Detailed Explanation

**1️⃣ `docker exec` — run commands inside the container**
```bash
docker exec -it <container-id or name> /bin/sh
docker exec -it <container-id> /bin/bash
docker exec -it myapp cat /var/log/app.log
docker exec -it myapp env
```

**2️⃣ `docker logs` — view stdout/stderr**
```bash
docker logs -f <container-id>
```

**3️⃣ `docker inspect` — metadata and configuration**
```bash
docker inspect <container-id>
docker inspect -f '{{.Config.Env}}' <container-id>
```

**4️⃣ `docker top` — inspect running processes**
```bash
docker top <container-id>
```

**5️⃣ Check network settings and connectivity**
```bash
docker exec -it <container-id> netstat -tulnp
docker exec -it <container-id> curl http://localhost:port
```

**6️⃣ `docker attach` (with caution)**
```bash
docker attach <container-id>
```
> ⚠️ Pressing `Ctrl+C` may stop the container. Use `Ctrl+P` then `Ctrl+Q` to detach safely.

**🔹 Real-world insight**
> *"Our containerized Flask app returned 500s. We `docker exec`'d in, checked logs, and found a missing environment variable. We confirmed the fix live before deploying a proper image."*

> **Key takeaway:** Use `exec` for shell access, `logs` for output, `inspect` for config, and `top` for processes — all without downtime.

<br>

---

### Q9. Which container registry do you use in your organization?

#### 📌 Short Explanation
Helps interviewers understand your familiarity with secure image storage, image scanning, versioning, and registry integration in CI/CD.

#### ✅ Answer
We primarily use **Amazon Elastic Container Registry (ECR)** — it integrates well with AWS services, supports image scanning, and works smoothly with CI/CD. We also occasionally use **Docker Hub** for public images and **GitHub Container Registry** for GitHub-hosted internal projects.

#### 📖 Detailed Explanation

**🔹 Amazon ECR**
- Fully managed Docker registry by AWS
- IAM integration for fine-grained permissions
- Private and public repositories
- Integrates with ECS, EKS, CodePipeline
- Built-in vulnerability scanning (Amazon Inspector)
- Image tagging and lifecycle policies to clean up old images

**Example workflow**

```bash
aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account>.dkr.ecr.<region>.amazonaws.com
docker build -t myapp .
docker tag myapp:latest <repo-url>:latest
docker push <repo-url>:latest
```

**🔹 GitHub Container Registry (GHCR)**
- Great for projects already on GitHub; authenticates with a GitHub token; easy GitHub Actions integration

**🔹 Other common registries**

| Registry | Best for |
|----------|----------|
| Docker Hub | Public images |
| Azure Container Registry (ACR) | Azure-based projects |
| Google Artifact Registry | GCP users |
| Harbor | On-prem self-hosted enterprise registries |

**🔹 Real-world insight**
> *"We had a policy to scan every image before deployment. ECR's native scanning and IAM-based access control made it easy to enforce. Lifecycle rules deleted untagged images older than 14 days, saving storage cost."*

<br>

---

### Q10. What is the difference between `CMD` and `ENTRYPOINT` in a Dockerfile?

#### 📌 Short Explanation
Evaluates your understanding of how containers are initialized — how default commands are defined and how they behave when overridden at runtime.

#### ✅ Answer
Both define what runs when a container starts, but:
- **`ENTRYPOINT`** is fixed and always executed.
- **`CMD`** provides default arguments that can be overridden.
- When combined, `CMD` acts as arguments to `ENTRYPOINT`.

#### 📖 Detailed Explanation

**🔹 CMD**
```dockerfile
FROM ubuntu
CMD ["echo", "Hello from CMD"]
```
```bash
docker run myimage
# Output: Hello from CMD

docker run myimage echo "Overridden"
# Output: Overridden
```

**🔹 ENTRYPOINT**
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
```
```bash
docker run myimage Hello from ENTRYPOINT
# Output: Hello from ENTRYPOINT
```
You can't override `ENTRYPOINT` without the `--entrypoint` flag.

**🔹 ENTRYPOINT + CMD together**
```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello from CMD"]
```
```bash
docker run myimage                  # Output: Hello from CMD
docker run myimage Custom message   # Output: Custom message
```
Final command = `echo Hello from CMD`.

**🔹 Summary Table**

| Feature | `CMD` | `ENTRYPOINT` |
|---------|-------|--------------|
| Purpose | Default command or args | Fixed executable |
| Overridable | ✅ Easily | ❌ Only with `--entrypoint` |
| Combination | Acts as args to ENTRYPOINT | Works with CMD as default args |

**🔹 Real-world example**
> *"Our image used `ENTRYPOINT` to run the app's binary. We passed different flags for staging and prod via `CMD`, which worked across environments without changing the image."*

<br>

---

### Q11. What Docker commands do you use on a day-to-day basis?

#### 📌 Short Explanation
Evaluates your practical experience — whether you're just running containers or actively building, debugging, and cleaning up Docker environments.

#### ✅ Answer

| # | Command | Example | Purpose |
|---|---------|---------|---------|
| 1 | `docker ps` | `docker ps` / `docker ps -a` | Show running (or all) containers |
| 2 | `docker build` | `docker build -t myapp:latest .` | Build an image from a Dockerfile |
| 3 | `docker run` | `docker run -d -p 8080:80 myapp` | Run a container from an image |
| 4 | `docker exec` | `docker exec -it myapp_container /bin/bash` | Execute a command inside a running container |
| 5 | `docker logs` | `docker logs -f myapp_container` | View (follow) container logs |
| 6 | `docker stop` / `start` | `docker stop myapp_container` | Stop or restart a container |
| 7 | `docker images` | `docker images` | List images; check size/version |
| 8 | `docker rm` / `rmi` | `docker rm myapp_container` · `docker rmi myapp:latest` | Remove a container or image |
| 9 | `docker system prune` | `docker system prune -a` | Clean unused containers, images, networks |
| 10 | `docker inspect` | `docker inspect myapp_container` | Inspect network, mounts, env variables |

**🔹 Real-world insight**
> *"While troubleshooting I use `docker exec` for a shell, `docker logs` for output, and restart if needed. During build optimizations, `docker system prune` freed over 5GB of unused image data."*

<br>

---

### Q12. When will you forcefully remove a container and how?

#### 📌 Short Explanation
Tests your understanding of container lifecycle management and handling stuck or unresponsive containers in real-world situations.

#### ✅ Answer
Yes — when containers were stuck in a dead or unresponsive state. Use:

```bash
docker rm -f <container-id>
```

#### 📖 Detailed Explanation
Containers may hang during shutdown because of:
- An unresponsive main process (PID 1)
- Resource locks or file descriptor issues
- Failed signal handling
- Docker daemon bugs or system resource exhaustion

In such cases, a regular `docker stop` may not terminate the container.

`docker rm -f` forcibly stops the container (sends `SIGKILL`) and then removes it.

**🔹 Before you force remove — check the state**

```bash
docker ps -a
docker inspect <container-id>
docker logs <container-id>
```

Try a graceful shutdown first:

```bash
docker stop <container-id>
```
If it doesn't respond in time (default 10s timeout):

```bash
docker rm -f <container-id>
```

**🔹 Real-world example**
> *"Our Python app became unresponsive due to a runaway thread. `docker stop` didn't work, so we used `docker rm -f` and redeployed a fixed version."*

**⚠️ Caution**
- Force removal kills the container immediately — no cleanup
- Any unsaved/non-persisted data is lost
- Use with caution in production

> **Key takeaway:** Use `docker rm -f` only when a container is stuck. Always try a graceful stop first, and keep important data in volumes.

<br>

---

<br>

<a id="kubernetes"></a>
# ☸️ TOPIC 7: KUBERNETES

> Architecture, Services, Ingress, networking, deployment & rollback strategies, scheduling, probes and troubleshooting.

<br>

---

### Q1. Can you explain the architecture of a Kubernetes cluster and the components involved?

#### 📌 Short Explanation
Tests your conceptual understanding of how Kubernetes is structured — control plane, worker nodes, and the communication between components that enables orchestration.

#### ✅ Answer
A cluster consists of a **Control Plane** (API Server, Scheduler, Controller Manager, etcd) and multiple **Worker Nodes** (Kubelet, Kube Proxy, Container Runtime). The control plane manages cluster state, while worker nodes run the actual workloads (pods).

#### 📖 Detailed Explanation

**🔹 1. Control Plane — the brain of the cluster**

| Component | Purpose |
|-----------|---------|
| **kube-apiserver** | Entry point to the cluster. All communication (kubectl, controllers) goes through this REST API |
| **etcd** | Distributed key-value store for all cluster data (config, state, secrets) |
| **kube-scheduler** | Assigns pods to nodes based on resources, taints/tolerations, affinities |
| **controller-manager** | Runs controllers (Node, ReplicaSet, Job) to maintain desired state |

**🔹 2. Worker Nodes — where your apps run**

| Component | Purpose |
|-----------|---------|
| **kubelet** | Agent on each node; talks to the API server; ensures containers are running |
| **kube-proxy** | Manages networking rules to route traffic to the correct pod using Services |
| **Container Runtime** | Runs the containers (containerd, CRI-O, Docker) |

**🔹 3. Pods, Deployments, and Services**
- **Pod:** smallest deployable unit; wraps one or more containers
- **Deployment:** manages ReplicaSets; ensures the desired number of pods are running
- **Service:** stable IP and DNS name for a set of pods; handles load balancing

**🔹 4. Add-Ons (optional but critical)**

| Add-on | Purpose |
|--------|---------|
| **CoreDNS** | Resolves service and pod names to IPs inside the cluster |
| **Ingress Controller** | Manages HTTP/HTTPS access from outside the cluster |
| **Metrics Server** | Collects metrics for autoscaling |

**🔹 Communication Flow (simplified)**
1. You run `kubectl apply -f deployment.yaml`
2. `kubectl` talks to the **kube-apiserver**
3. The API server stores config/state in **etcd**
4. The **scheduler** finds the best node for the pod
5. The **kubelet** on that node pulls the image and starts the container
6. **kube-proxy** and the Service route traffic to the pod

<br>

---

### Q2. What happens behind the scenes when you run `kubectl apply -f pod.yaml`?

#### 📌 Short Explanation
Checks whether you understand the full control flow and interaction between Kubernetes components — from API request to pod scheduling and execution.

#### ✅ Answer
The request goes to the API server, which stores the desired state in etcd. The scheduler selects a node, and the kubelet on that node pulls the image and starts the container using the container runtime.

#### 📖 Detailed Explanation

**Step 1 — `kubectl apply -f pod.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: app
      image: nginx
```
```bash
kubectl apply -f pod.yaml
```
Sends a REST request to the Kubernetes API server.

**Step 2 — API server receives and validates**
- The kube-apiserver authenticates and validates the request.
- If valid, it stores the desired state of the Pod in **etcd**.

**Step 3 — Controllers monitor etcd state**
- The scheduler constantly watches for unscheduled pods via the API server and sees your pod has no assigned node.

**Step 4 — Scheduler picks a suitable node**
- Considers resources, taints, affinities, and rules; assigns the pod by updating the spec with `nodeName`.

**Step 5 — kubelet on the selected node takes over**
- Sees the updated pod spec, pulls the `nginx` image, and uses the container runtime (e.g., containerd) to start the container.

**Step 6 — kube-proxy handles networking**
- Sets up networking rules to route traffic if the pod is part of a Service; DNS resolution via CoreDNS becomes available.

**Step 7 — Pod is running**
```bash
kubectl get pods
kubectl describe pod myapp
```

**🔹 Summary of Interactions**

| Component | Role in the Flow |
|-----------|------------------|
| `kubectl` | Sends request to API |
| `kube-apiserver` | Validates & stores the object |
| `etcd` | Stores desired state |
| `kube-scheduler` | Chooses node for pod |
| `kubelet` | Pulls image and starts container |
| `containerd` | Runs the actual container |
| `kube-proxy` | Sets up network rules |
| `CoreDNS` | Resolves internal DNS |

**🔹 Real-world insight**
> *"Pods remained in `Pending`. We found the scheduler couldn't place the pod due to node taints. Understanding this flow helped us fix it quickly."*

<br>

---

### Q3. What is the purpose of Services in Kubernetes?

#### 📌 Short Explanation
Checks if you understand how communication happens between pods and external clients in a dynamic environment where pods are frequently recreated with new IPs.

#### ✅ Answer
A Kubernetes Service provides a **stable network identity (IP and DNS)** to access a set of dynamic, ephemeral pods. It enables internal and external communication regardless of pod IPs, and supports load balancing across replicas.

#### 📖 Detailed Explanation
Pods are ephemeral — they can die and restart anytime, getting a new IP each time. A Service solves this.

**🔹 Key Features**

| Feature | Explanation |
|---------|-------------|
| **Stable IP/DNS** | Fixed ClusterIP and DNS name (`myapp.default.svc.cluster.local`) |
| **Load Balancing** | Routes requests to all healthy pods behind the service |
| **Pod Discovery** | Lets other pods locate and talk to a set of backends |
| **Selectors** | Uses labels to find and group pods automatically |

**🔹 Example**

Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx
```

Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```
This service forwards traffic to all 3 nginx pods and provides one DNS name: `web-service.default.svc.cluster.local`.

**🔹 Types of Services**

| Type | Use Case |
|------|----------|
| `ClusterIP` (default) | Internal-only communication |
| `NodePort` | Exposes service on a port of each node |
| `LoadBalancer` | Exposes via a cloud provider's external LB |
| `ExternalName` | Maps service to an external DNS name |
| `Headless` | No load balancing — exposes individual pod DNS (StatefulSets) |

**🔹 Real-world insight**
> *"We had an `auth-service` behind a ClusterIP. Other services used it via DNS. Even as pods restarted or scaled, the endpoint remained stable."*

<br>

---

### Q4. Why is hardcoding Pod IP communication a bad practice?

#### 📌 Short Explanation
Evaluates your understanding of Kubernetes networking and the dynamic nature of pods — and why abstractions like Services should be used.

#### ✅ Answer
Pod IPs are **ephemeral**. Pods can restart, scale, or be rescheduled, resulting in a new IP. This breaks communication. Use Kubernetes **Services** to provide a stable endpoint.

#### 📖 Detailed Explanation

**🔹 Problem with hardcoding pod IPs** — pod IPs change due to:
- Pod crash/restart
- Node failure or scaling events
- Rolling updates
- Horizontal Pod Autoscaler changes

If you hardcode `10.244.1.17` and that pod is recreated, the IP is invalid and your app fails.

**🔹 Better approach: use Services**

```python
# ❌ Bad practice
requests.post("http://10.244.1.17:5000/api")

# ✅ Good practice
requests.post("http://auth-service.default.svc.cluster.local:5000/api")
```
- Requests always reach a healthy pod
- Built-in load balancing
- Kubernetes reroutes if pods change

**🏨 Analogy:** Hardcoding pod IPs is like writing to someone's hotel room — if they check out, your letter won't reach them. A Service is their permanent address.

**🔹 Bonus insight**
Stateful apps (MySQL, Kafka) may need stable network IDs — use **headless services with StatefulSets**, not static IPs.

> **Key takeaway:** Pod IPs are temporary. Use Services to decouple applications from underlying pod infrastructure.

<br>

---

### Q5. What are the different types of Kubernetes Services, and when would you use each?

#### 📌 Short Explanation
Evaluates your understanding of how different service types expose applications inside or outside the cluster.

#### ✅ Answer
Kubernetes provides 5 main service types: **ClusterIP**, **NodePort**, **LoadBalancer**, **ExternalName**, and **Headless**.

#### 📖 Detailed Explanation

**🔹 1. ClusterIP (default)** — internal-only communication

```yaml
spec:
  type: ClusterIP
```
*Use case:* Microservice A (backend) talks to Microservice B (auth-service) inside the cluster.

**🔹 2. NodePort** — exposes the service on a static port on each node's IP

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080
```
Access: `<NodeIP>:<NodePort>` — *Use case:* dev/test environments needing direct access without a load balancer.

**🔹 3. LoadBalancer** — exposes via a cloud provider's load balancer

```yaml
spec:
  type: LoadBalancer
```
*Use case:* production on AWS/GCP/Azure needing a managed public endpoint.

**🔹 4. ExternalName** — maps the service name to an external DNS name

```yaml
spec:
  type: ExternalName
  externalName: db.mycompany.com
```
*Use case:* pod needs to talk to an external DB/API using a Kubernetes-style DNS name.

**🔹 5. Headless Service** — no cluster IP; reach individual pod IPs

```yaml
spec:
  clusterIP: None
```
*Use case:* stateful apps (Kafka, MySQL) where each pod has a stable identity.

**🔹 Summary Table**

| Type | Exposed Outside Cluster | Use Case |
|------|-------------------------|----------|
| ClusterIP | ❌ | Internal microservice communication |
| NodePort | ✅ (via node IP:port) | Quick access for dev/test |
| LoadBalancer | ✅ (via public IP) | External user traffic in cloud |
| ExternalName | ✅ (resolves to DNS) | Connect to external services |
| Headless | ❌ (no VIP, DNS only) | Stateful apps needing stable IDs |

**🔹 Real-world insight**
> *"We expose APIs via LoadBalancer, but internal microservice communication uses ClusterIP. For MySQL StatefulSets, we use headless services to target specific replicas."*

<br>

---

### Q6. What are Labels and Selectors in Kubernetes and how are they used?

#### 📌 Short Explanation
Evaluates your understanding of how Kubernetes identifies and groups objects using metadata — fundamental to pod selection, service discovery, and workload management.

#### ✅ Answer
**Labels** are key-value pairs attached to objects for identification. **Selectors** filter or group objects based on those labels. Services, ReplicaSets, and Deployments use selectors to manage the right set of pods.

#### 📖 Detailed Explanation

**🔹 Labels**
```yaml
metadata:
  labels:
    app: frontend
    env: production
```

**🔹 Selectors**
```yaml
selector:
  app: frontend
```

**🔹 Example: Pod + Service using Labels and Selectors**

Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  labels:
    app: frontend
    tier: web
spec:
  containers:
    - name: nginx
      image: nginx
```

Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - port: 80
```
The service automatically routes traffic to all pods with `app: frontend`.

**🔹 Why Labels & Selectors Are Useful**

| Use Case | How Labels Help |
|----------|-----------------|
| Service-to-Pod communication | Services use selectors to match pods |
| Rolling updates & scaling | Deployments use label selectors |
| Monitoring & grouping metrics | Tools like Prometheus use labels |
| Cost allocation / chargeback | Labels represent teams, owners, projects |
| Node affinity & scheduling | Pods match labels on nodes |

**🔹 Real-world insight**
> *"We used labels like `team: payments` and `env: staging` to filter pods in monitoring dashboards and CI pipelines."*

> **Key takeaway:** Labels describe objects; selectors find and group them.

<br>

---

### Q7. NodePort or LoadBalancer — what would you recommend, and why?

#### 📌 Short Explanation
Tests your understanding of service types for external traffic routing and cloud-native practices — scalability, security, and maintainability.

#### ✅ Answer
I recommend **LoadBalancer** for production environments, especially in cloud clusters. It provides a managed, scalable, and reliable way to expose services externally.

#### 📖 Detailed Explanation

**🔹 NodePort**
- Exposes a service on a static port (30000–32767) on every node
- Access via `<NodeIP>:<NodePort>`
- Suitable for development/testing, or behind an external load balancer

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080
```

❌ **Cons:** manual external load balancing/DNS management · port range limitation · no TLS termination or advanced routing.

**🔹 LoadBalancer**
- Provisions a cloud-managed external load balancer (AWS ELB, Azure LB)
- Automatically routes traffic to backend pods

```yaml
spec:
  type: LoadBalancer
  ports:
    - port: 80
```

✅ **Pros:** public IP automatically assigned · built-in cloud integrations · scales with the app · easier TLS and health checks.

**🔹 Example Use Cases**

| Use Case | Recommended |
|----------|-------------|
| Local testing on Minikube | NodePort |
| Dev/staging in the cloud | NodePort (sometimes) |
| Production in the cloud | **LoadBalancer** ✅ |

**🔹 Real-world insight**
> *"In our AWS cluster we use LoadBalancer services to expose APIs — it attaches an ELB and handles routing. For local Docker Desktop clusters we sometimes use NodePort for quick testing."*

<br>

---

### Q8. How are Kubernetes Services related to kube-proxy?

#### 📌 Short Explanation
Evaluates your understanding of internal networking — how services route traffic to pods and what role kube-proxy plays behind the scenes.

#### ✅ Answer
**kube-proxy implements the logic of Kubernetes Services.** It runs on each node and routes service traffic to the correct backend pods using iptables, ipvs, or eBPF rules.

#### 📖 Detailed Explanation

**🔹 What is kube-proxy?**
- A networking daemon running on every node
- Watches the API server for updates on Services and Endpoints
- Programs system-level networking (iptables, ipvs, eBPF) to route traffic

**🔹 How kube-proxy works with Services**
1. When a Service is created, Kubernetes generates:
   - A stable virtual IP (ClusterIP)
   - A mapping to pods matching the selector (via Endpoints / EndpointSlices)
2. kube-proxy receives this info and updates networking rules on the node to:
   - Listen on the Service IP and port
   - Forward traffic to one of the matching pods (load balancing)

**🔹 Example Flow**

```yaml
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app
  ports:
    - port: 80
```
When a user accesses `my-app.default.svc.cluster.local:80`:
- DNS resolves to ClusterIP (e.g., `10.96.0.15`)
- Traffic hits this virtual IP
- kube-proxy intercepts the request
- It forwards to one of the pods selected by the service

**🔹 Relationship Table**

| Component | Role |
|-----------|------|
| Kubernetes Service | Defines virtual IP + selector |
| kube-proxy | Implements traffic routing logic |
| Endpoints / EndpointSlices | Lists pod IPs backing the service |
| iptables / ipvs / eBPF | Actual mechanism for forwarding packets |

**🔹 Real-world insight**
> *"Traffic was inconsistent — kube-proxy was misconfigured on one node and not syncing rules. Restarting kube-proxy and checking iptables resolved it."*

> **Key takeaway:** Services define *what* to expose — kube-proxy makes it happen.

<br>

---

### Q9. What are the disadvantages of the LoadBalancer service type in Kubernetes?

#### 📌 Short Explanation
Checks your awareness of trade-offs when exposing services with cloud-managed load balancers — cost, scalability, and flexibility.

#### ✅ Answer
- 💰 **Cost** — a load balancer per service
- 📈 **Scalability limits** — 1:1 mapping between service and LB
- 🔒 **Vendor lock-in** — cloud-specific implementation
- 🛣️ **No advanced (L7) routing** — unlike Ingress controllers

#### 📖 Detailed Explanation

**🔹 Quick recap**
```yaml
spec:
  type: LoadBalancer
  ports:
    - port: 80
```
Kubernetes asks the cloud provider to provision an L4 load balancer (e.g., AWS ELB).

**🔹 Key Disadvantages**

**1️⃣ Cost overhead** — each LoadBalancer service creates a separate cloud LB that costs money even when idle.
> *"We had 10 microservices each with LoadBalancer — leading to unneeded monthly costs."*

**2️⃣ Scalability and management** — you can't reuse the same LB for multiple services; dozens of LB services are messy.

**3️⃣ Vendor lock-in** — only works where managed LBs exist; bare-metal/local needs something like **MetalLB**.

**4️⃣ No L7 (HTTP) routing** — only TCP/UDP (Layer 4); no path/host-based routing (can't split `/api` vs `/web` without Ingress).

**🔹 Alternative: Ingress Controller** (NGINX, AWS ALB) — one LB + multiple services, better routing, SSL termination, cost efficiency.

**🔹 Comparison**

| Feature | LoadBalancer | Ingress |
|---------|--------------|---------|
| Cost per service | High (1 LB each) | Low (1 LB shared) |
| HTTP routing support | ❌ | ✅ |
| External IP assigned | ✅ | ✅ |
| Works locally | ❌ (cloud only) | ✅ (via NGINX etc.) |

**🔹 Real-world insight**
> *"We used a LoadBalancer for the Ingress Controller only — not for individual services. That gave us cost control and L7 routing."*

<br>

---

### Q10. What is a Headless Service in Kubernetes and when did you use it?

#### 📌 Short Explanation
Tests your understanding of how Kubernetes provides direct access to individual pods — useful for stateful or clustered applications.

#### ✅ Answer
A Headless Service has **no ClusterIP**, so Kubernetes doesn't load balance. DNS returns the individual pod IPs. I've used it for StatefulSets like MySQL clusters and Kafka brokers where each pod needs direct access.

#### 📖 Detailed Explanation

```yaml
spec:
  clusterIP: None
```
DNS returns A/AAAA records for each backing pod, rather than a single IP.

**🔹 Why use it?**
- Each pod needs a stable network identity
- Clients connect to pods individually (not through a load balancer)
- You're using StatefulSets (DB clusters, message queues)

**🔹 Example: Headless Service with StatefulSet (MySQL)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
```
Pod DNS names:
```text
mysql-0.mysql-headless.default.svc.cluster.local
mysql-1.mysql-headless.default.svc.cluster.local
```

**🔹 Use case from experience**
> *"We used a headless service for a Kafka cluster. Each broker needed a stable hostname and had to be discoverable individually."*

**🔹 Key Differences vs Normal Service**

| Feature | ClusterIP Service | Headless Service |
|---------|-------------------|------------------|
| DNS returns | Single ClusterIP | Individual Pod IPs |
| Load balancing | Yes (round-robin) | No |
| Use with StatefulSet | ❌ Not ideal | ✅ Recommended |
| Use case | Web apps, APIs | Databases, clusters |

<br>

---

### Q11. Can a Pod access a Service in a different namespace? If yes, how?

#### 📌 Short Explanation
Tests your understanding of Kubernetes networking and DNS conventions across namespaces, and restrictions that might apply (e.g., NetworkPolicies).

#### ✅ Answer
**Yes.** Kubernetes networking is flat by default. Use the fully qualified service DNS name:

```text
<service-name>.<namespace>.svc.cluster.local
```

#### 📖 Detailed Explanation

**🔹 1. Using the FQDN**

```yaml
# In namespace "backend"
kind: Service
metadata:
  name: api
  namespace: backend
spec:
  ports:
    - port: 80
```
From a pod in namespace `frontend`:
```bash
curl http://api.backend.svc.cluster.local:80
```
DNS resolution order: `api` → `api.backend` → `api.backend.svc` → `api.backend.svc.cluster.local`.

**🔹 2. Using the ClusterIP (not recommended long-term)**
```bash
curl http://10.96.24.7:80
```
Brittle — IPs can change if the Service is recreated.

**🔹 3. Considerations & Restrictions**

| Item | Notes |
|------|-------|
| **NetworkPolicies** | Traffic is allowed by default; policies can restrict cross-namespace communication |
| **RBAC / ServiceAccounts** | DNS reachability ≠ RBAC. API server calls or secrets still require permissions |
| **Headless Services** | Same FQDN pattern, but DNS returns individual pod IPs |

**🔹 Real-world example**
> *"Our frontend pods called the payments API in the `payments` namespace using one env var: `PAYMENTS_URL=https://payments.payments.svc.cluster.local:443`. No extra config was needed because no NetworkPolicy blocked traffic."*

<br>

---

### Q12. How can you restrict access to a DB pod to only one app in the same namespace?

#### 📌 Short Explanation
Checks your knowledge of Kubernetes **NetworkPolicies** to enforce fine-grained, pod-level network security, even when resources share a namespace.

#### ✅ Answer
Create a NetworkPolicy that (1) selects the database pods and (2) allows ingress only from pods with a specific label. All other traffic is denied once the policy is in place.

#### 📖 Detailed Explanation
Kubernetes networking is open by default. NetworkPolicies whitelist traffic based on pod labels, namespaces, ports, and protocols.

**Step 1 — Label your pods**
```bash
kubectl label pods db-0 role=db
kubectl label pods app-0 role=api
```

**Step 2 — Create the NetworkPolicy**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-to-db
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: api
      ports:
        - protocol: TCP
          port: 5432
```

**Step 3 — Verify**
- `app-0` → `db-0` on port 5432 ✅
- Any other pod → `db-0` ❌ (connection refused / timed out)

**🔹 Why this works**

| Component | Purpose |
|-----------|---------|
| `podSelector` | Targets the DB pods (`role=db`) |
| `from` | Whitelists only pods with `role=api` |
| `ports` | Optionally restricts to 5432 |
| Default deny | Once a policy exists, all other ingress to selected pods is blocked unless allowed |

**🔹 Real-world insight**
> *"We secured a PostgreSQL StatefulSet by labelling it `role=db` and applying an ingress NetworkPolicy. Only the payments deployment could connect. QA pods in the same namespace lost DB access unless whitelisted."*

<br>

---

### Q13. What deployment strategy do you follow in your organization?

#### 📌 Short Explanation
Tests your understanding of real-world CI/CD practices and deployment risk mitigation — blue-green, rolling updates, canary.

#### ✅ Answer
We primarily follow **Rolling Update** using Kubernetes Deployments, combined with **Canary** deployments via Argo Rollouts or Flagger for critical services — ensuring zero downtime with gradual releases and automated rollback on failure.

#### 📖 Detailed Explanation

**🔹 1. Rolling Update (default in Kubernetes)**
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```
- Old pods are terminated gradually while new pods spin up
- Continuous availability / zero downtime; suited for stateless workloads

*Why we use it:* safe for non-critical changes · works out-of-the-box · easy to observe.

**🔹 2. Canary Deployment (for critical changes)**
Tools: [Argo Rollouts](https://argo-rollouts.readthedocs.io/), [Flagger](https://flagger.app/), Istio/Linkerd.

Traffic shifts like: `10% → 50% → 100%` with checks between steps.

*Benefits:* catch issues early with real traffic · automated rollback on metrics/log failure · controlled, observable releases.

**🔹 3. Blue-Green Deployment (used less frequently)**
- Deploy v2 alongside v1 in full → switch traffic via LB/Ingress → instant rollback possible.
- *Downsides:* resource-heavy, more infra complexity.

**🔹 Tooling We Use**

| Tool | Purpose |
|------|---------|
| **ArgoCD** | GitOps-based deployment management |
| **Argo Rollouts** | Progressive delivery strategies |
| **Prometheus** | Monitors health and SLOs |
| **Helm** | Templated Kubernetes deployments |

**🔹 Real-world insight**
> *"During a major upgrade of our authentication service, we used a Canary rollout with Argo Rollouts — 5%, 25%, then 100%. Argo Rollouts auto-paused when latency increased, preventing a major outage."*

<br>

---

### Q14. What is the rollback strategy you follow in your organization?

#### 📌 Short Explanation
Evaluates your team's preparedness for failed deployments — how you recover quickly while minimizing downtime and user impact.

#### ✅ Answer
An automated rollback strategy integrated with CI/CD (GitHub Actions + Argo CD). Rollbacks are triggered manually via Git revert, or automatically if health checks fail during canary/rolling deployments. The GitOps model makes rollbacks clean and reproducible.

#### 📖 Detailed Explanation

**🔹 GitOps Rollback (primary method)**
- Deployments are version-controlled as YAML in Git.
- If a deployment breaks production, revert the Git commit — Argo CD syncs the previous state back.

```bash
git revert <bad-commit>
git push origin main
```
✅ Fully auditable · consistent rollback to a known good state · Git history = deployment history.

**🔹 Canary Rollback (Argo Rollouts)**
- If the new version causes latency or errors, the rollout is paused automatically and either auto-rolled back or manually reverted.
```yaml
analysis:
  templates:
    - templateName: success-rate-check
  args:
    - name: success-rate
      value: "95"
```
If success rate drops below the threshold, the rollout fails.

**🔹 Helm Rollback (legacy services)**
```bash
helm rollback my-app 2
```

**🔹 Real-world scenario**
> *"We pushed a change to our payment API and saw increased 500 errors. Since Argo CD managed the deployment, we ran `git revert` and pushed. Within 2 minutes Argo CD synced the rollback and the service stabilized."*

**🔹 Safeguards We Use**

| Strategy | Purpose |
|----------|---------|
| Pre-deploy validation | Prevents pushing broken YAML to Git |
| Readiness & liveness probes | Catch bad pods early |
| SLO-based auto rollback | Canary rollouts monitored via metrics |
| Alerts on sync divergence | Argo CD notifies on drift |

<br>

---

### Q15. Design a solution to avoid rollbacks in production.

#### 📌 Short Explanation
Focuses on proactive quality assurance and risk mitigation — preventing bad releases from reaching production rather than reacting with a rollback.

#### ✅ Answer
A **"shift-left"** strategy with robust pre-deployment validation, progressive delivery, and automated quality gates using GitOps and observability tools — so only validated, low-risk changes reach production.

#### 📖 Detailed Explanation

**🔹 1. Pre-Deployment Safety Nets**

| Technique | Description |
|-----------|-------------|
| **Automated Testing** | Unit, integration, regression tests in CI before merge/deploy |
| **Schema & Config Validation** | `kubeval`, `kubeconform`, `opa`, `tflint` to validate infra code |
| **Security Scanning** | `Trivy`, `Snyk`, `Checkov` in the pipeline |
| **Static Code Analysis** | Linting, code smells, coverage via SonarQube |

**🔹 2. Use Progressive Delivery**
- Canary deployments (Argo Rollouts / Flagger)
- Feature flags to toggle features without deploying code
- Blue-Green for large releases where rollback speed matters

**🔹 3. Observability + Quality Gates**

| Type | Examples |
|------|----------|
| Latency | Increase in request duration |
| Error rate | 4xx/5xx spikes |
| Resource usage | Pod CPU/memory |
| Logs | Error/warning patterns |

Use these metrics in Argo Rollouts or CI to auto-pause or fail releases before full rollout.

**🔹 4. Use GitOps for Controlled Deployments**
- All deployments go through Git (Argo CD); no manual YAML applies → less human error
- Every change is traceable, auditable, reversible

**🔹 5. Real-World Guardrails**
> *"Every PR runs 500+ unit tests, Helm template validations, and schema checks. Once merged, Argo Rollouts starts a canary at 10% traffic, monitored via Prometheus. We proceed to 100% only if no error spikes appear within 10 minutes."*

**🔹 Tech Stack**

| Area | Tools |
|------|-------|
| CI/CD | GitHub Actions, Argo CD, Argo Rollouts |
| Code Quality | SonarQube, ESLint, PyLint |
| Infra Linting | kubeval, tflint, checkov |
| Observability | Prometheus, Loki, Grafana |
| Security | Trivy, Snyk, Aqua |

> **Key takeaway:** Treat deployment as a gradual, monitored process — not a one-shot push.

<br>

---

### Q16. Explain the various deployment strategies you've used in past roles.

#### 📌 Short Explanation
Helps the interviewer understand your experience with real-world release patterns, how you balance risk and velocity, and how you adapt to an application's nature and criticality.

#### ✅ Answer
I've used **Rolling Updates**, **Blue-Green**, and **Canary** deployments, depending on business impact, criticality, and the maturity of observability tooling.

#### 📖 Detailed Explanation

**🔹 1. Rolling Update (most common)**
- *What:* replaces old pods gradually with new ones; no downtime.
- *Where:* stateless microservices (product APIs, frontend apps) using native Kubernetes rolling updates.

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```
- *Tools:* Kubernetes Deployments, Argo CD, GitHub Actions
- *Why:* simple to manage; works well when monitoring is in place and rollback is quick (via GitOps).

**🔹 2. Blue-Green Deployment**
- *What:* deploy the new version in parallel (green); once validated, switch traffic from old (blue) to new.
- *Where:* monolith applications with complex startup logic; services needing immediate rollback due to customer SLA.
- *Tools:* AWS ALB + target groups, Helm, Jenkins pipelines
- *Why:* zero downtime; quick rollback (switch traffic back to blue).

**🔹 3. Canary Deployment**
- *What:* gradually roll out to a small % of users; observe metrics before increasing traffic.
- *Where:* payment services and high-risk business workflows with Prometheus-based health checks.
- *Tools:* Argo Rollouts, Flagger, Prometheus + Grafana
- *Why:* safer than rolling update for mission-critical components; catches errors before impacting all users.

<br>

---

### Q17. What is the role of CoreDNS in a Kubernetes cluster?

#### 📌 Short Explanation
Checks your understanding of service discovery and DNS resolution inside Kubernetes — a key part of internal communication.

#### ✅ Answer
CoreDNS is the default DNS server in Kubernetes. It translates internal service names (like `my-service.default.svc.cluster.local`) into Pod/Cluster IPs, enabling communication via DNS instead of hardcoded IPs.

#### 📖 Detailed Explanation

**🔹 What is CoreDNS?**
- A lightweight, extensible DNS server written in Go
- Replaced kube-dns as the default since Kubernetes v1.13
- Deployed as a Deployment in the `kube-system` namespace

**🔹 Why is it critical?**
Services are accessed like `http://my-app.default.svc.cluster.local`. Without CoreDNS:
- Pods couldn't resolve service names
- Inter-pod communication would break
- Service discovery would fail

**🔹 How CoreDNS works**
1. A pod makes a DNS request to resolve a service name.
2. The request is sent to the virtual IP `10.96.0.10` (default ClusterIP for CoreDNS).
3. CoreDNS uses the Kubernetes API to resolve the query.
4. It returns the Cluster IP (or Pod IPs for headless services).

**🔹 Configuration** (lives in a ConfigMap)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

**🔹 Common Use Cases**

| Use Case | Example |
|----------|---------|
| Service-to-service communication | `curl http://orders.default.svc.cluster.local` |
| StatefulSet communication | `mysql-0.my-db.default.svc.cluster.local` |
| Pod discovery in custom DNS zones | Extending CoreDNS with plugins for external name resolution |

<br>

---

### Q18. A node is tainted as `NoSchedule`. Can you still schedule a pod on it?

#### 📌 Short Explanation
Checks your understanding of taints and tolerations and how they affect pod scheduling.

#### ✅ Answer
**Yes** — but only if the pod has a **matching toleration**. Without a toleration, the scheduler ignores that node for the pod.

#### 📖 Detailed Explanation

**🔹 What does a `NoSchedule` taint do?**

```bash
kubectl taint nodes node-1 env=dev:NoSchedule
```
> *"Don't schedule any pods on this node unless the pod explicitly tolerates this taint."*

**🔹 How to still schedule a pod there — add a toleration**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  tolerations:
    - key: "env"
      operator: "Equal"
      value: "dev"
      effect: "NoSchedule"
  containers:
    - name: demo
      image: nginx
```

**🔹 Taint Effects**

| Taint Effect | Behavior |
|--------------|----------|
| `NoSchedule` | Scheduler won't place pods unless they have a matching toleration |
| `PreferNoSchedule` | Tries to avoid, but may still place pods if necessary |
| `NoExecute` | Evicts already-running pods unless they tolerate the taint |

**🔹 Real-world use case**
> *"We taint nodes with `team=analytics:NoSchedule` to dedicate them for heavy data processing. Only pods with that toleration land there — isolating workloads without separate node pools."*

<br>

---

### Q19. A pod is stuck in `CrashLoopBackOff` — what steps will you take?

#### 📌 Short Explanation
Tests your ability to debug failing pods, especially when containers crash repeatedly due to application or environment-related issues.

#### ✅ Answer
Check pod logs, container events, and resource limits. Common causes: application bugs, incorrect configs, failed dependencies, or OOM errors. Resolve the root cause based on findings.

#### 📖 Step-by-Step Troubleshooting

**1️⃣ Check pod status and reason**
```bash
kubectl get pod <pod-name> -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
```
Look under *Last State*, *Exit Code*, and *Events*.

**2️⃣ View container logs**
```bash
kubectl logs <pod-name> -c <container-name> --previous -n <namespace>
```
Use `--previous` for the last failed attempt. Look for: `Connection refused` · `File not found` · `Segmentation fault` · `Permission denied`.

**3️⃣ Check configuration or secret issues**
- Is a mounted ConfigMap/Secret missing or misconfigured? Are env vars or args wrong?

**4️⃣ Check missing dependencies**
- Is a required service/database unavailable? Are DNS entries resolving?

**5️⃣ Check resource constraints**
```bash
kubectl describe pod <pod-name>
```
If it shows `Reason: OOMKilled`, increase memory limits or optimize the app.

**6️⃣ Check image, CMD, ENTRYPOINT**
- Wrong image version · missing binary · script without execute permission. Test with Docker or `kubectl run`.

**7️⃣ Use an ephemeral container (K8s v1.23+)**
```bash
kubectl debug -it <pod-name> --image=busybox --target=<container-name>
```

**🔹 Real-world fix examples**
> *"A ConfigMap change removed a required ENV variable — we restored it and the pod worked. Another time we hit `OOMKilled` and increased memory limits from 256Mi to 512Mi."*

> **Key takeaway:** CrashLoopBackOff means the container is repeatedly failing. Find the root cause via logs, events, configs, and resource usage — don't just restart the pod.

<br>

---

### Q20. What is the difference between liveness and readiness probes in Kubernetes?

#### 📌 Short Explanation
Checks your understanding of Kubernetes health checks that manage container lifecycle and service availability.

#### ✅ Answer
- **Liveness probe** — checks if a container is alive; restarts it if unresponsive.
- **Readiness probe** — checks if a container is ready to receive traffic; if not, it's removed from service endpoints until ready.

#### 📖 Detailed Explanation
Probes are periodic checks. There are three types: **liveness**, **readiness**, and **startup**.

**🔹 Liveness Probe**
- *Purpose:* detect a stuck/dead container (infinite loops, deadlocks).
- *Behavior:* if it fails, the container is **restarted**.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```
💡 *"Should this container be killed and restarted?"*

**🔹 Readiness Probe**
- *Purpose:* check if the app is ready to accept traffic.
- *Behavior:* if it fails, the pod is removed from the Service endpoint list — **not restarted**.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 3
```
💡 *"Is this container ready to serve traffic?"*

**🔹 Real-world example**
> *"A Java app took ~40 seconds to load its cache. The readiness probe prevented early traffic, while the liveness probe restarted it if the app crashed at runtime."*

**🔹 Summary Table**

| Feature | Liveness Probe | Readiness Probe |
|---------|----------------|-----------------|
| Checks if app is | **Alive** | **Ready to serve traffic** |
| Failure action | Restarts the container | Removes from service routing |
| Restarted on fail? | ✅ Yes | ❌ No |
| Affects traffic? | ❌ No | ✅ Yes |

<br>

---

### Q21. What is the difference between Ingress and a LoadBalancer service in Kubernetes?

#### 📌 Short Explanation
Evaluates your understanding of exposing services externally and managing routing/traffic to applications inside the cluster.

#### ✅ Answer
A **LoadBalancer** service exposes a **single** service using a cloud provider's external load balancer. **Ingress** acts as a reverse proxy and routes HTTP(S) traffic to **multiple** services based on hostnames and paths — using a single external IP.

#### 📖 Detailed Explanation

**🔹 LoadBalancer Service**
- Creates a cloud-managed external LB (AWS ELB, Azure LB); one public IP per service
- Best for simple apps or non-HTTP protocols (TCP, UDP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

**🔹 Ingress Resource**
- HTTP reverse proxy; routes by hostname or path
- Uses a single LB IP (cost-effective); requires an Ingress Controller (NGINX, AWS ALB Controller)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

**🔹 Real-world use case**
> *"We had 12 backend services. Instead of 12 LoadBalancers, we used Ingress with host-based rules (`api.example.com`, `auth.example.com`) and handled TLS termination at the Ingress controller."*

**🔹 Summary Table**

| Feature | LoadBalancer Service | Ingress |
|---------|----------------------|---------|
| External IP per service | ✅ Yes (per service) | ❌ No (shared) |
| HTTP routing rules | ❌ No | ✅ Yes (path/host based) |
| TLS termination support | Manual | ✅ Built-in |
| Cost efficient | ❌ More IPs = more cost | ✅ Single entry point |
| Handles non-HTTP traffic | ✅ Yes (TCP/UDP) | ❌ HTTP/HTTPS only |
| Requires Ingress Controller | ❌ No | ✅ Yes |

<br>

---

### Q22. Your app works with ClusterIP but fails with Ingress — how do you troubleshoot?

#### 📌 Short Explanation
Tests your practical knowledge of Kubernetes networking, particularly Ingress controllers and how Ingress routing works.

#### ✅ Answer
Check that the Ingress controller is installed and running, verify ingress rules, confirm DNS/host header matches, and review logs and service connectivity. Most often the issue is incorrect rules, missing annotations, or DNS misconfiguration.

#### 📖 Step-by-Step Troubleshooting

**1️⃣ Is the Ingress Controller installed and running?**
```bash
kubectl get pods -n ingress-nginx
```

**2️⃣ Check the Ingress resource definition**
```bash
kubectl describe ingress <ingress-name>
```
Correct rules? Correct host/path? Do service name and port match your ClusterIP service?

**3️⃣ Check annotations and class**
```yaml
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
```
Or for newer versions:
```yaml
spec:
  ingressClassName: nginx
```

**4️⃣ Check DNS or host header configuration**
- DNS must point to the ingress controller's external IP, or add to `/etc/hosts`:
```text
<ingress-external-ip> app.example.com
```
Hitting the IP directly with host-based routing may return 404.

**5️⃣ Check logs of the Ingress controller**
```bash
kubectl logs -n ingress-nginx <controller-pod-name>
```
Common errors: `default backend - 404` · `no matching path rule` · TLS errors.

**6️⃣ Check backend service and pod health**
```bash
kubectl get endpoints <service-name>
```

**7️⃣ Check TLS/HTTPS configuration**
- TLS secret valid? Rules include a `tls` section? Controller supports HTTPS?

**🔹 Real-world example**
> *"Our app worked internally via ClusterIP but gave a 404 over Ingress. A missing `ingressClassName` meant the NGINX controller never picked up the resource."*

**🔹 Summary Table**

| Check | Description |
|-------|-------------|
| Ingress Controller Running | Must be deployed for Ingress to work |
| Ingress Rules & Paths | Must match service correctly |
| Hostname or Path Match | Ensure DNS or `/etc/hosts` is correct |
| Logs of Ingress Controller | Debug routing errors |
| Backend Service & Endpoints | Ensure pods are reachable |

<br>

---

### Q23. Why do I need to set up an Ingress Controller after creating an Ingress?

#### 📌 Short Explanation
Checks whether you understand the distinction between a declarative Ingress resource and the component that actually handles traffic — the Ingress Controller.

#### ✅ Answer
The Ingress resource is **just a set of routing rules**. Without an Ingress Controller, nothing interprets those rules or routes external HTTP/HTTPS traffic into the cluster.

#### 📖 Detailed Explanation

**🔹 What is an Ingress Resource?**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```
This defines the rule — but nothing processes it.

**🔹 What is an Ingress Controller?**
The implementation that reads Ingress rules and configures a proxy (NGINX, Traefik, HAProxy, AWS ALB) to route traffic. Popular controllers: `nginx-ingress`, `traefik`, `aws-alb-ingress-controller`, `gce-ingress-controller`.

**🔹 Ingress without a controller = no routing**
- You hit `myapp.example.com` and nothing responds
- You may see a 404 default backend or a connection timeout

**🔹 Real-world example**
> *"In a new dev cluster, we created an Ingress for our frontend but couldn't access it. We'd forgotten to install the NGINX ingress controller. Once installed via Helm, the route worked."*

**🔹 Summary Table**

| Component | Role |
|-----------|------|
| Ingress Resource | Specifies rules (host/path/service) |
| Ingress Controller | Watches Ingress resources and implements them via a proxy |
| Result without Controller | Rules are never executed → traffic not routed |

> **Key takeaway:** Creating an Ingress is like writing a script but never running it. You need a controller to process the rules.

<br>

---

### Q24. We have an in-house load balancer — can we use Ingress with it?

#### 📌 Short Explanation
Tests your understanding of the Ingress controller's role and how Kubernetes integrates with existing enterprise infrastructure.

#### ✅ Answer
Yes — you can place your in-house load balancer **in front of** a Kubernetes Ingress controller. But the load balancer cannot **replace** the Ingress controller; you still need one running in the cluster to process Ingress resources.

#### 📖 Detailed Explanation

**🔹 How it works**

```text
Client --> Load Balancer --> Ingress Controller --> Services --> Pods
```
- The in-house LB handles external traffic
- It forwards requests to the Ingress Controller (e.g., NGINX) inside the cluster
- The controller routes traffic based on Ingress resource rules

**🔹 What you cannot do**
Your in-house LB cannot:
- Understand Kubernetes Ingress YAML resources
- Dynamically route to services/pods based on Kubernetes annotations or labels

**🔹 Real-world example**
> *"Our team had a powerful in-house F5 load balancer. We forwarded all `*.company.com` traffic to the NGINX Ingress controller service. The F5 handled SSL termination; NGINX handled path-based routing inside the cluster."*

**🔹 Summary Table**

| Component | Role |
|-----------|------|
| In-house Load Balancer | Routes traffic to the cluster entry point (NodePort or LoadBalancer service) |
| Ingress Controller | Understands Ingress resources and routes inside the cluster |
| Ingress Resource | YAML rules for host/path-based routing |

**🔹 Recommendation**
Let a capable in-house LB handle TLS termination, WAF, and global traffic steering — then send traffic to a NodePort or LoadBalancer service fronting your Ingress Controller.

<br>

---

### Q25. Your Deployment has `replicas: 3`, but only 1 Pod is running — what could be wrong?

#### 📌 Short Explanation
Tests your ability to troubleshoot replica mismatches — where the desired state (3 pods) doesn't match the actual state (1 pod).

#### ✅ Answer
Possible reasons: resource constraints on nodes, scheduling issues, crashlooping pods, or affinity/taint restrictions preventing pods from starting.

#### 📖 Troubleshooting Checklist

**1️⃣ Check pod statuses**
```bash
kubectl get pods -l app=my-app
```
You might see 1 `Running` and 2 in `Pending` / `CrashLoopBackOff` / `ImagePullBackOff`.

**2️⃣ Describe the Deployment and Pods**
```bash
kubectl describe deployment my-deployment
kubectl describe pod <pod-name>
```
Look for failed scheduling events, crash loops, image pull errors, volume mounting errors.

**3️⃣ Check node capacity**
```bash
kubectl describe nodes
```
Out-of-resources nodes can't start new pods.

**4️⃣ Check affinity rules and taints**
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```
Pods will only schedule on matching nodes.

**5️⃣ Check for pod crashes**
```bash
kubectl logs <pod-name>
```

**🔹 Real-world example**
> *"We set a memory limit of 100Mi but the app needed 200Mi. Only one pod ran because the others kept getting OOMKilled. Increasing memory fixed it."*

**🔹 Summary Table**

| Check | What It Tells You |
|-------|-------------------|
| `kubectl get pods` | Status of all pods |
| `kubectl describe` | Events and reasons for pending/crashes |
| Node capacity | Resource exhaustion |
| Affinity/Taints | Constraints preventing scheduling |
| Container logs | Runtime crashes or app issues |

<br>

---

### Q26. Your Pod mounts a ConfigMap, but changes to the ConfigMap are not reflected — why?

#### 📌 Short Explanation
Tests your understanding of how ConfigMaps work when mounted as volumes, and what affects the propagation of updates.

#### ✅ Answer
If the ConfigMap is mounted as a volume, Kubernetes does reflect changes inside the container after a short delay (every ~1–2 minutes). But if your application reads the config only once at startup, changes won't be picked up unless the pod is restarted.

#### 📖 Detailed Explanation

**🔹 ConfigMap mount behavior**

```yaml
volumes:
  - name: config-volume
    configMap:
      name: my-config
volumeMounts:
  - name: config-volume
    mountPath: /etc/my-config
```
- Files in `/etc/my-config` are updated by kubelet roughly every minute
- The container sees new content only if it re-reads the file

**🔹 Why changes might not be reflected**
1. **App reads config only on startup** — caches it in memory
2. **Change detection delay** — kubelet syncs periodically, not instantly
3. **You edited the wrong ConfigMap** — verify it's the one mounted
4. **No redeploy triggered** — if the app needs a restart:
   ```bash
   kubectl rollout restart deployment <name>
   ```

**🔹 Best practices**
- Use watchers or `inotify` in your app to re-read files
- Env vars (`envFrom`/`env`) require a pod restart to take effect
- Add checksum annotations to trigger rollout on change (Helm):
  ```yaml
  annotations:
    checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
  ```

**🔹 Real-world example**
> *"A backend service read a YAML config from a ConfigMap. Even after changing it, behavior didn't change — the app loaded config at startup. We added a `rollout restart` step to the deployment job."*

**🔹 Summary Table**

| Reason | Description |
|--------|-------------|
| App reads config only once | File updated, app doesn't reload it |
| Kubelet delay | File sync happens every 1–2 minutes |
| Wrong ConfigMap | Editing a different resource |
| ConfigMap used as env vars | Requires pod restart |

<br>

---

### Q27. How does Node Affinity work in Kubernetes and when would you use it?

#### 📌 Short Explanation
Checks your understanding of how Kubernetes schedules pods on nodes based on custom rules, and when to control this using node affinity.

#### ✅ Answer
Node Affinity constrains which nodes a pod can be scheduled on, based on **node labels**. It's useful when workloads must run on specific node types — GPU nodes, SSD-backed nodes, or nodes in specific availability zones.

#### 📖 Detailed Explanation

**🔹 What is Node Affinity?**
Nodes carry labels such as `disktype=ssd`. Node affinity lets you require or prefer pods to run only on nodes with specific labels.

**🔹 Types**
1. **`requiredDuringSchedulingIgnoredDuringExecution`** — hard rule; the pod won't schedule unless matched (e.g., only on GPU nodes).
2. **`preferredDuringSchedulingIgnoredDuringExecution`** — soft rule; prefers a match but can fall back (e.g., prefer `us-east-1a`).

**🔹 Example (required affinity)**

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```
This pod is only scheduled on nodes labeled `disktype=ssd`.

**🔹 When would you use it?**

| Use Case | Why Use Node Affinity |
|----------|-----------------------|
| GPU workloads | Run pods only on GPU nodes (`gpu=true`) |
| Zone/locality awareness | Pin workloads to a specific zone |
| Storage constraints | Run only on SSD-backed nodes |
| Licensing/Compliance | Restrict workloads to labeled nodes |

**🔹 Real-world example**
> *"We had a mixed node pool — some SSD, some spinning disks. Our database pods needed fast disk. We labeled SSD nodes `disktype=ssd` and used nodeAffinity so they only ran there."*

**🔹 Common mistakes**
- Forgetting to label nodes
- Using `required` when you should use `preferred` (causing scheduling failures)
- Using `nodeSelector` for complex rules instead of `nodeAffinity`

**🔹 Summary Table**

| Type | Behavior | Use Case |
|------|----------|----------|
| `requiredDuringScheduling...` | Must meet label to be scheduled | Critical workloads |
| `preferredDuringScheduling...` | Prefer label but not mandatory | Best-effort distribution |

<br>

---

### Q28. What is the difference between Node Affinity and Node Label Selector (`nodeSelector`)?

#### 📌 Short Explanation
Differentiates two commonly used pod scheduling mechanisms. Though they serve similar purposes, their flexibility and use cases differ.

#### ✅ Answer
`nodeSelector` is a simple way to schedule pods on nodes with a specific label (equality only). `nodeAffinity` is more expressive — it supports multiple match conditions, operators like `In` / `NotIn`, and both required and preferred rules.

#### 📖 Detailed Explanation

**🔹 `nodeSelector` — the simpler way**

```yaml
spec:
  nodeSelector:
    disktype: ssd
```
- Only exact match (`key = value`)
- If no matching node exists, the pod won't schedule

**🔹 `nodeAffinity` — the advanced way** (Kubernetes 1.6+)

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```
- Operators: `In`, `NotIn`, `Exists`, `DoesNotExist`, etc.
- Hard (required) and soft (preferred) rules
- Better suited for complex or evolving scheduling logic

**🔹 Comparison Table**

| Feature | `nodeSelector` | `nodeAffinity` |
|---------|----------------|----------------|
| Supported operators | Only `=` | `In`, `NotIn`, `Exists`, `DoesNotExist`, etc. |
| Rule type | Only required | Required or Preferred |
| Multiple label match support | Limited | ✅ |
| Flexibility | Low | High |
| Readability for complex rules | Hard to maintain | Easier and more structured |

**🔹 Analogy**
> `nodeSelector` is a basic filter (*"I want apples"*). `nodeAffinity` is a full search query (*"I want either green or red apples, but prefer red"*).

**🔹 When to use what?**

| Use Case | Recommended |
|----------|-------------|
| Quick filtering with one label | `nodeSelector` |
| Multiple conditions or preferences | `nodeAffinity` |

<br>

---

### Q29. What is a Container Runtime in Kubernetes?

#### 📌 Short Explanation
Checks your understanding of one of the most foundational pieces in Kubernetes — the container runtime — and how it runs and manages containers within pods.

#### ✅ Answer
A container runtime is the software responsible for **running containers**. It pulls images, starts and stops containers, and manages their lifecycle on each node. Examples: **containerd**, **CRI-O**, and previously **Docker**.

#### 📖 Detailed Explanation

**🔹 What does a container runtime do?**
On every worker node, it:
- Pulls the container image from a registry (Docker Hub, ECR, etc.)
- Unpacks and runs the container in an isolated environment
- Reports container status back to the Kubelet
- Stops and removes containers as needed

**🔹 How does it fit in Kubernetes?**
The Kubelet interacts with the runtime via the **Container Runtime Interface (CRI)**:

```text
Kubelet → CRI → Container Runtime (e.g., containerd)
```

**🔹 Popular Container Runtimes**

| Runtime | Description |
|---------|-------------|
| **containerd** | Lightweight core runtime; default in most Kubernetes distributions |
| **CRI-O** | Purpose-built for Kubernetes; used by OpenShift and others |
| **Docker** | Previously used directly; deprecated in Kubernetes since v1.20+ |

> Kubernetes no longer requires Docker as a runtime — it uses containerd or CRI-O via CRI.

**🔹 Real-world example**
> *"In our EKS cluster, some pods were slow to start. We found containerd had image pull backoff issues. Restarting the containerd service and pre-pulling images fixed it."*

**🔹 Related concepts**
- **CRI:** interface between Kubelet and the runtime
- **OCI:** Open Container Initiative — defines container image and runtime standards

<br>

---

<br>

<a id="observability"></a>
# 📊 TOPIC 8: OBSERVABILITY

> 🚧 **Coming soon** — notes for this section are actively being updated.

<br>

---

<br>

<a id="aws"></a>
# ☁️ TOPIC 9: AWS

> 🚧 **Coming soon** — notes for this section are actively being updated.

<br>

---

<br>

<a id="azure"></a>
# 🔷 TOPIC 10: AZURE

> 🚧 **Coming soon** — notes for this section are actively being updated.

<br>

---

<br>

<a id="python"></a>
# 🐍 TOPIC 11: PYTHON

> Common DevOps packages and practical automation scripts.

<br>

---

### Q1. What are some common Python packages that you use as a DevOps Engineer?

#### 📌 Short Explanation
DevOps Engineers use Python for scripting, automation, cloud interactions, and infrastructure management. A set of well-known packages simplify OS operations, cloud APIs, configuration, monitoring, and CI/CD workflows.

#### ✅ Answer
`boto3`, `paramiko`, `requests`, `pyyaml`, `docker`, `kubernetes`, `fabric`, and `pytest`.

#### 📖 Detailed Explanation (with examples)

**1️⃣ `boto3` — AWS SDK for Python** (automate and manage AWS services)

```python
import boto3

ec2 = boto3.client('ec2')
response = ec2.describe_instances()
print(response)
```

**2️⃣ `paramiko` — SSH and remote command execution** (remote shell commands, SFTP)

```python
import paramiko

ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect(hostname='remote.server.com', username='user', password='pass')
stdin, stdout, stderr = ssh.exec_command('uptime')
print(stdout.read().decode())
ssh.close()
```

**3️⃣ `requests` — HTTP requests** (REST APIs and webhooks)

```python
import requests

response = requests.get('https://api.example.com/status')
print(response.status_code)
print(response.json())
```

**4️⃣ `pyyaml` — YAML parsing and generation** (Kubernetes manifests, config files)

```python
import yaml

with open('config.yaml', 'r') as file:
    config = yaml.safe_load(file)
print(config)
```

**5️⃣ `docker` — Docker SDK for Python** (containers, images, volumes)

```python
import docker

client = docker.from_env()
for container in client.containers.list():
    print(container.name, container.status)
```

**6️⃣ `kubernetes` — Kubernetes Python client** (create, delete, monitor resources)

```python
from kubernetes import client, config

config.load_kube_config()
v1 = client.CoreV1Api()
pods = v1.list_pod_for_all_namespaces()
for pod in pods.items:
    print(pod.metadata.name)
```

**7️⃣ `fabric` — High-level SSH command execution**

```python
from fabric import Connection

c = Connection('user@host')
c.run('uname -a')
```

**8️⃣ `pytest` — Testing framework** (infrastructure/config script tests)

```python
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5
```

<br>

---

### Q2. Write a Python script to run a container using the Docker SDK — the user provides the image name as input.

#### 📌 Short Explanation
Use the `docker` Python SDK to programmatically start a container. The script takes an image name as input, pulls the image if it doesn't exist locally, and runs a container from it.

#### ✅ Answer
Use `docker.from_env()` to connect to the local Docker engine and run the container with the provided image name.

#### 📖 Complete Script

```python
import docker

# Initialize Docker client
client = docker.from_env()

# Get image name from user input
image_name = input("Enter the Docker image name (e.g., nginx:latest): ").strip()

try:
    # Pull the image (if not present locally)
    print(f"Pulling image '{image_name}'...")
    client.images.pull(image_name)
    print(f"Image '{image_name}' pulled successfully.")

    # Run the container
    print(f"Running container from image '{image_name}'...")
    container = client.containers.run(image_name, detach=True)
    print(f"Container started with ID: {container.id[:12]}")

except docker.errors.ImageNotFound:
    print(f"Error: Image '{image_name}' not found.")
except docker.errors.APIError as e:
    print(f"Docker API error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

**🖥️ Example usage**

```text
Enter the Docker image name (e.g., nginx:latest): alpine
```
Expected output:
```text
Pulling image 'alpine'...
Image 'alpine' pulled successfully.
Running container from image 'alpine'...
Container started with ID: 3e1fabc2d789
```

> This script assumes Docker is installed and the user has permission to run Docker commands. It runs the container in the background (`detach=True`) without any specific command or port mapping.

<br>

---

### Q3. Write a Python script to fetch logs from a log website and print all logs with `404: Not Found`.

#### 📌 Short Explanation
Fetches logs from a publicly available log file, parses it line by line, and prints all entries that include the `404` HTTP status code ("Not Found").

#### ✅ Answer
Use Python's `requests` library to fetch the log file from a public URL and filter for lines containing `404`.

#### 📖 Complete Script

Sample Apache-style log source:
`https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs`

```python
import requests

# Publicly available Apache log sample
log_url = 'https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs'

try:
    # Fetch the log content
    response = requests.get(log_url)
    response.raise_for_status()
    logs = response.text.splitlines()

    print("Log lines with 404 Not Found:\n")

    # Search for lines with HTTP 404
    for line in logs:
        if ' 404 ' in line:
            print(line)

except requests.exceptions.RequestException as e:
    print(f"Error fetching logs: {e}")
```

**🔹 Example matching line**

```text
216.46.173.126 - - [27/May/2015:10:27:47 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1" 404 146
```

**🔹 What the script does**
- Fetches logs using `requests.get()`
- Splits the logs line by line
- Filters for lines containing ` 404 ` (with spaces) to avoid false matches in URLs or timestamps
- Prints all matching lines to the console

> 💡 You can modify the script to write filtered lines to a file or analyze other HTTP status codes like `500`, `403`, etc.

<br>

---

<div align="center">

**DevOps Interviews 🚀**

</div>
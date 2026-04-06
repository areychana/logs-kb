# Git & GitHub

---

## Part 1: How Git Actually Works Internally

Most people learn Git commands without understanding *what Git is doing*, so everything feels like magic (or black magic when it breaks). Let's fix that.

### Git is just a key-value store

At its core, Git stores your data as **objects** in a hidden folder called `.git` inside your repo. Every file, every folder, every commit, all of it is stored as an object with a unique ID called a **SHA hash** (a 40-character string like `a3f5c2d...`).

There are 3 types of objects:

**Blob** - stores the contents of a single file (no filename, just raw content)

**Tree** - stores a directory listing (maps filenames to blobs or other trees)

**Commit** - stores a snapshot. Points to a tree (the root of your project at that moment), has a message, author, timestamp, and a pointer to the *previous* commit

```
Commit Object
├── tree → a3f5c2d (root tree of your project)
├── parent → 9b2e1aa (previous commit)
├── author → Chana
├── date → 2026-04-02
└── message → "fix: handle null user"

Tree Object (a3f5c2d)
├── blob 8f1d3c2 → main.py
├── blob 2a9e7f1 → README.md
└── tree 4c3b1d9 → src/

Blob Object (8f1d3c2)
└── [raw contents of main.py]
```

So a commit isn't a "diff" - it's a **full snapshot** of your entire project at that moment, stored efficiently by reusing unchanged blobs.

### What is a branch, really?

A branch is just a **text file** containing one SHA hash of the latest commit on that branch. That's it. When you create a branch, Git creates a tiny file. When you commit, Git updates that file to point to the new commit.

```
.git/refs/heads/main        → points to commit C3
.git/refs/heads/feature-x   → points to commit C5
```

### What is HEAD?

`HEAD` is a pointer to whichever branch you're currently on. When you switch branches, HEAD moves. When you commit, the branch HEAD points to moves forward.

```
HEAD → main → C3 → C2 → C1
```

### The 3 areas you must understand

```
Working Directory   →  git add  →  Staging Area  →  git commit  →  Local Repo
(files on disk)                    (index/.git)                    (.git objects)
```

- **Working Directory** - your actual files. Edit them freely.
- **Staging Area (Index)** - a preview of your next commit. `git add` puts changes here.
- **Local Repo** - the permanent history. `git commit` writes here.

`git status` shows you the difference between all three areas.

---

## Part 2: Branching & Merging in Detail

### Creating and switching branches

```bash
git branch feature-login       # create branch
git checkout feature-login     # switch to it

# OR do both in one line:
git checkout -b feature-login
```

Internally: Git just creates a new pointer at the current commit. Your files don't change yet.

### What happens when you commit on a branch

```
Before:
main    → C3
feature → C3   (just created, same point)
HEAD    → feature

After committing on feature:
main    → C3
feature → C4   (moved forward)
HEAD    → feature
```

`main` hasn't moved. You've been working in parallel.

### Merging - two types

**Fast-forward merge** - happens when the branch you're merging into hasn't moved since you branched off. Git just slides the pointer forward. No merge commit created.

```
Before:              After git merge feature (from main):
main → C3            main → C4 → C3
         \
feature → C4
```

**3-way merge** - happens when both branches have new commits since they diverged. Git finds the common ancestor, compares both branches to it, and creates a new **merge commit** with two parents.

```
Before:
main    → C5 → C3
                \
feature → C4 → C3   (C3 is the common ancestor)

After:
main → M (merge commit) → C5, C4
```

---

## Part 3: Resolving Conflicts

A conflict happens when the **same lines** in the same file were changed differently on two branches. Git can't auto-decide which version to keep, so it asks you.

### What a conflict looks like in your file

```
<<<<<<< HEAD
return user.name or "Anonymous"
=======
return user.username
>>>>>>> feature-login
```

- Everything between `<<<<<<< HEAD` and `=======` is your current branch's version
- Everything between `=======` and `>>>>>>>` is the incoming branch's version

### How to resolve it

1. Open the file, read both versions
2. Decide what the final code should be (could be one, the other, or a mix)
3. Delete the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
4. Save the file
5. `git add <file>` to mark it resolved
6. `git commit` to complete the merge

### Tips to avoid conflicts

- Pull from main frequently while working on your branch (`git pull origin main`)
- Keep branches short-lived. the longer a branch lives, the more it diverges
- Communicate with teammates about who's touching what

---

## Part 4: Open Source Workflow (Fork → PR → Rebase)

This is the exact workflow for contributing to projects you don't own.

### Step by step

```
1. Fork on GitHub
   → Creates your own copy: github.com/areychana/some-project

2. Clone your fork locally
   git clone https://github.com/areychana/some-project.git
   cd some-project

3. Add the original repo as "upstream"
   git remote add upstream https://github.com/originalowner/some-project.git

   Now you have two remotes:
   origin   → your fork
   upstream → the original

4. Create a branch for your work
   git checkout -b fix/your-issue-name

5. Make changes, commit
   git add .
   git commit -m "fix: describe what you fixed"

6. Push to YOUR fork
   git push origin fix/your-issue-name

7. Open Pull Request on GitHub
   → Go to original repo → "Compare & pull request"
   → Fill description, link issue if any
   → Submit

8. Address review comments
   → Make changes locally → commit → push
   → PR updates automatically

9. Maintainer merges it ✓
```

### Keeping your fork up to date

While your PR is open, the original repo keeps moving. You need to sync:

```bash
git fetch upstream              # download latest from original
git checkout main
git merge upstream/main         # update your local main
git push origin main            # update your fork's main
```

### Rebase - what and why

`git merge` creates a merge commit and preserves the full history. `git rebase` *replays* your commits on top of another branch resulting in a cleaner, linear history.

```
Before rebase:
main    → C5 → C4 → C3
                     \
feature → F2 → F1

After git rebase main (from feature branch):
main    → C5 → C4 → C3
                          \
feature → F2' → F1'   (replayed on top of C5)
```

Your commits get rewritten with new SHAs (hence the `'`). The code is the same but the history is linear.

**When to use rebase:** Before opening a PR, rebase your branch on top of the latest main so there are no conflicts and the history is clean.

```bash
git fetch upstream
git rebase upstream/main
# resolve any conflicts during rebase if they appear
git push origin your-branch --force  # needed because history was rewritten
```

> ⚠️ Never rebase a branch that other people are also working on — rewriting shared history causes chaos.

---

## Part 5: GitHub Actions / CI-CD

### What is CI/CD?

**CI (Continuous Integration)** - automatically run tests every time someone pushes code or opens a PR. Catches bugs before they're merged.

**CD (Continuous Deployment)** - automatically deploy to production when code is merged to main.

GitHub Actions is GitHub's built-in automation system for this.

### How it works

You create a YAML file inside `.github/workflows/` in your repo. GitHub reads it and runs the defined steps automatically on triggers you specify.

### A simple example you should run tests on every PR

```yaml
# .github/workflows/test.yml

name: Run Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest
```

Every time someone opens a PR targeting `main`, GitHub spins up a fresh Ubuntu machine, runs these steps, and reports pass/fail on the PR. Maintainers often require this to pass before merging.

### Key concepts in Actions

**Trigger (`on`)** - what event starts the workflow. Common ones: `push`, `pull_request`, `schedule` (cron), `workflow_dispatch` (manual)

**Job** - a group of steps that run together on one machine

**Step** - a single command or reusable action

**Runner** - the machine that runs your job (`ubuntu-latest`, `windows-latest`, `macos-latest`)

**Action** - a reusable plugin (`uses: actions/checkout@v3` checks out your code)

**Secrets** - sensitive values (API keys, tokens) stored in GitHub settings, accessed as `${{ secrets.MY_KEY }}`

### A deploy example

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Deploy via SSH
        run: |
          ssh user@yourserver.com "cd /app && git pull && systemctl restart myapp"
        env:
          SSH_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

---

## How it all connects

```
You write code locally
       │
       │ git add + git commit
       ▼
  Local branch (feature-x)
       │
       │ git push origin feature-x
       ▼
  Your GitHub fork
       │
       │ Open Pull Request
       ▼
  Original Repo PR page
       │
       │ GitHub Actions runs tests automatically ✓
       │
       │ Maintainer reviews
       ▼
  Merged into main
       │
       │ GitHub Actions deploys to production ✓
       ▼
  Live app updated
```

# Git & GitHub

## Git vs GitHub

**Git** - version control system, runs locally, tracks changes over time.  
**GitHub** - cloud platform to host Git repos and collaborate.

Git is the tool. GitHub is the website that uses the tool.

---

## How Git Works Internally

Git stores everything as **objects** in `.git/`, each with a unique SHA hash.

Three object types:
- **Blob** - raw contents of a file (no filename)
- **Tree** - directory listing, maps filenames to blobs/trees
- **Commit** - snapshot pointing to a tree + parent commit + metadata

A commit is not a diff. It's a full snapshot, stored efficiently by reusing unchanged blobs.

**Branch** - just a text file containing one SHA (the latest commit). Nothing more.

**HEAD** - pointer to whichever branch you're currently on.

### The 3 areas
```
Working Directory → git add → Staging Area → git commit → Local Repo
```

- Working Directory: files on disk
- Staging Area (Index): preview of next commit
- Local Repo: permanent history in `.git/`

---

## Common Commands

| Command | What it does |
|---|---|
| `git init` | Start tracking a folder |
| `git clone <url>` | Download a repo |
| `git status` | See what changed |
| `git add .` | Stage all changes |
| `git commit -m "msg"` | Save a snapshot |
| `git push` | Send commits to GitHub |
| `git pull` | Get latest from GitHub |
| `git checkout -b <name>` | Create + switch to new branch |
| `git merge <branch>` | Merge branch into current |
| `git log` | See commit history |

---

## Branching & Merging

### Fast-forward merge
Happens when the base branch hasn't moved. Git just slides the pointer forward. No merge commit.

### 3-way merge
Happens when both branches have diverged. Git finds common ancestor, compares both, creates a merge commit with two parents.

---

## Resolving Conflicts

Conflict = same lines changed differently on two branches.

What it looks like in the file:
```
<<<<<<< HEAD
return user.name or "Anonymous"
=======
return user.username
>>>>>>> feature-login
```

How to resolve:
1. Open file, read both versions
2. Decide the final version
3. Delete the conflict markers
4. `git add <file>`
5. `git commit`

Tip: pull from main frequently to minimize conflicts.

---

## Open Source Workflow
```
Fork repo on GitHub
→ git clone your fork
→ git remote add upstream <original-url>
→ git checkout -b fix/your-branch
→ make changes → git add → git commit
→ git push origin fix/your-branch
→ Open Pull Request on original repo
→ Address review comments (commit + push, PR updates automatically)
→ Maintainer merges ✓
```

### Keep fork up to date
```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### Rebase vs Merge

**Merge** - preserves full history, creates a merge commit.  
**Rebase** - replays your commits on top of another branch, linear history.
```bash
git fetch upstream
git rebase upstream/main
git push origin your-branch --force
```

Use rebase before opening a PR to keep history clean.  
Never rebase a branch others are working on.

---

## GitHub Actions / CI-CD

**CI** - auto-run tests on every push or PR.  
**CD** - auto-deploy when code merges to main.

Workflows live in `.github/workflows/*.yml`.

### Example: run tests on every PR
```yaml
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
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest
```

### Key concepts

| Term | What it is |
|---|---|
| `on` | Trigger (push, pull_request, schedule, workflow_dispatch) |
| Job | Group of steps on one machine |
| Step | Single command or reusable action |
| Runner | Machine that runs the job (ubuntu-latest etc.) |
| Action | Reusable plugin (`uses: actions/checkout@v3`) |
| Secrets | Sensitive values stored in GitHub settings |

---

## Full Flow
```
write code locally
→ git add + git commit
→ git push origin feature-branch
→ open Pull Request
→ GitHub Actions runs tests ✓
→ maintainer reviews + merges
→ GitHub Actions deploys ✓
→ live app updated
```
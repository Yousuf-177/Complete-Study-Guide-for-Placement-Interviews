```
╔══════════════════════════════════════════════════════╗
║   GIT, GITHUB & VCS — Placement Interview Cheat Sheet ║
║   Subject: Version Control  |  Level: Placements/SDE  ║
╚══════════════════════════════════════════════════════╝
```

---

## ⚡ 1. CORE CONCEPTS (30-second recall)

- **Version Control System (VCS)** → tool that tracks changes to files over time, allowing collaboration & rollback.
- **Centralized VCS (CVCS)** → single central server holds history (e.g., SVN) — single point of failure.
- **Distributed VCS (DVCS)** → every user has a full copy of the repo & history locally (e.g., Git) — no single point of failure, works offline.
- **Git** → a free, open-source **distributed** version control system created by Linus Torvalds (2005).
- **GitHub** → a cloud-hosting platform for Git repositories, adding collaboration features (PRs, Issues, Actions) — Git itself has nothing to do with GitHub; GitHub is just one host among many (GitLab, Bitbucket).
- **Repository (Repo)** → a project's folder tracked by Git, containing all files + full history.
- **Commit** → a snapshot of changes at a point in time, with a unique SHA-1 hash ID, author, timestamp, and message.
- **Branch** → an independent line of development; a movable pointer to a commit.
- **HEAD** → pointer to the current branch/commit you're working on.
- **Working Directory** → the actual files you see/edit on disk.
- **Staging Area (Index)** → intermediate area where changes are prepared before committing.
- **Remote** → a version of the repo hosted elsewhere (e.g., on GitHub), referenced by name (commonly `origin`).
- **Clone** → copy a remote repo (with full history) to your local machine.
- **Fork** → create your own personal copy of someone else's repo on GitHub (server-side, not local).

---

## 📐 2. GIT ARCHITECTURE — THE 3 TREES

```
Working Directory  →  Staging Area (Index)  →  Local Repo (.git)  →  Remote Repo
     (edit)         git add        (stage)    git commit   (commit)   git push
```

- **Working Directory** → where you make edits; Git tracks these as "modified" until staged.
- **Staging Area** → snapshot of what will go into the next commit (`git add` moves changes here).
- **Local Repository** → committed history, stored in the hidden `.git` folder (`git commit` moves staged changes here permanently, locally).
- **Remote Repository** → shared copy on a server (GitHub/GitLab); `git push`/`git pull` sync local ↔ remote.

💡 **Mnemonic** → "**A**dd moves to stage, **C**ommit moves to local repo, **P**ush moves to remote" (A→C→P, in that order)

---

## 🧩 3. ESSENTIAL COMMANDS

### Setup & Config
```bash
git init                        # initialize a new local repo
git clone <url>                 # copy a remote repo locally
git config --global user.name "Name"
git config --global user.email "email@example.com"
```

### Basic Workflow
```bash
git status                      # see changed/staged/untracked files
git add <file>                  # stage a specific file
git add .                       # stage all changes
git commit -m "message"         # commit staged changes
git commit -am "message"        # add (tracked files only) + commit in one step
git log                         # view commit history
git log --oneline --graph       # compact visual history
git diff                        # see unstaged changes
git diff --staged               # see staged (not yet committed) changes
```

### Branching
```bash
git branch                      # list branches
git branch <name>               # create a new branch
git checkout <branch>           # switch to a branch
git checkout -b <name>          # create + switch in one step
git switch <branch>             # modern alternative to checkout for switching
git switch -c <name>            # modern alternative to checkout -b
git branch -d <name>             # delete a branch (safe, checks merge status)
git branch -D <name>             # force delete a branch
```

### Merging & Rebasing
```bash
git merge <branch>              # merge given branch into current branch
git rebase <branch>             # replay current branch's commits on top of given branch
git rebase -i HEAD~3             # interactive rebase (squash/edit/reorder last 3 commits)
```

### Remote Operations
```bash
git remote -v                   # list remotes
git remote add origin <url>     # link local repo to a remote
git fetch                       # download remote changes WITHOUT merging
git pull                        # fetch + merge (or rebase) in one step
git push                        # upload local commits to remote
git push -u origin <branch>     # push + set upstream tracking branch
```

### Undoing Changes
```bash
git restore <file>              # discard changes in working directory (modern)
git restore --staged <file>     # unstage a file (modern)
git checkout -- <file>          # discard changes (legacy syntax)
git reset --soft HEAD~1         # undo last commit, keep changes staged
git reset --mixed HEAD~1        # undo last commit, keep changes unstaged (default)
git reset --hard HEAD~1         # undo last commit, DISCARD changes completely
git revert <commit>             # create a NEW commit that undoes a previous commit (safe for shared history)
```

### Stashing
```bash
git stash                       # temporarily save uncommitted changes
git stash list                  # view all stashes
git stash pop                   # apply + remove most recent stash
git stash apply                 # apply most recent stash but keep it in stash list
git stash drop                  # delete a stash without applying
```

---

## 🔄 4. MERGE vs REBASE — the classic interview question

| Basis | Merge | Rebase |
|---|---|---|
| History | Preserves full history, creates a "merge commit" | Rewrites commit history, creates a linear sequence |
| Commit graph | Non-linear (shows branching/joining) | Linear (looks like changes happened sequentially) |
| Safety on shared branches | ✅ Safe — doesn't rewrite existing commits | ❌ Risky — rewrites commit hashes, avoid on shared/public branches |
| Use case | Preserving true history / collaborative branches | Cleaning up local commit history before merging (e.g., squashing) |

💡 **Golden Rule** → "Never rebase commits that have already been pushed and shared with others" — since rebase rewrites commit hashes, it can cause serious conflicts for anyone else who already has the old commits.

📝 **Visual difference:**
```
MERGE:                          REBASE:
main:    A---B---C---M          main:    A---B---C
                     /                            \
feature:    D---E---'           feature:            D'---E'  (replayed on top of C)
```

---

## ⚠️ 5. MERGE CONFLICTS

A **merge conflict** happens when Git can't automatically reconcile changes — usually because the same lines in the same file were changed differently in both branches being merged.

```
<<<<<<< HEAD
Your current branch's version
=======
Incoming branch's version
>>>>>>> feature-branch
```

**Resolution steps:**
1. Open the conflicted file(s), manually edit to keep the correct content, remove the `<<<<<<<`, `=======`, `>>>>>>>` markers.
2. `git add <file>` to mark it as resolved.
3. `git commit` (for merge) or `git rebase --continue` (for rebase) to finalize.

**Abort options:** `git merge --abort` or `git rebase --abort` to cancel and return to the pre-merge/rebase state.

---

## 📊 6. GIT RESET — THE THREE MODES (very commonly confused)

| Mode | Working Dir | Staging Area | Commit History | Use Case |
|---|---|---|---|---|
| `--soft` | Unchanged | Unchanged (still staged) | Moved back | Redo a commit message, combine commits |
| `--mixed` (default) | Unchanged | Reset (unstaged) | Moved back | Undo commit + staging, keep edits |
| `--hard` | **Reset (changes lost!)** | Reset | Moved back | Completely discard commit + changes |

💡 **Mnemonic** → "**S**oft = **S**tays staged, **M**ixed = **M**oves to working dir only, **H**ard = **H**istory AND changes gone"

### Reset vs Revert (another favorite comparison)
| Basis | Reset | Revert |
|---|---|---|
| Action | Moves branch pointer backward, can rewrite history | Creates a new commit that undoes changes |
| Safe for shared/pushed commits? | ❌ No (rewrites history) | ✅ Yes (adds new history, doesn't erase old) |
| Use case | Local cleanup before pushing | Undoing changes already shared with others |

---

## 🌳 7. GIT BRANCHING STRATEGIES

- **Git Flow** → structured model with `main` (production), `develop` (integration), `feature/*`, `release/*`, `hotfix/*` branches. Good for scheduled releases.
- **GitHub Flow** → simpler: `main` is always deployable, create a feature branch, open a PR, merge after review — used heavily in modern CI/CD workflows.
- **Trunk-Based Development** → everyone commits frequently (often directly) to a single main branch (`trunk`), using feature flags to hide incomplete work — favors fast integration over long-lived branches.

---

## 🗺️ 8. GITHUB-SPECIFIC CONCEPTS

- **Pull Request (PR)** → a request to merge changes from one branch/fork into another, enabling code review & discussion before merging.
- **Fork vs Clone** → Fork = server-side copy of a repo under your GitHub account (used for contributing to others' projects); Clone = local copy of any repo (yours or forked) onto your machine.
- **Issues** → GitHub's built-in tracker for bugs, feature requests, and tasks.
- **GitHub Actions** → CI/CD automation triggered by repo events (push, PR, schedule) defined in YAML workflow files.
- **README.md** → the landing page/documentation shown on a repo's homepage.
- **.gitignore** → file listing patterns of files/folders Git should NOT track (e.g., `node_modules/`, `.env`).
- **License file** → defines usage rights/permissions for the repo's code.
- **GitHub Pages** → free static site hosting directly from a repo.

---

## 📝 9. .gitignore & COMMON PATTERNS

```gitignore
# Common .gitignore entries
node_modules/
*.log
.env
__pycache__/
*.pyc
.DS_Store
dist/
build/
```

💡 Note: `.gitignore` only prevents **untracked** files from being tracked. If a file is already tracked, you must first run `git rm --cached <file>` before adding it to `.gitignore`.

---

## 🔍 10. INSPECTING HISTORY

```bash
git log --author="name"          # commits by a specific author
git log --since="2 weeks ago"    # commits within a time range
git log -p <file>                # show diffs for each commit affecting a file
git blame <file>                 # see who last modified each line & in which commit
git show <commit-hash>           # show details/diff of a specific commit
git diff <branch1> <branch2>     # compare two branches
git cherry-pick <commit-hash>    # apply a specific commit from another branch onto current branch
```

---

## 📊 11. FETCH vs PULL vs CLONE

| Command | What it does |
|---|---|
| `git clone` | Downloads a full copy of a remote repo (history + files) for the first time |
| `git fetch` | Downloads new commits/branches from remote, but does NOT merge them into your working branch |
| `git pull` | `git fetch` + automatically merges (or rebases with `--rebase`) into your current branch |

💡 **Why prefer fetch+review over pull sometimes?** `fetch` lets you inspect incoming changes (`git log origin/main`) before deciding to merge, giving more control than `pull`'s automatic merge.

---

## ⚠️ 12. COMMON EXAM/INTERVIEW TRAPS

- ❌ Wrong: Git and GitHub are the same thing → ✅ Right: Git is the version control *tool*; GitHub is a cloud *hosting platform* for Git repos (one of several, alongside GitLab/Bitbucket).
- ❌ Wrong: `git pull` and `git fetch` do the same thing → ✅ Right: `fetch` only downloads; `pull` downloads AND merges automatically.
- ❌ Wrong: Rebase is always better than merge → ✅ Right: Rebase creates cleaner linear history but is DANGEROUS on shared/pushed branches since it rewrites commit hashes.
- ❌ Wrong: `git reset --hard` is safe to undo a commit → ✅ Right: It permanently discards uncommitted working directory changes too — use with caution.
- ❌ Wrong: Fork and Clone are the same → ✅ Right: Fork = server-side copy on GitHub under your account; Clone = local copy on your machine.
- ❌ Wrong: `.gitignore` removes already-tracked files → ✅ Right: It only prevents NEW untracked files from being added; already-tracked files need `git rm --cached`.
- ❌ Wrong: A commit hash is sequential/incremental → ✅ Right: Commit hashes are SHA-1 hashes based on content+metadata, not sequential numbers.
- ❌ Wrong: `git revert` deletes the commit from history → ✅ Right: `revert` creates a NEW commit that undoes changes; the original commit still exists in history (safe for shared branches).

---

## 🃏 13. MNEMONICS & MEMORY TRICKS

- 💡 **Workflow order** → "**A**dd, **C**ommit, **P**ush" = A-C-P, moving changes progressively further from your local edits toward the shared remote.
- 💡 **Reset modes** → "Soft Stays staged, Mixed moves to working dir, Hard Hurts (loses changes)"
- 💡 **Merge vs Rebase** → "Merge = Marries histories together (keeps both parents); Rebase = Replays your commits like retelling a story in order"
- 💡 **Fetch vs Pull** → "Fetch = Look before you leap; Pull = Fetch + leap (merge) automatically"
- 💡 **Fork vs Clone** → "Fork = Friend's copy on GitHub (server); Clone = Copy onto your Computer (local)"

---

## 📝 14. TOP INTERVIEW Q&A (rapid fire)

**Q: What is the difference between Git and GitHub?**
📝 Git is a distributed version control *tool* that runs locally and tracks changes; GitHub is a *cloud platform* that hosts Git repositories and adds collaboration features like PRs and Issues.

**Q: What is the difference between `git merge` and `git rebase`?**
📝 Merge combines two branches by creating a new merge commit, preserving full non-linear history. Rebase replays your branch's commits on top of another branch, producing a linear history — but it rewrites commit hashes, so it's unsafe on already-shared/pushed branches.

**Q: What happens during a merge conflict, and how do you resolve it?**
📝 Git can't auto-merge because the same lines were changed differently in both branches; you manually edit the file to resolve the `<<<<<<<`/`=======`/`>>>>>>>` markers, then `git add` and commit/continue.

**Q: Difference between `git reset` and `git revert`?**
📝 Reset moves the branch pointer backward and can discard history (unsafe if already pushed/shared); revert creates a new commit that undoes changes, preserving history (safe for shared branches).

**Q: What is a detached HEAD state?**
📝 Occurs when you check out a specific commit (not a branch) — HEAD points directly to that commit instead of a branch reference. Any new commits made here aren't attached to any branch and can be lost unless you create a new branch from that point.

**Q: How would you undo the last commit but keep the changes?**
📝 `git reset --soft HEAD~1` (keeps changes staged) or `git reset --mixed HEAD~1` (keeps changes unstaged, working directory intact).

**Q: What's the purpose of `.gitignore`?**
📝 Prevents specified files/folders (like build artifacts, secrets, dependencies) from being tracked by Git, keeping the repo clean and avoiding accidental commits of sensitive/generated files.

**Q: What is a Pull Request and why is it useful?**
📝 A PR proposes merging changes from one branch/fork into another, enabling team code review, automated CI checks, and discussion before the changes become part of the main codebase.

---

## 🎯 15. LAST-MINUTE INTERVIEW TIPS

1. Be ready to **explain the 3-tree architecture** (Working Directory → Staging → Repository) — this underlies almost every Git command's behavior.
2. Practice explaining **merge vs rebase with a diagram** — this is one of the most commonly asked Git questions in interviews.
3. Know the **exact difference between reset's three modes** (`--soft`, `--mixed`, `--hard`) — a favorite "what would happen if you ran X" trick question.
4. Be ready to **walk through resolving a merge conflict** step by step, not just define what it is.
5. If asked about your own project experience, be ready to explain **your actual branching strategy** (feature branches + PRs is the safest, most common real-world answer).
6. Mention practical GitHub features you've used — Issues, PRs, Actions/CI — to show hands-on experience beyond just command-line Git.
7. If unsure of exact command syntax, **explain the underlying concept first** — interviewers value understanding of what Git is doing internally over rote command memorization.

---

## 🔑 16. ONE-GLANCE SUMMARY BOX

```
┌───────────────────────────────────────────────────────────┐
│  🔑 MUST-KNOW ESSENTIALS — GIT/GITHUB PLACEMENT             │
│  1. Git = distributed VCS tool | GitHub = cloud host for it │
│  2. 3 trees: Working Dir → Staging(add) → Local Repo(commit)│
│     → Remote(push)                                          │
│  3. Merge = preserves history, new merge commit              │
│     Rebase = linear history, rewrites hashes (unsafe shared)│
│  4. Reset: soft(keep staged) / mixed(keep unstaged) /        │
│     hard(discard everything)                                 │
│  5. Revert = safe undo via new commit (shared branches OK)  │
│  6. Fetch = download only | Pull = fetch + merge              │
│  7. Fork = server-side copy(GitHub) | Clone = local copy     │
│  8. .gitignore = stops NEW files from being tracked only     │
│  9. PR = propose merge + enable code review                  │
│  10. Never rebase/force-push commits already shared/pushed   │
└───────────────────────────────────────────────────────────┘
```

---

*Want me to go deeper on any section (e.g., a detailed merge-conflict-resolution walkthrough, Git internals like objects/blobs/trees, or a mock Git interview Q&A round), or create a practice quiz to test yourself?*

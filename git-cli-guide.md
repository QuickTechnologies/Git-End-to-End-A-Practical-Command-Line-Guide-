# Git, End-to-End: A Practical, Command-Line Guide (with Explanations)

> This guide takes you from “what is Git?” through real-world workflows: creating a repo, branching, merging, pushing, pulling, and opening pull requests. It includes copy-pasteable commands, clear explanations, and practical tips—especially for Windows users.

---

## 0) Mental Model & Glossary (High-Level)

- **Git**: A distributed version control system. Every clone is a full copy of the repository (history + files).
- **Repository (repo)**: A project folder tracked by Git. Contains your code and a hidden `.git/` directory with history.
- **Commit**: A snapshot of your files with a message. Commits form a history (DAG).
- **Branch**: A movable pointer to a line of development. Common default is `main`.
- **Remote**: A hosted copy of the repo (e.g., GitHub, GitLab, Bitbucket). Usually named `origin`.
- **Push**: Send your local commits to a remote repo/branch.
- **Pull**: Fetch + merge (or rebase) the remote changes into your local branch.
- **Fetch**: Download new commits/branches from remote—no merge.
- **Merge**: Combine histories; creates a merge commit (unless fast-forward).
- **Rebase**: Reapply your commits onto another base; yields a linear history.
- **Pull Request (PR) / Merge Request (MR)**: A platform feature (GitHub/GitLab) proposing to merge one branch into another; enables code review and CI checks.
- **Stash**: Temporary shelf for uncommitted changes.
- **HEAD**: Your current checked-out commit/branch pointer.

---

## 1) One-Time Setup

### 1.1 Install Git & Identify Yourself
```bash
git --version

# Configure your identity (used in commit metadata)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Use main as the default initial branch
git config --global init.defaultBranch main

# Colorized output and helpful defaults
git config --global color.ui auto
git config --global pull.rebase false   # or true if you prefer a rebase workflow
```

### 1.2 Windows Line Endings (CRLF vs LF)
```bash
# Safe default on Windows: check out CRLF, commit LF
git config --global core.autocrlf true
```
For cross-platform repos, add a `.gitattributes` with line-ending rules to the repo itself (see §11.6).

### 1.3 Authentication (GitHub as example)
**HTTPS + Personal Access Token (easiest):**
```bash
# When prompted for password during push, use a GitHub personal access token.
# Windows Credential Manager can securely remember it.
```

**SSH (fast, no tokens once set):**
```bash
# Generate a key (Ed25519)
ssh-keygen -t ed25519 -C "you@example.com"

# Start the agent and add key (Windows Git Bash)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy the public key and add to GitHub > Settings > SSH and GPG keys
cat ~/.ssh/id_ed25519.pub
```
If you ever see `Permission denied (publickey).`, your SSH key isn’t being used/recognized—re-add to agent and confirm it’s registered on the hosting platform.

---

## 2) Create or Clone a Repository

### 2.1 Start a New Local Repo
```bash
mkdir my-project && cd my-project
git init
echo "# My Project" > README.md
echo ".venv/" >> .gitignore        # ignore Python venv
echo "__pycache__/" >> .gitignore  # ignore Python bytecode
git add .
git commit -m "Initial commit"
```

### 2.2 Connect to a Remote and Push
```bash
# Add remote (SSH URL shown; HTTPS works too)
git remote add origin git@github.com:youruser/my-project.git

# If you need to change it later:
# git remote set-url origin NEW_URL

# Push initial branch and set upstream
git push -u origin main
```
If you get `remote origin already exists`, either change it:
```bash
git remote set-url origin git@github.com:youruser/my-project.git
```
…or remove & re-add:
```bash
git remote remove origin
git remote add origin git@github.com:youruser/my-project.git
```

### 2.3 Clone an Existing Repo
```bash
git clone git@github.com:yourorg/existing-repo.git
cd existing-repo
```

---

## 3) Daily Dev Workflow: Status → Edit → Stage → Commit

```bash
git status                     # see what's changed
git diff                       # see unstaged changes
git add path/to/file.py        # stage a file
git add .                      # or stage all changes carefully
git commit -m "Explain what & why, not how"
```

**Commit message tips**
- First line: ≤ 50 chars, imperative mood (“Add X”, “Fix Y”).
- Blank line, then details if needed (wrap at ~72 chars).

---

## 4) Branching

### 4.1 Create & Switch Branches
```bash
git branch                     # list local branches
git branch -r                  # list remote branches
git switch -c feature/login    # create & switch (new)
# or: git checkout -b feature/login

git switch main                # switch back
# or: git checkout main
```

### 4.2 Tracking Upstream for New Branches
```bash
git push -u origin feature/login  # set upstream so future push/pull is shorter
```

---

## 5) Syncing with Remote: Fetch, Pull, Push

```bash
git fetch                      # download updates, no merge
git pull                       # fetch + merge (or rebase if configured)
git push                       # upload local commits
```

**Configure pull behavior (merge vs rebase):**
```bash
git config --global pull.rebase false  # merge (default)
# or:
git config --global pull.rebase true   # rebase
```

---

## 6) Merging & Conflict Resolution

### 6.1 Merge another branch into your current branch
```bash
git switch main
git pull
git merge feature/login
# Resolve conflicts if prompted:
#   - Open files, find conflict markers <<<<<<< ======= >>>>>>>
#   - Edit to desired state, then:
git add conflicted_file.py
git commit   # completes the merge
```

### 6.2 Fast-Forward Merge (no merge commit if linear)
```bash
git merge --ff-only feature/login
```

---

## 7) Rebasing (Optional but Powerful)

**Reapply your branch on top of the latest `main` to keep a linear history:**
```bash
git switch feature/login
git fetch origin
git rebase origin/main
# Resolve conflicts (edit files), then:
git add .
git rebase --continue

# If you regret the rebase:
git rebase --abort
```

**Interactive rebase for squashing/fixing messages:**
```bash
git rebase -i HEAD~5
# Pick/squash/reword in the editor, save & exit.
```
> **Note**: Avoid rebasing shared branches others are already using (it rewrites history).

---

## 8) Stashing Work in Progress

```bash
git stash push -m "WIP: partial parser"
# Do something else...
git stash list
git stash show -p stash@{0}
git stash pop                 # reapply and remove from stash
# or: git stash apply         # reapply but keep stash
```

---

## 9) Undo & Recovery

- **Unstage a file** (keep changes):
  ```bash
  git restore --staged path/to/file
  ```
- **Discard local changes** (dangerous):
  ```bash
  git restore path/to/file
  ```
- **Reset branch to a previous commit**:
  ```bash
  # Soft: keep changes staged
  git reset --soft <commit>
  # Mixed (default): keep changes unstaged
  git reset <commit>
  # Hard: discard changes (danger!)
  git reset --hard <commit>
  ```
- **Reflog** (powerful “time machine” of HEAD moves):
  ```bash
  git reflog
  git checkout <reflog-hash>
  # or recover a lost branch/commit via reset/branch from reflog
  ```

---

## 10) Pull Requests (PRs) / Merge Requests (MRs)

> PRs/MRs are **not** pure Git; they’re a feature of hosting platforms.

Typical steps:
1. Create a feature branch:
   ```bash
   git switch -c feature/login
   # code, commit, push
   git push -u origin feature/login
   ```
2. Open a PR/MR **on the platform’s UI** to merge `feature/login` into `main`.
3. Pass code review & CI checks, then merge via the platform.

**Optional (GitHub CLI)**
```bash
# Install GitHub CLI: https://cli.github.com/
gh auth login
gh pr create --base main --head feature/login --title "Add login" --body "Details..."
gh pr status
gh pr view --web
gh pr merge --squash --delete-branch
```

---

## 11) Practical Essentials You’ll Use Often

### 11.1 .gitignore
Ignore generated files, venvs, caches, secrets:
```
# Python
.venv/
__pycache__/
*.pyc
.env
.env.*
dist/
build/
*.egg-info/
```
```bash
git rm -r --cached .      # if you accidentally committed ignored files
git add .
git commit -m "Apply .gitignore"
```

### 11.2 Remotes
```bash
git remote -v
git remote add origin <URL>
git remote set-url origin <NEW_URL>
```

### 11.3 Tags (for releases)
```bash
git tag -a v1.0.0 -m "First stable release"
git push origin v1.0.0
git tag                    # list tags
```

### 11.4 Differences & Blame
```bash
git diff                   # unstaged changes
git diff --staged          # staged vs HEAD
git log --oneline --graph --decorate --all
git blame path/to/file     # who last changed each line
```

### 11.5 Clean Untracked Files (dangerous)
```bash
git clean -fd              # remove untracked files/dirs
git clean -n               # dry-run first
```

### 11.6 .gitattributes (line endings, binary files, diff rules)
```
# Normalize line endings to LF in repo, convert to CRLF on Windows checkout
* text=auto

# Treat these as binary (no diffs)
*.png binary
*.jpg binary
```

### 11.7 Commit Signing (optional)
```bash
# After configuring GPG/SSH signing:
git config --global commit.gpgsign true
git config --global user.signingkey <KEY_ID>
```

### 11.8 Git LFS (Large Files)
```bash
# Install Git LFS, then:
git lfs install
git lfs track "*.psd"
git add .gitattributes
git add path/to/large.psd
git commit -m "Track large assets with LFS"
```

---

## 12) Branching Strategies (Pick One for Your Team)

- **Trunk-Based**: Small, frequent merges into `main`; feature flags for incomplete work. Simple & popular.
- **Git Flow**: `main` + `develop` with long-lived branches and release/hotfix branches. Heavier process.
- **GitHub Flow**: Branch from `main`, open PR, merge back to `main`, deploy often. Lightweight and common.

For solo or small teams, **GitHub Flow** or **Trunk-Based** is usually best.

---

## 13) Advanced but Useful

### 13.1 Cherry-Pick (grab a specific commit)
```bash
git switch main
git cherry-pick <commit-sha>
```

### 13.2 Bisect (find the commit that introduced a bug)
```bash
git bisect start
git bisect bad              # current commit is bad
git bisect good <good-sha>  # known good commit
# Test state at each step, then mark:
git bisect good | bad
# When done:
git bisect reset
```

### 13.3 Worktrees (multiple working dirs for one repo)
```bash
git worktree add ../proj-fix bugfix/issue-123
# Now you have a second checkout of the repo on a different branch
```

### 13.4 Sparse Checkout (partial clone of large repos)
```bash
git sparse-checkout init --cone
git sparse-checkout set src/ docs/
```

### 13.5 Submodules vs Subtrees
- **Submodule**: Links to another repo at a fixed commit.
  ```bash
  git submodule add <URL> external/libfoo
  git submodule update --init --recursive
  ```
- **Subtree**: Vendor code copied into the repo with history; simpler for consumers.

---

## 14) Typical End-to-End Example (Feature → PR → Merge)

```bash
# 1) Update your base
git switch main
git pull

# 2) Create a feature branch
git switch -c feature/parser

# 3) Work, stage, commit
git status
git add src/parser.py
git commit -m "Add initial parser for MWV/MWD/MDA/XDR"

# 4) Sync with remote and open PR
git push -u origin feature/parser

# (Open PR on platform or use gh pr create ...)
# Review, address comments, push more commits if needed

# 5) After merge on platform:
git switch main
git pull           # now includes merged feature/parser
git branch -d feature/parser
git push origin --delete feature/parser   # delete remote branch
```

---

## 15) Tips, Tricks, and Good Habits

- **Small commits with clear messages**: Easier review and rollback.
- **Review your diff before committing**: `git diff --staged`.
- **Consistent branch names**: `feature/*`, `bugfix/*`, `hotfix/*`.
- **Protect main on remote**: Require PRs and passing CI checks.
- **Keep secrets out of Git**: Use `.gitignore`, `.env`, or secret managers. If you accidentally commit a secret, rotate it.
- **Avoid committing venvs and build artifacts**: Use `.gitignore`.
- **Rebase for local cleanup, merge for shared history**: Prefer squash merge in PRs if your team likes a clean history.
- **Windows CRLF gotchas**: Use `.gitattributes` for consistency.
- **When stuck, `git status` & `git log` are your compass`.
- **Backups**: `git push` frequently; your remote is a safety net.
- **Aliases** (speed up common commands):
  ```bash
  git config --global alias.st "status -sb"
  git config --global alias.lg "log --oneline --graph --decorate --all"
  git config --global alias.co switch
  git config --global alias.br branch
  git config --global alias.ci commit
  git config --global alias.df "diff --word-diff"
  ```

---

## 16) Common Errors You’ll See (and Fixes)

- **`Permission denied (publickey).`**
  - SSH key not loaded or not added to GitHub.
  - Fix: `ssh-add ~/.ssh/id_ed25519` and ensure the public key is in GitHub.
- **`fatal: remote origin already exists.`**
  - You already added `origin`. Use `git remote set-url origin <URL>`.
- **Push rejected (non-fast-forward)**:
  - Your remote has new commits. `git pull --rebase` (or `git pull`), resolve, then `git push`.
- **Detached HEAD**:
  - You checked out a commit instead of a branch. Make a branch to keep changes:
    ```bash
    git switch -c hotfix/off-by-one
    ```

---

## 17) Quick “Cheat Sheet” (Copy-Paste)

```bash
# Create repo
git init
git add .
git commit -m "Initial commit"
git remote add origin <URL>
git push -u origin main

# Clone repo
git clone <URL>
cd repo

# Branching
git switch -c feature/x
git switch main
git branch -d feature/x

# Stage & commit
git status
git add <file>
git commit -m "Message"

# Sync
git fetch
git pull
git push

# Merge
git switch main
git merge feature/x

# Rebase
git switch feature/x
git fetch origin
git rebase origin/main

# Stash
git stash push -m "WIP"
git stash list
git stash pop

# Undo
git restore --staged <file>
git restore <file>
git reset --hard HEAD~1

# Inspect
git log --oneline --graph --decorate --all
git diff
git diff --staged
git blame <file>

# Tags
git tag -a v1.0.0 -m "Release"
git push origin v1.0.0

# Recovery
git reflog
```

---

### Final Note
If you’d like, I can tailor this to your exact environment (e.g., Windows + VS Code + GitHub) and generate a ready-to-use `.gitignore`, `.gitattributes`, and a set of `git config` defaults that match your Python projects and branching preferences.

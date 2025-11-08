# Git Essentials

## 🎯 Summary

Frequently used Git commands and concepts in practice

---

## 📌 Key Points

### 1. fetch vs pull

**Core concept**: `pull` is **merging**, not overwriting

| Command | Action | Analogy |
|---------|--------|---------|
| **fetch** | Only get remote changes | Receive package 📦 |
| **pull** | Get + merge with my code | Receive & organize package 📦➡️📚 |

#### A. fetch (Safe)
```bash
# Get remote changes (doesn't touch my code)
git fetch origin main

# Check changes
git log origin/main

# Merge if okay
git merge origin/main
```

#### B. pull (Fast)
```bash
# fetch + merge in one command
git pull origin main
```

**Misconception**: "Will pull delete my code?" ❌
**Truth**: pull **combines** remote and local code

---

### 2. Basic Workflow

```bash
# 1. Check status
git status

# 2. Add files
git add .

# 3. Commit
git commit -m "feat: add financial calendar crawler"

# 4. Push to remote
git push origin main
```

---

### 3. Branch Workflow

```bash
# Create and switch to branch
git checkout -b feature/calendar-crawler

# Work and commit
git add .
git commit -m "feat: implement crawler"

# Switch to main
git checkout main

# Get latest changes
git pull origin main

# Merge branch
git merge feature/calendar-crawler

# Push
git push origin main
```

---

### 4. Conflict Resolution

**When conflict occurs**:
```bash
git pull origin main
# Conflict!
```

**Resolution**:
1. Open conflicted file
```python
<<<<<<< HEAD
# My code
def crawl():
    print("my version")
=======
# Remote code
def crawl():
    print("remote version")
>>>>>>> origin/main
```

2. Choose version (or combine)
```python
def crawl():
    print("merged version")
```

3. Commit
```bash
git add .
git commit -m "fix: resolve merge conflict"
git push origin main
```

---

### 5. Useful Commands

#### A. Check Changes
```bash
# List changed files
git status

# Show detailed changes
git diff

# Commit history
git log
git log --oneline  # Simplified
```

#### B. Undo Mistakes
```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo changes (⚠️ dangerous)
git reset --hard HEAD~1

# Undo file changes
git checkout -- file.py
```

---

### 6. Atomic Commit (Small Unit Commits)

**Core Principle**: One commit = One logical change

#### Why Commit Small?

**Problem**:
```bash
# ❌ 3 days of work in one commit
git add .
git commit -m "3 days work done"

→ Rollback loses 3 days of work
→ Code review impossible (+500 lines)
→ Merge conflicts nightmare
→ Contribution unclear
```

**Solution**:
```bash
# ✅ Commit in small units frequently
git add src/components/LoginButton.tsx
git commit -m "feat: add login button component"

# 30min~1hr later
git add src/api/auth.ts
git commit -m "feat: implement login API function"

# Bug found → Fix immediately
git add src/components/LoginButton.tsx
git commit -m "fix: add loading state to login button"
```

#### Commit Frequency

**Typical developer**: 5-10 commits per day
- Each feature completion
- Each bug fix
- Each file addition

**Real Example**:
```bash
09:00 - Build login form UI
09:30 - git commit "feat: implement login form component"

10:30 - Connect API
11:00 - git commit "feat: integrate login API"

11:30 - Fix bug
11:45 - git commit "fix: add validation for empty email"

14:00 - Write tests
15:00 - git commit "test: add login form unit tests"
```

---

### 7. File Specification and Push Strategy

#### A. Avoid git add .

**Problem**:
```bash
# ❌ Add all files indiscriminately
git add .
git commit -m "work done"

→ Commits unnecessary files (.env, node_modules, etc.)
→ Unrelated changes in one commit
```

**Solution**:
```bash
# ✅ Specify files
git add src/components/Dashboard.tsx
git commit -m "feat: add dashboard layout"

git add src/api/stock.ts
git commit -m "feat: add stock data API"

# Or interactive selection
git add -p  # Select changes interactively
```

#### B. Push Timing

**Option 1: Push by feature (Recommended)**
```bash
# After 2-3 related commits
git commit -m "feat: implement login form"
git commit -m "feat: integrate login API"
git commit -m "test: add login tests"
git push origin main
```

**Option 2: 1-2 times per day**
```bash
# Before lunch
git push origin main

# Before leaving (Required)
git push origin main
```

**Option 3: After each commit (Team collaboration)**
```bash
git commit -m "feat: new feature"
git push origin main  # Push immediately
```

**Key**: Push at least once per day

---

### 8. Commit Message Convention

**Format**: `<type>: <description>`

| Type | Meaning | Example |
|------|---------|---------|
| **feat** | New feature | `feat: add financial calendar crawler` |
| **fix** | Bug fix | `fix: update popup removal logic` |
| **refactor** | Refactoring | `refactor: separate API service layer` |
| **docs** | Documentation | `docs: update README` |
| **test** | Testing | `test: add crawler tests` |
| **chore** | Miscellaneous | `chore: update package versions` |

**Bad examples**:
```bash
git commit -m "update"
git commit -m "fix bug"
git commit -m "git add ."
git commit -m "3 days work"
```

**Good examples**:
```bash
git commit -m "feat: implement Selenium crawler"
git commit -m "fix: update jQuery Datepicker manipulation logic"
git commit -m "refactor: improve model design (separate DateField)"
git commit -m "test: add crawler API unit tests"
```

---

## 💡 Why I Learned This

**Zerothon 2025 Hackathon**:
- Immature Git branching strategy caused commit history issues
- One team member batch-uploaded all code with `final commit`
- Individual contributions unclear

**Lessons**:
- Realized importance of Atomic Commits
- Small frequent commits make contributions clear
- File specification for add, meaningful commit messages essential

**Improvements**:
- Applied small unit commits starting from QuantrumAI project
- 5-10 commits per day, daily push
- Switched to PR-based collaboration

---

## 🔧 Practical Application

### Dipping Financial Calendar Development
1. Created `feature/calendar-crawler` branch
2. Implemented crawler and committed
3. Merged to main branch
4. Resolved conflicts when they occurred

---

## 🔗 Related Projects

- [Dipping Financial Calendar Crawler](../../Projects/QuantrumAI/dipping-calendar-crawler/): Real-world Git usage

---

## ❓ Interview Prep Questions

**Q1: What's the difference between fetch and pull?**
> A: fetch only retrieves remote changes without touching local code. pull performs fetch + merge in one command, combining remote changes with local code. Use fetch for safe checking, pull for quick updates.

**Q2: How often should you commit?**
> A: Each time a logical change is complete. Typically 5-10 commits per day is appropriate. Committing 3 days of work at once makes rollback difficult and contributions unclear. After experiencing this issue in the Zerothon project, I developed the habit of committing frequently in small units.

**Q3: Is it okay to use git add .?**
> A: Should be avoided. It can commit unnecessary files (.env, node_modules, etc.) or unrelated changes together. Safer to specify files or use git add -p for interactive selection.

**Q4: What makes a good commit message?**
> A: It should clearly state "what and why." Add types like `feat:`, `fix:` and write concisely. Avoid vague messages like "update" or "fix."

**Q5: When should you push?**
> A: At least once per day. Push after 2-3 related commits accumulate, or for team collaboration, push immediately after each commit. Pushing before leaving is mandatory.

**Q6: What's the difference between reset --soft and --hard?**
> A: `--soft` only undoes the commit while keeping changes. `--hard` undoes both commit and changes irreversibly. Generally `--soft` is safer.

---

## 📚 Core Commands Cheat Sheet

```bash
# Basics
git status                 # Check status
git add .                  # Add all changes
git commit -m "message"    # Commit
git push origin main       # Push

# Sync
git fetch origin main      # Get remote
git pull origin main       # Get + merge

# Branches
git branch                 # List branches
git checkout -b feat/xxx   # Create and switch
git merge feat/xxx         # Merge branch

# History
git log                    # Commit history
git log --oneline          # Simplified view
git diff                   # Show changes

# Undo
git reset --soft HEAD~1    # Undo last commit (safe)
git checkout -- file.py    # Undo file changes
```

---

**Core Principles**:
1. `pull` is merging, not overwriting
2. **Commit small and often** (5-10 per day)
3. **Avoid git add .**, specify files
4. **Write clear commit messages** (feat, fix, docs, etc.)
5. **Push at least once per day** (before leaving mandatory)
6. Separate features with branches
7. Always pull before push

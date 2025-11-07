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

### 6. Commit Message Convention

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
git commit -m "asdf"
```

**Good examples**:
```bash
git commit -m "feat: implement Selenium crawler"
git commit -m "fix: update jQuery Datepicker manipulation logic"
git commit -m "refactor: improve model design (separate DateField)"
```

---

## 💡 Why I Learned This

**Dipping project** needed:
- Share code with team (push/pull)
- Feature branch workflow
- Conflict resolution

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

**Q2: Does pull delete my code?**
> A: No. Pull merges remote changes with local code. If conflicts occur, Git notifies you and you can resolve them manually.

**Q3: When do conflicts occur?**
> A: When the same part of the same file is modified differently by remote and local. Git cannot automatically merge and requires manual resolution.

**Q4: What makes a good commit message?**
> A: It should clearly state "what and why." Add types like `feat:`, `fix:` and write concisely. Avoid vague messages like "update" or "fix."

**Q5: What's the difference between reset --soft and --hard?**
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
2. Commit small and often
3. Write clear commit messages
4. Separate features with branches
5. Always pull before push

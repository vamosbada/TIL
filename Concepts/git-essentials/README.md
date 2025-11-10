# Git Essentials

## 🎯 Summary

Frequently used Git commands and concepts in practice

---

## 📌 Key Points

### 1. fetch vs pull

| Command | Action |
|---------|--------|
| **fetch** | Get remote changes only (doesn't touch local code) |
| **pull** | Get + merge with local code (fetch + merge) |

```bash
# fetch (safe)
git fetch origin main
git log origin/main  # Check
git merge origin/main  # Merge if okay

# pull (fast)
git pull origin main  # fetch + merge in one
```

**Important**: pull is **merging**, not overwriting

---

### 2. Git's 3 Stages

```
Working Directory (working)
   ↓ git add
Staging Area (ready to commit)
   ↓ git commit
Repository (committed)
```

```bash
# Stage files
git add file.py

# Unstage
git reset file.py
```

---

### 3. Basic Workflow

```bash
git status
git add .
git commit -m "feat: add feature"
git push origin main
```

---

### 4. Branches

```bash
# Create and switch
git checkout -b feature/new-feature

# Work and merge
git checkout main
git pull origin main
git merge feature/new-feature
git push origin main
```

---

### 5. Conflict Resolution

```bash
git pull origin main
# CONFLICT!

# 1. Open file and fix conflict
# <<<<<<< HEAD
# My code
# =======
# Remote code
# >>>>>>> origin/main

# 2. Fix and commit
git add .
git commit -m "fix: resolve conflict"
git push origin main
```

---

### 6. git reset

```bash
# --soft: undo commit only (files stay staged)
git reset --soft HEAD~1

# --mixed (default): undo commit + unstage
git reset HEAD~1

# --hard: undo commit + delete files (⚠️ irreversible)
git reset --hard HEAD~1
```

**Use cases**:
- `--soft`: Combine multiple commits
- `--mixed`: Split commits into smaller units
- `--hard`: Completely revert work

---

### 7. git stash

```bash
# Save work temporarily
git stash

# View list
git stash list

# Retrieve
git stash pop  # Retrieve and delete
git stash apply  # Retrieve and keep
```

**When to use?**
- Need to switch branches for urgent bug fix
- Work in progress but not ready to commit

---

### 8. Atomic Commit

**Principle**: One commit = One logical change

```bash
# ❌ Bad
git add .
git commit -m "3 days work"

# ✅ Good
git add src/components/Button.tsx
git commit -m "feat: add button component"

git add src/api/auth.ts
git commit -m "feat: implement login API"
```

**Typical frequency**: 5-10 commits per day

---

### 9. File Specification and Push

```bash
# Specify files
git add src/components/Dashboard.tsx
git commit -m "feat: add dashboard"

# Or interactive
git add -p
```

**Push timing**:
- After 2-3 related commits
- At least once per day

---

### 10. git push --force

```bash
git push origin main --force
```

**⚠️ Warning**:
- Overwrites remote history with local
- Can delete others' commits

**Safer alternative**:
```bash
# --force-with-lease (safer)
git push origin main --force-with-lease
# → Rejects if others pushed
```

**Safe to use**:
- ✅ Personal branches
- ❌ Shared branches

---

### 11. Commit Message Convention

| Type | Meaning |
|------|---------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Refactoring |
| `docs` | Documentation |
| `test` | Testing |
| `chore` | Miscellaneous |

```bash
# ✅ Good
git commit -m "feat: implement Selenium crawler"
git commit -m "fix: add email validation"

# ❌ Bad
git commit -m "update"
git commit -m "3 days work"
```

---

## 📚 Core Commands Cheat Sheet

```bash
# Basics
git status
git add <file>
git commit -m "message"
git push origin main

# Sync
git fetch origin main
git pull origin main

# Branches
git checkout -b feat/xxx
git merge feat/xxx

# Undo
git reset --soft HEAD~1
git reset HEAD~1
git reset --hard HEAD~1

# Temporary save
git stash
git stash pop

# History
git log --oneline
git diff
```

---

## ❓ Interview Questions

**Q: What's the difference between fetch and pull?**
> fetch retrieves remote changes only, while pull performs fetch + merge.

**Q: What's the difference between git reset options?**
> `--soft` undoes commit only, `--mixed` undoes commit + unstage, `--hard` undoes commit + deletes files.

**Q: What is Atomic Commit?**
> Including only one logical change per commit. Makes rollback easier and contributions clearer.

**Q: When to use git stash?**
> When you need to switch branches urgently, you can temporarily save current work and retrieve it later.

**Q: When to use push --force?**
> Only on personal branches. Use `--force-with-lease` on shared branches for safety.

---

## 💡 Core Principles

1. pull is merging, not overwriting
2. Commit small and often (5-10 per day)
3. Specify files for add
4. Write clear commit messages
5. Push at least once per day

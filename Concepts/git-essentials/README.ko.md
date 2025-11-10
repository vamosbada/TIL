# Git 기초

## 🎯 한 줄 요약

실전에서 자주 쓰는 Git 명령어와 개념

---

## 📌 핵심 내용

### 1. fetch vs pull

| 명령어 | 동작 |
|--------|------|
| **fetch** | 원격 변경사항만 가져오기 (내 코드 안 건드림) |
| **pull** | 가져오기 + 내 코드와 합치기 (fetch + merge) |

```bash
# fetch (안전)
git fetch origin main
git log origin/main  # 확인
git merge origin/main  # 괜찮으면 합치기

# pull (빠름)
git pull origin main  # fetch + merge 한 번에
```

**중요**: pull은 덮어쓰기가 아니라 **합치기(merge)**

---

### 2. Git의 3단계

```
Working Directory (작업 중)
   ↓ git add
Staging Area (커밋 준비)
   ↓ git commit
Repository (커밋 완료)
```

```bash
# Staged 상태 만들기
git add file.py

# Unstage
git reset file.py
```

---

### 3. 기본 워크플로우

```bash
git status
git add .
git commit -m "feat: 기능 추가"
git push origin main
```

---

### 4. 브랜치

```bash
# 생성 및 이동
git checkout -b feature/new-feature

# 작업 후 병합
git checkout main
git pull origin main
git merge feature/new-feature
git push origin main
```

---

### 5. 충돌 해결

```bash
git pull origin main
# CONFLICT!

# 1. 파일 열어서 충돌 부분 수정
# <<<<<<< HEAD
# 내 코드
# =======
# 원격 코드
# >>>>>>> origin/main

# 2. 수정 후 커밋
git add .
git commit -m "fix: 충돌 해결"
git push origin main
```

---

### 6. git reset

```bash
# --soft: 커밋만 취소 (파일은 staged)
git reset --soft HEAD~1

# --mixed (기본): 커밋 취소 + unstaged
git reset HEAD~1

# --hard: 커밋 + 파일 모두 삭제 (⚠️ 복구 불가)
git reset --hard HEAD~1
```

**용도**:
- `--soft`: 여러 커밋을 하나로 합칠 때
- `--mixed`: 커밋을 다시 나눌 때
- `--hard`: 작업을 완전히 되돌릴 때

---

### 7. git stash

```bash
# 작업 임시 저장
git stash

# 목록 확인
git stash list

# 꺼내기
git stash pop  # 꺼내고 삭제
git stash apply  # 꺼내고 목록 유지
```

**언제 사용?**
- 급한 버그 수정으로 브랜치 전환 필요할 때
- 작업 중인 내용을 커밋하기 애매할 때

---

### 8. Atomic Commit

**원칙**: 하나의 커밋 = 하나의 논리적 변경

```bash
# ❌ 나쁜 예
git add .
git commit -m "3일치 작업"

# ✅ 좋은 예
git add src/components/Button.tsx
git commit -m "feat: 버튼 컴포넌트 추가"

git add src/api/auth.ts
git commit -m "feat: 로그인 API 구현"
```

**일반적인 커밋 빈도**: 하루 5~10개

---

### 9. 파일 지정 및 Push

```bash
# 파일 지정해서 add
git add src/components/Dashboard.tsx
git commit -m "feat: 대시보드 추가"

# 또는 대화형
git add -p
```

**Push 타이밍**:
- 기능 단위로 커밋 2-3개 쌓인 후
- 하루 1번은 무조건 push

---

### 10. git push --force

```bash
git push origin main --force
```

**⚠️ 주의**:
- 원격 저장소 히스토리를 로컬로 덮어씀
- 다른 사람의 커밋이 날아갈 수 있음

**안전한 사용**:
```bash
# --force-with-lease (더 안전)
git push origin main --force-with-lease
# → 다른 사람이 푸시했으면 거부됨
```

**사용 가능한 경우**:
- ✅ 혼자 작업하는 브랜치
- ❌ 공유 중인 브랜치

---

### 11. 커밋 메시지 컨벤션

| 타입 | 의미 |
|------|------|
| `feat` | 새 기능 |
| `fix` | 버그 수정 |
| `refactor` | 리팩토링 |
| `docs` | 문서 |
| `test` | 테스트 |
| `chore` | 기타 |

```bash
# ✅ 좋은 예
git commit -m "feat: Selenium 크롤러 구현"
git commit -m "fix: 빈 이메일 검증 추가"

# ❌ 나쁜 예
git commit -m "수정"
git commit -m "3일치 작업"
```

---

## 📚 핵심 명령어 치트시트

```bash
# 기본
git status
git add <file>
git commit -m "message"
git push origin main

# 동기화
git fetch origin main
git pull origin main

# 브랜치
git checkout -b feat/xxx
git merge feat/xxx

# 되돌리기
git reset --soft HEAD~1
git reset HEAD~1
git reset --hard HEAD~1

# 임시 저장
git stash
git stash pop

# 히스토리
git log --oneline
git diff
```

---

## ❓ 면접 질문

**Q: fetch와 pull의 차이는?**
> fetch는 원격 변경사항만 가져오고, pull은 fetch + merge를 수행합니다.

**Q: git reset 옵션의 차이는?**
> `--soft`는 커밋만 취소, `--mixed`는 커밋+unstage, `--hard`는 커밋+파일 모두 삭제합니다.

**Q: Atomic Commit이란?**
> 하나의 커밋에 하나의 논리적 변경만 포함하는 것입니다. 롤백이 쉽고 기여 내역이 명확해집니다.

**Q: git stash는 언제 사용하나요?**
> 작업 중 급하게 브랜치를 전환해야 할 때, 현재 작업을 임시 저장하고 나중에 꺼낼 수 있습니다.

**Q: push --force는 언제 사용하나요?**
> 혼자 작업하는 브랜치에서만 사용해야 합니다. 공유 브랜치에서는 `--force-with-lease`를 사용합니다.

---

## 💡 핵심 원칙

1. pull은 합치기, 덮어쓰기 아님
2. 커밋은 작고 자주 (하루 5~10개)
3. 파일 지정해서 add
4. 명확한 커밋 메시지
5. 하루 1번은 무조건 push

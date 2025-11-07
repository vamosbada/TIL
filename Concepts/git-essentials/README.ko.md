# Git 기초

## 🎯 한 줄 요약

실전에서 자주 쓰는 Git 명령어와 개념

---

## 📌 핵심 내용

### 1. fetch vs pull

**핵심 개념**: `pull`은 덮어쓰기가 아니라 **합치기(merge)**

| 명령어 | 동작 | 비유 |
|--------|------|------|
| **fetch** | 원격 변경사항만 가져오기 | 택배 받기 📦 |
| **pull** | 가져오기 + 내 코드와 합치기 | 택배 받고 정리 📦➡️📚 |

#### A. fetch (안전)
```bash
# 원격 변경사항 가져오기 (내 코드 안 건드림)
git fetch origin main

# 변경사항 확인
git log origin/main

# 괜찮으면 합치기
git merge origin/main
```

#### B. pull (빠름)
```bash
# fetch + merge를 한 번에
git pull origin main
```

**오해**: "pull하면 내 코드가 사라진다?" ❌
**진실**: pull은 원격과 내 코드를 **합치는(combine)** 작업

---

### 2. 기본 워크플로우

```bash
# 1. 변경사항 확인
git status

# 2. 파일 추가
git add .

# 3. 커밋
git commit -m "feat: 금융 캘린더 크롤러 추가"

# 4. 원격에 푸시
git push origin main
```

---

### 3. 브랜치 작업

```bash
# 브랜치 생성 및 이동
git checkout -b feature/calendar-crawler

# 작업 후 커밋
git add .
git commit -m "feat: 크롤러 구현"

# main으로 이동
git checkout main

# 변경사항 가져오기
git pull origin main

# 브랜치 병합
git merge feature/calendar-crawler

# 푸시
git push origin main
```

---

### 4. 충돌 해결

**충돌 발생 시**:
```bash
git pull origin main
# 충돌 발생!
```

**해결 방법**:
1. 충돌 파일 열기
```python
<<<<<<< HEAD
# 내 코드
def crawl():
    print("내 버전")
=======
# 원격 코드
def crawl():
    print("원격 버전")
>>>>>>> origin/main
```

2. 원하는 버전 선택 (또는 합치기)
```python
def crawl():
    print("합친 버전")
```

3. 커밋
```bash
git add .
git commit -m "fix: 충돌 해결"
git push origin main
```

---

### 5. 유용한 명령어

#### A. 변경사항 확인
```bash
# 변경된 파일 목록
git status

# 변경 내용 상세
git diff

# 커밋 히스토리
git log
git log --oneline  # 간단하게
```

#### B. 실수 복구
```bash
# 마지막 커밋 취소 (변경사항은 유지)
git reset --soft HEAD~1

# 변경사항 취소 (⚠️ 위험)
git reset --hard HEAD~1

# 특정 파일 변경사항 취소
git checkout -- file.py
```

---

### 6. 커밋 메시지 컨벤션

**형식**: `<타입>: <설명>`

| 타입 | 의미 | 예시 |
|------|------|------|
| **feat** | 새 기능 | `feat: 금융 캘린더 크롤러 추가` |
| **fix** | 버그 수정 | `fix: 팝업 제거 로직 수정` |
| **refactor** | 리팩토링 | `refactor: API 서비스 레이어 분리` |
| **docs** | 문서 | `docs: README 업데이트` |
| **test** | 테스트 | `test: 크롤러 테스트 추가` |
| **chore** | 기타 | `chore: 패키지 버전 업데이트` |

**나쁜 예시**:
```bash
git commit -m "수정"
git commit -m "ㅁㄴㅇㄹ"
git commit -m "asdf"
```

**좋은 예시**:
```bash
git commit -m "feat: Selenium 크롤러 구현"
git commit -m "fix: jQuery Datepicker 조작 로직 수정"
git commit -m "refactor: 모델 설계 개선 (DateField 분리)"
```

---

## 💡 왜 배웠나?

**Dipping 프로젝트**에서:
- 팀원들과 코드 공유 (push/pull)
- 기능별 브랜치 작업
- 충돌 해결

---

## 🔧 실제 적용

### Dipping 금융 캘린더 개발
1. `feature/calendar-crawler` 브랜치 생성
2. 크롤러 구현 후 커밋
3. main 브랜치에 병합
4. 충돌 발생 시 해결

---

## 🔗 관련 프로젝트

- [Dipping 금융 캘린더 크롤러](../../Projects/QuantrumAI/dipping-calendar-crawler/): Git 실전 사용

---

## ❓ 면접 예상 질문

**Q1: fetch와 pull의 차이는?**
> A: fetch는 원격 변경사항만 가져오고 내 코드는 건드리지 않습니다. pull은 fetch + merge를 한 번에 수행하여 원격 변경사항을 내 코드와 합칩니다. 안전하게 확인하고 싶으면 fetch, 빠르게 작업하고 싶으면 pull을 사용합니다.

**Q2: pull을 하면 내 코드가 사라지나요?**
> A: 아닙니다. pull은 원격 변경사항과 내 코드를 합치는(merge) 작업입니다. 충돌이 발생하면 Git이 알려주고, 직접 해결할 수 있습니다.

**Q3: 충돌은 언제 발생하나요?**
> A: 같은 파일의 같은 부분을 원격과 내가 다르게 수정했을 때 발생합니다. Git이 자동으로 합칠 수 없어 수동으로 해결해야 합니다.

**Q4: 좋은 커밋 메시지란?**
> A: "무엇을 왜 했는지" 명확히 알 수 있어야 합니다. `feat:`, `fix:` 같은 타입을 붙이고, 간결하게 작성합니다. "수정" 같은 모호한 메시지는 피해야 합니다.

**Q5: reset --soft와 --hard의 차이는?**
> A: `--soft`는 커밋만 취소하고 변경사항은 유지합니다. `--hard`는 커밋과 변경사항을 모두 취소하여 복구가 불가능합니다. 일반적으로 `--soft`가 안전합니다.

---

## 📚 핵심 명령어 치트시트

```bash
# 기본
git status                 # 상태 확인
git add .                  # 모든 변경사항 추가
git commit -m "메시지"      # 커밋
git push origin main       # 푸시

# 동기화
git fetch origin main      # 원격 가져오기
git pull origin main       # 가져오기 + 합치기

# 브랜치
git branch                 # 브랜치 목록
git checkout -b feat/xxx   # 브랜치 생성 및 이동
git merge feat/xxx         # 브랜치 병합

# 히스토리
git log                    # 커밋 히스토리
git log --oneline          # 간단히 보기
git diff                   # 변경사항 보기

# 복구
git reset --soft HEAD~1    # 마지막 커밋 취소 (안전)
git checkout -- file.py    # 파일 변경사항 취소
```

---

**핵심 원칙**:
1. `pull`은 합치기, 덮어쓰기 아님
2. 커밋은 작고 자주
3. 커밋 메시지는 명확하게
4. 브랜치로 기능 분리
5. push 전에 항상 pull

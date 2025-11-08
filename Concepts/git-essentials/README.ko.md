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

### 6. Atomic Commit (작은 단위 커밋)

**핵심 원칙**: 하나의 커밋 = 하나의 논리적 변경

#### 왜 작게 커밋해야 하나?

**문제 상황**:
```bash
# ❌ 3일치 작업을 한 번에
git add .
git commit -m "3일치 작업 완료"

→ 버그 발견 시 3일치 전부 롤백
→ 코드 리뷰 불가능 (+500 lines)
→ 협업 시 충돌 대참사
→ 기여 내역 불명확
```

**해결책**:
```bash
# ✅ 작은 단위로 자주 커밋
git add src/components/LoginButton.tsx
git commit -m "feat: 로그인 버튼 컴포넌트 추가"

# 30분~1시간 후
git add src/api/auth.ts
git commit -m "feat: 로그인 API 함수 구현"

# 버그 발견 → 즉시 수정
git add src/components/LoginButton.tsx
git commit -m "fix: 로그인 버튼 로딩 상태 표시 추가"
```

#### 커밋 주기

**일반적인 개발자**: 하루 5~10개 커밋
- 기능 하나 완성할 때마다
- 버그 하나 고칠 때마다
- 파일 하나 추가할 때마다

**실전 예시**:
```bash
09:00 - 로그인 폼 UI 구현
09:30 - git commit "feat: 로그인 폼 컴포넌트 구현"

10:30 - API 연결
11:00 - git commit "feat: 로그인 API 연동"

11:30 - 버그 수정
11:45 - git commit "fix: 빈 이메일 입력 시 검증 추가"

14:00 - 테스트 코드
15:00 - git commit "test: 로그인 폼 단위 테스트 추가"
```

---

### 7. 파일 지정 및 Push 전략

#### A. git add . 지양하기

**문제**:
```bash
# ❌ 모든 파일 무분별하게 추가
git add .
git commit -m "작업함"

→ 불필요한 파일 커밋 (.env, node_modules 등)
→ 관련 없는 변경사항이 한 커밋에
```

**해결**:
```bash
# ✅ 파일 지정해서 추가
git add src/components/Dashboard.tsx
git commit -m "feat: 대시보드 레이아웃 추가"

git add src/api/stock.ts
git commit -m "feat: 주식 데이터 조회 API 추가"

# 또는 대화형 선택
git add -p  # 변경사항을 하나씩 확인하며 추가
```

#### B. Push 타이밍

**Option 1: 기능 단위로 push (추천)**
```bash
# 관련된 커밋 2-3개 쌓인 후
git commit -m "feat: 로그인 폼 구현"
git commit -m "feat: 로그인 API 연동"
git commit -m "test: 로그인 테스트 추가"
git push origin main
```

**Option 2: 하루 1-2번**
```bash
# 점심 전
git push origin main

# 퇴근 전 (필수)
git push origin main
```

**Option 3: 커밋마다 (팀 협업 시)**
```bash
git commit -m "feat: 새 기능"
git push origin main  # 바로 push
```

**핵심**: 최소 하루 1번은 무조건 push

---

### 8. 커밋 메시지 컨벤션

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
git commit -m "git add ."
git commit -m "3일치 작업"
```

**좋은 예시**:
```bash
git commit -m "feat: Selenium 크롤러 구현"
git commit -m "fix: jQuery Datepicker 조작 로직 수정"
git commit -m "refactor: 모델 설계 개선 (DateField 분리)"
git commit -m "test: 크롤러 API 단위 테스트 추가"
```

---

## 💡 왜 배웠나?

**Zerothon 2025 해커톤**에서:
- Git 브랜치 전략 미숙으로 커밋 히스토리 문제 발생
- 한 팀원이 `final commit`으로 모든 코드 일괄 업로드
- 개별 기여 내역 불명확

**교훈**:
- Atomic Commit의 중요성 깨달음
- 작은 단위로 자주 커밋해야 기여 내역 명확
- 파일 지정해서 add, 의미 있는 커밋 메시지 필수

**이후 개선**:
- QuantrumAI 프로젝트부터 작은 단위 커밋 적용
- 하루 5~10개 커밋, 매일 push
- PR 기반 협업으로 전환

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

**Q2: 커밋은 얼마나 자주 해야 하나요?**
> A: 하나의 논리적 변경이 완료될 때마다 커밋해야 합니다. 일반적으로 하루 5~10개 정도가 적절하며, 3일치 작업을 한 번에 커밋하면 롤백이 어렵고 기여 내역이 불명확해집니다. Zerothon 프로젝트에서 이 문제를 겪은 후 작은 단위로 자주 커밋하는 습관을 들였습니다.

**Q3: git add . 을 사용해도 되나요?**
> A: 지양해야 합니다. 불필요한 파일(.env, node_modules 등)이나 관련 없는 변경사항이 함께 커밋될 수 있습니다. 파일을 지정해서 add하거나 git add -p로 대화형으로 선택하는 것이 안전합니다.

**Q4: 좋은 커밋 메시지란?**
> A: "무엇을 왜 했는지" 명확히 알 수 있어야 합니다. `feat:`, `fix:` 같은 타입을 붙이고, 간결하게 작성합니다. "수정" 같은 모호한 메시지는 피해야 합니다.

**Q5: push는 언제 해야 하나요?**
> A: 최소 하루 1번은 push해야 합니다. 기능 단위로 커밋 2-3개 쌓인 후 push하거나, 팀 협업 시에는 커밋마다 바로 push하기도 합니다. 퇴근 전 push는 필수입니다.

**Q6: reset --soft와 --hard의 차이는?**
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
2. **커밋은 작고 자주** (하루 5~10개)
3. **git add .** 지양, 파일 지정해서 add
4. **커밋 메시지는 명확하게** (feat, fix, docs 등)
5. **하루 1번은 무조건 push** (퇴근 전 필수)
6. 브랜치로 기능 분리
7. push 전에 항상 pull

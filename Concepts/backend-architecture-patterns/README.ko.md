# 백엔드 아키텍처 패턴

## 🎯 한 줄 요약

백엔드 코드 구조 설계 방법: 계층형, 마이크로서비스 등 주요 패턴 이해

---

## 📌 핵심 내용

### 1. 아키텍처 설계란?

**개념**: 집을 짓기 전 설계도를 그리는 것처럼, 코드를 어떻게 구성하고 조직할지 미리 계획하는 것

**왜 필요한가?**
- ✅ 유지보수 쉬워짐
- ✅ 팀 협업 편해짐
- ✅ 확장 시 구조 안정
- ✅ 버그 감소 (책임 분리로 사이드 이팩트 감소)

---

## 🏗 5가지 주요 아키텍처 패턴

### 1. 계층형 (Layered)

**비유**: 🏢 아파트 건물

**구조**: 위아래로 층층이 쌓는 방식
```
4층: UI (Controller)
3층: 비즈니스 로직 (Service)
2층: 데이터 접근 (Repository)
1층: 데이터베이스 (Database)
```

**핵심 규칙**: 위층에서 아래층으로만 접근 가능
- ✅ 4층 → 3층 → 2층 → 1층
- ❌ 4층 → 1층 (직접 접근 금지!)

**언제 쓰나**:
- 대부분의 웹 서비스 (90%+)
- 스타트업, 중소기업
- 개인 프로젝트, 포트폴리오

---

### 2. 클라이언트-서버 (Client-Server)

**비유**: 🍽 음식점

```
손님 (클라이언트)
  ↓ "불고기 주세요" (HTTP 요청)
주문 받는 곳 (API)
  ↓
주방 (서버)
  ↓ "불고기 완성!" (HTTP 응답)
손님에게 전달
```

**특징**: 손님은 주방이 어떻게 돌아가는지 몰라도 됨

**언제 쓰나**:
- 웹/모바일 앱
- REST API, GraphQL

---

### 3. 마이크로서비스 (Microservices)

**비유**: 🚚 푸드트럭 단지

```
김밥트럭 (김밥만)    타코트럭 (타코만)    커피트럭 (커피만)
   ↓                   ↓                   ↓
각각 독립적 운영    김밥트럭 고장나도    타코트럭은 멀쩡!
```

**특징**:
- 각 서비스가 독립적
- 하나 고장나도 나머지는 작동
- 서비스별 팀 분리 가능

**언제 쓰나**:
- Netflix, Amazon, Google
- 팀 100명+
- 트래픽 엄청날 때
- ⚠️ 비용 10배+

---

### 4. BFF (Backend for Frontend)

**비유**: 🏬 백화점 안내데스크

```
웹 손님      → 웹 안내데스크    → 맞춤 서비스
모바일 손님  → 모바일 안내데스크 → 맞춤 서비스
VIP 손님     → VIP 안내데스크   → 맞춤 서비스
```

**특징**: 클라이언트 타입별로 다른 API 제공

**언제 쓰나**:
- 웹/모바일 동시 서비스
- 각 플랫폼 요구사항 다를 때

---

### 5. 오케스트레이션 (Orchestration)

**비유**: 💒 웨딩플래너

```
신랑신부: "결혼식 해주세요"
         ↓
    웨딩플래너 (조율자)
    ├→ 음악팀 섭외
    ├→ 케이터링 연락
    ├→ 사진사 예약
    ├→ 꽃집 주문
    └→ 리무진 렌탈
```

**특징**: 여러 서비스를 조율하고 관리

**언제 쓰나**:
- 복잡한 비즈니스 플로우
- 여러 서비스 연동 필요

---

## 🏢 계층형 아키텍처 상세

**가장 많이 쓰이는 패턴 (90%+)**

### A. 4층 구조

```
┌─────────────────────────────────┐
│  4층: Controller (컨트롤러)      │  ← 사용자 요청 받음
│  "요청 받고 응답만"               │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│  3층: Service (서비스)           │  ← 실제 비즈니스 로직
│  "실제 일 처리"                   │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│  2층: Repository (레포지토리)    │  ← DB 접근만
│  "DB 전문가"                      │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│  1층: Database (데이터베이스)    │  ← 실제 데이터
└─────────────────────────────────┘
```

### B. 각 층의 역할

#### Controller (4층)
```python
@app.route('/users', methods=['GET'])
def get_users():
    # 요청 받기
    # Service 호출
    users = user_service.get_all_users()
    # 응답 반환
    return jsonify(users)
```

**역할**:
- HTTP 요청 받기
- Service 호출
- HTTP 응답 반환
- ❌ 비즈니스 로직 없음!

#### Service (3층)
```python
class UserService:
    def get_all_users(self):
        # 비즈니스 로직
        users = user_repository.find_all()
        # 데이터 가공
        return [self.format_user(u) for u in users]
```

**역할**:
- 실제 비즈니스 로직
- 데이터 가공/검증
- Repository 호출
- ❌ DB 직접 접근 금지!

#### Repository (2층)
```python
class UserRepository:
    def find_all(self):
        # DB 접근만
        return db.session.query(User).all()

    def find_by_id(self, user_id):
        return db.session.query(User).filter_by(id=user_id).first()
```

**역할**:
- DB 접근만 (SQL 쿼리)
- CRUD 작업
- ❌ 비즈니스 로직 없음!

#### Database (1층)
```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100))
    email = db.Column(db.String(100))
```

**역할**: 실제 데이터 저장

---

### C. 계층형의 장점

**1. 책임 명확**
```
"사용자 목록 어디 있지?"
→ Controller 보면 됨!

"비즈니스 로직 어디 있지?"
→ Service 보면 됨!

"DB 쿼리 어디 있지?"
→ Repository 보면 됨!
```

**2. 테스트 용이**
```python
# Service 테스트 (Repository 목 사용)
def test_get_users():
    mock_repo = Mock()
    service = UserService(mock_repo)
    # Service만 테스트 가능!
```

**3. 유지보수 쉬움**
```
DB 변경:
✅ Repository만 수정 (1개 파일)

vs

계층 없이:
❌ 10~15개 파일 수정
```

**4. 재사용성**
```python
# Service는 여러 Controller에서 사용 가능
user_service.get_all_users()  # REST API
user_service.get_all_users()  # GraphQL
user_service.get_all_users()  # CLI
```

---

## 🔀 아키텍처 선택 기준

| 기준 | 계층형 | 마이크로서비스 |
|------|--------|---------------|
| **팀 규모** | 1~10명 | 100명+ |
| **서비스 복잡도** | 간단~보통 | 매우 복잡 |
| **트래픽** | ~수천명 | 수만명+ |
| **예산** | 적음 | 많음 (10배+) |
| **개발 속도** | 빠름 (MVP) | 느림 (초기 설정) |
| **확장성** | 보통 | 매우 높음 |

**실제 사례**:
- **Netflix**: 초기 계층형 → 현재 마이크로서비스
- **Facebook**: 초기 계층형 → 성장기 클라이언트-서버 → 현재 마이크로서비스 + BFF
- **토스**: 결제(마이크로서비스) + 내부 어드민(계층형)

---

## 💡 왜 배웠나?

**개인 학습**으로:
- 실제 프로젝트 코드 구조 문제 발견
- 계층형 아키텍처 필요성 깨달음
- 리팩토링 방향 설정

---

## 🔧 실전 경험: 현재 코드 문제점

### ❌ 문제: 층 뛰어넘기

```python
# routes.py (Controller)
@app.route('/users')
def get_users():
    # 4층에서 바로 1층으로!
    users = db.session.query(User).all()  # ❌
    return jsonify(users)
```

**문제점**:
1. **중복 코드**: 같은 쿼리가 10~15개 파일에 복붙
2. **유지보수 지옥**: DB 스키마 변경 시 10~15개 파일 수정
3. **테스트 어려움**: Controller와 DB가 강결합
4. **책임 뒤섞임**: 하나의 파일에 모든 로직

### ✅ 해결: 계층 분리

```python
# controller.py (4층)
@app.route('/users')
def get_users():
    users = user_service.get_all_users()  # Service 호출
    return jsonify(users)

# service.py (3층)
class UserService:
    def get_all_users(self):
        return user_repository.find_all()  # Repository 호출

# repository.py (2층)
class UserRepository:
    def find_all(self):
        return db.session.query(User).all()  # DB 접근
```

**효과**:
- ✅ 중복 제거: 쿼리가 1곳에만
- ✅ 유지보수: DB 변경 시 1개 파일만 수정
- ✅ 테스트: 각 층 독립적 테스트
- ✅ 책임 명확: 각자 역할 명확

---

## 🔗 관련 개념

- [Django 백엔드 패턴](../django-backend-patterns/): Django에서의 실제 구현

---

## ❓ 면접 예상 질문

**Q1: 계층형 아키텍처가 뭔가요?**
> A: 코드를 Controller, Service, Repository, Database 4층으로 나누는 구조입니다. 위층에서 아래층으로만 접근 가능하며, 각 층의 책임이 명확합니다. Controller는 요청/응답만, Service는 비즈니스 로직, Repository는 DB 접근만 담당합니다.

**Q2: 왜 Controller에서 DB를 직접 접근하면 안 되나요?**
> A: 책임이 뒤섞이고 유지보수가 어려워집니다. DB 스키마가 변경되면 모든 Controller를 수정해야 하고, 같은 쿼리가 여러 곳에 중복됩니다. Repository로 분리하면 DB 변경 시 1곳만 수정하면 됩니다.

**Q3: 마이크로서비스와 계층형의 차이는?**
> A: 계층형은 하나의 애플리케이션 내에서 코드를 층으로 나누는 것이고, 마이크로서비스는 기능별로 완전히 독립된 애플리케이션으로 분리하는 것입니다. 마이크로서비스는 확장성은 높지만 비용과 복잡도가 10배 이상 높습니다.

**Q4: 언제 계층형을 쓰고, 언제 마이크로서비스를 쓰나요?**
> A: 팀이 10명 미만이고 트래픽이 적당하면 계층형, 팀이 100명+이고 트래픽이 엄청나면 마이크로서비스입니다. Netflix 같은 대기업도 초기엔 계층형으로 시작했습니다. 90% 이상의 서비스는 계층형으로 충분합니다.

**Q5: Service 레이어가 왜 필요한가요?**
> A: 비즈니스 로직과 DB 접근을 분리하기 위해서입니다. Controller에 비즈니스 로직을 넣으면 테스트가 어렵고, Repository에 넣으면 재사용이 어렵습니다. Service는 여러 Repository를 조합하여 복잡한 비즈니스 로직을 처리하고, 여러 Controller에서 재사용할 수 있습니다.

---

## 📚 핵심 정리

### 1. 아키텍처는 설계도
```
코드 작성 전에 구조 계획
→ 유지보수 쉬움
→ 팀 협업 편함
→ 버그 적음
```

### 2. 계층형이 대세 (90%+)
```
Controller (요청/응답)
    ↓
Service (비즈니스 로직)
    ↓
Repository (DB 접근)
    ↓
Database (데이터)
```

### 3. 핵심 규칙
```
✅ 위 → 아래로만 접근
❌ 층 뛰어넘기 금지
✅ 각 층의 책임 명확히
```

### 4. 선택 기준
```
간단한 프로젝트 → 계층형
복잡한 대기업 → 마이크로서비스
처음부터 완벽할 필요 없음 (나중에 변경 가능)
```

---

**핵심 원칙**:
1. **분리**: 관심사의 분리 (Separation of Concerns)
2. **책임**: 단일 책임 원칙 (Single Responsibility)
3. **의존성**: 위에서 아래로만 (Dependency Direction)
4. **실용성**: 상황에 맞게 선택 (Pragmatism)

---

**배운 점**:
- 아키텍처는 복잡한 게 아니라 "정리 정돈"
- 처음엔 번거로워 보여도 나중엔 훨씬 편함
- 대부분 프로젝트는 계층형으로 충분
- 실제 문제를 겪어봐야 필요성 이해됨

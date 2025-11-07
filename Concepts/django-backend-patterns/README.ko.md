# Django 백엔드 패턴

## 🎯 한 줄 요약

Django ORM 설계, API 개발, 자동 스케줄링의 실전 패턴

---

## 📌 핵심 내용

### 1. ORM (Object-Relational Mapper)

**개념**: 파이썬 코드를 SQL로 자동 변환하는 '통역사'

```python
# Python (ORM)
Event.objects.create(name="GDP 발표", date="2025-10-20")

# SQL (자동 변환)
# INSERT INTO events (name, date) VALUES ('GDP 발표', '2025-10-20');
```

---

### 2. 모델 설계 원칙

#### A. "분리할 수 있으면 분리하라"

**❌ 나쁜 설계**:
```python
class Event(models.Model):
    datetime = models.DateTimeField()  # 날짜/시간 합쳐짐
```

**문제점**:
```python
# 2025-10-20의 모든 이벤트 조회 (복잡!)
Event.objects.filter(
    datetime__gte='2025-10-20 00:00:00',
    datetime__lt='2025-10-21 00:00:00'
)
```

**✅ 좋은 설계**:
```python
class Event(models.Model):
    date = models.DateField()  # 날짜
    time = models.TimeField()  # 시간
```

**장점**:
```python
# 2025-10-20의 모든 이벤트 조회 (간단!)
Event.objects.filter(date='2025-10-20')
```

#### B. unique_together로 중복 방지

```python
class EconomicEvent(models.Model):
    date = models.DateField()
    time = models.TimeField()
    event_name = models.CharField(max_length=200)

    class Meta:
        unique_together = ['date', 'time', 'event_name']
```

**효과**:
- DB 레벨에서 중복 데이터 원천 차단
- 애플리케이션 코드에서 중복 체크 불필요

---

### 3. 효율적인 DB 저장

#### ❌ 비효율적인 방법
```python
# 데이터가 있는지 확인 후 분기
if Event.objects.filter(date=date, event_name=name).exists():
    Event.objects.filter(date=date, event_name=name).update(...)
else:
    Event.objects.create(...)
```

**문제점**:
- 2번의 DB 쿼리 (조회 + 저장/업데이트)
- 경쟁 조건 (race condition) 가능성

#### ✅ 효율적인 방법
```python
Event.objects.update_or_create(
    date=date,
    time=time,
    event_name=event_name,
    defaults={
        'country': country,
        'importance': importance,
    }
)
```

**장점**:
- 한 번의 원자적 연산
- 경쟁 조건 방지
- 코드 간결

---

### 4. Django REST Framework (DRF) API

#### A. 기본 ViewSet

```python
from rest_framework import viewsets

class EconomicEventViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = EconomicEvent.objects.all()
    serializer_class = EconomicEventSerializer
```

**생성되는 엔드포인트**:
```
GET /api/events/       # 리스트 조회
GET /api/events/{id}/  # 단일 조회
```

#### B. 동적 필터링

```python
class EconomicEventViewSet(viewsets.ReadOnlyModelViewSet):
    serializer_class = EconomicEventSerializer

    def get_queryset(self):
        queryset = EconomicEvent.objects.all()

        # URL 파라미터로 필터링
        country = self.request.query_params.get('country')
        if country:
            queryset = queryset.filter(country=country)

        date = self.request.query_params.get('date')
        if date:
            queryset = queryset.filter(date=date)

        return queryset.order_by('date', 'time')
```

**사용 예시**:
```
GET /api/events/                 # 전체
GET /api/events/?country=US      # 미국 이벤트만
GET /api/events/?date=2025-10-20 # 특정 날짜만
```

#### C. Serializer

```python
from rest_framework import serializers

class EconomicEventSerializer(serializers.ModelSerializer):
    class Meta:
        model = EconomicEvent
        fields = '__all__'  # 모든 필드
        # fields = ['date', 'event_name']  # 특정 필드만
```

**역할**:
- 모델 → JSON 변환
- 데이터 검증
- 필드 커스터마이징

---

### 5. APScheduler (자동 스케줄링)

#### A. 기본 설정

```python
from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.triggers.cron import CronTrigger

def crawl_economic_calendar():
    """크롤링 함수"""
    print("크롤링 시작!")
    # 크롤링 로직...

# 스케줄러 생성
scheduler = BackgroundScheduler()

# 작업 추가 (매달 마지막 날 23시)
scheduler.add_job(
    crawl_economic_calendar,
    trigger=CronTrigger(day='last', hour=23),
    id='monthly_crawl',
    replace_existing=True
)

scheduler.start()
```

#### B. Django 통합

```python
# apps.py
from django.apps import AppConfig

class CalendarConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'calendar_app'

    def ready(self):
        """Django 서버 시작 시 자동 실행"""
        from . import scheduler
        scheduler.start()
```

#### C. CronTrigger 패턴

```python
# 매일 23시
CronTrigger(hour=23)

# 매달 1일 00시
CronTrigger(day=1, hour=0)

# 매달 마지막 날 23시
CronTrigger(day='last', hour=23)

# 매주 월요일 09시
CronTrigger(day_of_week='mon', hour=9)

# 매시 정각
CronTrigger(minute=0)
```

**장점**:
- 윤년, 30/31일 자동 계산
- 복잡한 스케줄 간단히 표현

---

### 6. APScheduler vs Celery

| 항목 | APScheduler | Celery |
|------|-------------|--------|
| **복잡도** | 간단 | 복잡 |
| **의존성** | 없음 | Redis/RabbitMQ 필요 |
| **사용 시기** | 간단한 스케줄링 | 대규모 분산 작업 |
| **Django 통합** | `apps.py`로 간단 | 별도 워커 프로세스 |

**선택 기준**:
- 월 1회 크롤링 → APScheduler
- 초당 100개 이메일 발송 → Celery

---

## 💡 왜 배웠나?

**Dipping 프로젝트**에서:
- 금융 이벤트 데이터 저장 (ORM)
- API 엔드포인트 제공 (DRF)
- 매달 자동 크롤링 (APScheduler)

---

## 🔧 실제 적용

### Dipping 금융 캘린더 백엔드
1. **모델 설계**: DateField + TimeField 분리
2. **중복 방지**: `unique_together` 사용
3. **효율적 저장**: `update_or_create` 사용
4. **동적 API**: 국가/날짜별 필터링
5. **자동화**: 매달 마지막 날 23시 크롤링

---

## 🔗 관련 프로젝트

- [Dipping 금융 캘린더 크롤러](../../Projects/QuantrumAI/dipping-calendar-crawler/): Django 백엔드 실전 적용

---

## ❓ 면접 예상 질문

**Q1: DateTimeField 대신 DateField + TimeField를 선택한 이유는?**
> A: 날짜별 필터링이 매우 빈번한 요구사항이었습니다. DateTimeField는 날짜만 필터링하려면 시간 범위(00:00~23:59)를 지정해야 하지만, 분리하면 `filter(date='2025-10-20')`처럼 간단합니다. "분리할 수 있으면 분리하라"는 원칙을 따랐습니다.

**Q2: update_or_create를 사용하는 이유는?**
> A: if exists() + update/create 방식은 2번의 DB 쿼리가 필요하고 경쟁 조건(race condition)이 발생할 수 있습니다. update_or_create는 한 번의 원자적 연산으로 데이터가 있으면 업데이트, 없으면 생성하여 안전하고 효율적입니다.

**Q3: APScheduler를 선택한 이유는?**
> A: Celery는 Redis/RabbitMQ 같은 메시지 브로커가 필요해 인프라가 복잡합니다. 우리 프로젝트는 월 1회 크롤링만 필요했기 때문에, Django 서버와 함께 백그라운드 스레드로 동작하는 APScheduler가 적합했습니다.

**Q4: unique_together를 사용한 이유는?**
> A: 애플리케이션 코드에서 중복 체크를 하면 경쟁 조건이 발생할 수 있습니다. unique_together를 사용하면 DB 레벨에서 중복을 원천 차단하여 더 안전합니다.

**Q5: get_queryset을 오버라이딩한 이유는?**
> A: 고정된 queryset 대신 URL 파라미터에 따라 동적으로 필터링하기 위해서입니다. 같은 엔드포인트로 전체 데이터, 국가별 데이터, 날짜별 데이터를 제공할 수 있어 유연합니다.

---

## 📚 핵심 코드 패턴

### 완전한 Django 백엔드 예시
```python
# models.py
from django.db import models

class EconomicEvent(models.Model):
    date = models.DateField()
    time = models.TimeField()
    event_name = models.CharField(max_length=200)
    country = models.CharField(max_length=100)
    importance = models.CharField(max_length=20)

    class Meta:
        unique_together = ['date', 'time', 'event_name']

# serializers.py
from rest_framework import serializers

class EconomicEventSerializer(serializers.ModelSerializer):
    class Meta:
        model = EconomicEvent
        fields = '__all__'

# views.py
from rest_framework import viewsets

class EconomicEventViewSet(viewsets.ReadOnlyModelViewSet):
    serializer_class = EconomicEventSerializer

    def get_queryset(self):
        queryset = EconomicEvent.objects.all()
        country = self.request.query_params.get('country')
        if country:
            queryset = queryset.filter(country=country)
        return queryset.order_by('date', 'time')

# scheduler.py
from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.triggers.cron import CronTrigger

def crawl_data():
    # 크롤링 로직
    EconomicEvent.objects.update_or_create(...)

scheduler = BackgroundScheduler()
scheduler.add_job(crawl_data, CronTrigger(day='last', hour=23))
scheduler.start()

# apps.py
class CalendarConfig(AppConfig):
    def ready(self):
        from . import scheduler
        scheduler.start()
```

---

**핵심 원칙**:
1. 모델: 분리 가능한 필드는 분리
2. 중복: `unique_together`로 DB 레벨 방지
3. 저장: `update_or_create` 사용
4. API: `get_queryset` 오버라이딩으로 동적 필터링
5. 스케줄링: APScheduler로 간단하게

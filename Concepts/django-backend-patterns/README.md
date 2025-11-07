# Django Backend Patterns

## 🎯 Summary

Practical patterns for Django ORM design, API development, and automated scheduling

---

## 📌 Key Points

### 1. ORM (Object-Relational Mapper)

**Concept**: 'Translator' that automatically converts Python code to SQL

```python
# Python (ORM)
Event.objects.create(name="GDP Report", date="2025-10-20")

# SQL (Auto-generated)
# INSERT INTO events (name, date) VALUES ('GDP Report', '2025-10-20');
```

---

### 2. Model Design Principles

#### A. "Separate what can be separated"

**❌ Poor design**:
```python
class Event(models.Model):
    datetime = models.DateTimeField()  # Date/time combined
```

**Problem**:
```python
# Query all events on 2025-10-20 (complex!)
Event.objects.filter(
    datetime__gte='2025-10-20 00:00:00',
    datetime__lt='2025-10-21 00:00:00'
)
```

**✅ Good design**:
```python
class Event(models.Model):
    date = models.DateField()  # Date
    time = models.TimeField()  # Time
```

**Advantage**:
```python
# Query all events on 2025-10-20 (simple!)
Event.objects.filter(date='2025-10-20')
```

#### B. Prevent duplicates with unique_together

```python
class EconomicEvent(models.Model):
    date = models.DateField()
    time = models.TimeField()
    event_name = models.CharField(max_length=200)

    class Meta:
        unique_together = ['date', 'time', 'event_name']
```

**Effect**:
- DB-level duplicate prevention
- No need for duplicate checking in application code

---

### 3. Efficient DB Operations

#### ❌ Inefficient approach
```python
# Check if exists then branch
if Event.objects.filter(date=date, event_name=name).exists():
    Event.objects.filter(date=date, event_name=name).update(...)
else:
    Event.objects.create(...)
```

**Problems**:
- Two DB queries (check + save/update)
- Possible race condition

#### ✅ Efficient approach
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

**Advantages**:
- Single atomic operation
- Prevents race conditions
- Concise code

---

### 4. Django REST Framework (DRF) API

#### A. Basic ViewSet

```python
from rest_framework import viewsets

class EconomicEventViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = EconomicEvent.objects.all()
    serializer_class = EconomicEventSerializer
```

**Generated endpoints**:
```
GET /api/events/       # List view
GET /api/events/{id}/  # Detail view
```

#### B. Dynamic Filtering

```python
class EconomicEventViewSet(viewsets.ReadOnlyModelViewSet):
    serializer_class = EconomicEventSerializer

    def get_queryset(self):
        queryset = EconomicEvent.objects.all()

        # Filter by URL parameters
        country = self.request.query_params.get('country')
        if country:
            queryset = queryset.filter(country=country)

        date = self.request.query_params.get('date')
        if date:
            queryset = queryset.filter(date=date)

        return queryset.order_by('date', 'time')
```

**Usage examples**:
```
GET /api/events/                 # All events
GET /api/events/?country=US      # US events only
GET /api/events/?date=2025-10-20 # Specific date only
```

#### C. Serializer

```python
from rest_framework import serializers

class EconomicEventSerializer(serializers.ModelSerializer):
    class Meta:
        model = EconomicEvent
        fields = '__all__'  # All fields
        # fields = ['date', 'event_name']  # Specific fields only
```

**Roles**:
- Model → JSON conversion
- Data validation
- Field customization

---

### 5. APScheduler (Automated Scheduling)

#### A. Basic Setup

```python
from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.triggers.cron import CronTrigger

def crawl_economic_calendar():
    """Crawling function"""
    print("Starting crawl!")
    # Crawling logic...

# Create scheduler
scheduler = BackgroundScheduler()

# Add job (last day of month at 23:00)
scheduler.add_job(
    crawl_economic_calendar,
    trigger=CronTrigger(day='last', hour=23),
    id='monthly_crawl',
    replace_existing=True
)

scheduler.start()
```

#### B. Django Integration

```python
# apps.py
from django.apps import AppConfig

class CalendarConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'calendar_app'

    def ready(self):
        """Auto-execute when Django server starts"""
        from . import scheduler
        scheduler.start()
```

#### C. CronTrigger Patterns

```python
# Daily at 23:00
CronTrigger(hour=23)

# 1st of month at 00:00
CronTrigger(day=1, hour=0)

# Last day of month at 23:00
CronTrigger(day='last', hour=23)

# Every Monday at 09:00
CronTrigger(day_of_week='mon', hour=9)

# Every hour on the hour
CronTrigger(minute=0)
```

**Advantages**:
- Auto-calculates leap years, 30/31 days
- Express complex schedules simply

---

### 6. APScheduler vs Celery

| Aspect | APScheduler | Celery |
|--------|-------------|--------|
| **Complexity** | Simple | Complex |
| **Dependencies** | None | Requires Redis/RabbitMQ |
| **Use case** | Simple scheduling | Large-scale distributed tasks |
| **Django integration** | Simple with `apps.py` | Separate worker process |

**Selection criteria**:
- Monthly crawl → APScheduler
- 100 emails/second → Celery

---

## 💡 Why I Learned This

**Dipping project** needed:
- Store financial event data (ORM)
- Provide API endpoints (DRF)
- Automated monthly crawling (APScheduler)

---

## 🔧 Practical Application

### Dipping Financial Calendar Backend
1. **Model design**: Separated DateField + TimeField
2. **Duplicate prevention**: Used `unique_together`
3. **Efficient storage**: Used `update_or_create`
4. **Dynamic API**: Filtering by country/date
5. **Automation**: Crawl on last day of month at 23:00

---

## 🔗 Related Projects

- [Dipping Financial Calendar Crawler](../../Projects/QuantrumAI/dipping-calendar-crawler/): Real-world Django backend application

---

## ❓ Interview Prep Questions

**Q1: Why DateField + TimeField instead of DateTimeField?**
> A: Date-based filtering was a very frequent requirement. DateTimeField requires specifying time range (00:00~23:59) to filter by date only, but with separation, it's as simple as `filter(date='2025-10-20')`. I followed the principle of "separate what can be separated."

**Q2: Why use update_or_create?**
> A: The if exists() + update/create approach requires two DB queries and can have race conditions. update_or_create is a single atomic operation that updates if exists or creates if not, making it safe and efficient.

**Q3: Why choose APScheduler?**
> A: Celery requires message brokers like Redis/RabbitMQ, adding infrastructure complexity. Our project only needed monthly crawling, so APScheduler, which operates as a background thread with the Django server, was more suitable.

**Q4: Why use unique_together?**
> A: Checking for duplicates in application code can have race conditions. Using unique_together enforces duplicate prevention at the DB level, making it safer.

**Q5: Why override get_queryset?**
> A: To dynamically filter based on URL parameters instead of a fixed queryset. This allows serving all data, country-specific data, or date-specific data through the same endpoint, providing flexibility.

---

## 📚 Core Code Patterns

### Complete Django Backend Example
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
    # Crawling logic
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

**Core Principles**:
1. Models: Separate separable fields
2. Duplicates: DB-level prevention with `unique_together`
3. Storage: Use `update_or_create`
4. API: Dynamic filtering via `get_queryset` override
5. Scheduling: Keep it simple with APScheduler

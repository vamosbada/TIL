# Dipping - Financial Calendar Crawler Full-Stack Implementation

## 🎯 Project Background

**Problem**: Stock investors miss important economic indicator announcements, failing to respond to sudden price movements

**Solution**: Crawl financial event data from Investing.com and present it in a user-friendly calendar UI

**Core Value**: "Never miss critical economic events"

---

## 🛠 Tech Stack

### Backend
- **Python 3.x**: Crawling and backend logic
- **Selenium + undetected-chromedriver**: Dynamic web crawling
- **BeautifulSoup4**: HTML parsing
- **Django 5.1**: Backend framework
- **Django REST Framework**: API server
- **PostgreSQL**: Database
- **APScheduler**: Automated scheduling

### Frontend
- **React 19**: UI framework
- **TypeScript**: Type safety
- **AJAX (fetch API)**: Asynchronous data communication

---

## 💻 My Role

**Full ownership of financial calendar crawler within the team**
- I was solely responsible for this feature from start to finish within the QuantrumAI team
- Web crawler implementation (Selenium + Anti-Bot bypass)
- Django backend development (ORM, API, scheduling)
- React + TypeScript frontend development
- End-to-end data flow design and integration

---

## 🔧 Key Implementation

### 1. Web Crawling (Selenium)

#### A. Anti-Bot Bypass Strategy
**Problem**: Investing.com's bot detection system

**Solution**:
```python
import undetected_chromedriver as uc

options = uc.ChromeOptions()
options.add_argument('--user-agent=Mozilla/5.0...')
driver = uc.Chrome(options=options)
```

- Used `undetected-chromedriver`
- Hid `navigator.webdriver = true` attribute
- Changed User-Agent

#### B. Popup Handling (Saved 200 seconds)
**❌ Failed approach**:
```python
# Wait for popup to appear and click
wait.until(EC.element_to_be_clickable((By.ID, "popup")))
popup.click()  # Slow!
```

**✅ Successful approach**:
```python
# Force remove popup elements from DOM using JavaScript
driver.execute_script("""
    document.querySelector('.popup').remove();
    document.querySelector('.general-overlay').remove();
""")
```

→ **Key learning**: DOM removal is much faster than waiting and clicking

#### C. jQuery UI Datepicker Manipulation (Project's Biggest Challenge)

**Problem**: Investing.com's date filter implemented with jQuery UI Datepicker

**❌ Failed attempts**:
```python
# 1. Direct value setting (displays but doesn't update internal state)
driver.execute_script("$('#dateInput').val('2025-10-01')")

# 2. Simple click (doesn't select range)
date_element.click()
```

**✅ Successful approach - Mimicking user sequence**:
```python
# 1. Click start date (1st)
start_date = driver.find_element(By.XPATH, "//a[text()='1']")
start_date.click()
time.sleep(1)

# 2. Click end date (31st)
end_date = driver.find_element(By.XPATH, "//a[text()='31']")
end_date.click()
time.sleep(1)

# 3. Click start date again (confirm range)
start_date.click()
time.sleep(1)

# 4. Click Apply button (trigger onClose event)
apply_button = driver.find_element(By.ID, "applyBtn")
apply_button.click()
```

**Key insight**:
> "Setting value ≠ User action"
> jQuery UI's internal state only updates through actual click events

**Why this sequence is necessary?**
- jQuery UI Datepicker confirms range via "start → end → start" clicks
- `applyBtn` click triggers `onClose` event to apply the date

#### D. Ensuring Stability

- Explicit Wait for element loading
- Retry logic (up to 3 attempts)
- Safe parsing (try/except)

---

### 2. Django Backend

#### Key Implementation

**Model Design**: Separate DateField + TimeField (easier date filtering)
- `unique_together` prevents duplicates

**DB Operations**: Efficient storage with `update_or_create`
- Updates if exists, creates if not

**API**: Dynamic filtering with DRF ViewSet
- `/api/events/?country=US` (country-specific queries)

**Automated Scheduling**: APScheduler
- Monthly crawling on last day at 23:00

---

### 3. React + TypeScript Frontend

**Type Definitions**: EconomicEvent interface ensures type safety

**API Service**: Service layer separation for reusability

**Data Processing**: Date-based grouping + D-Day calculation

---

## 📊 Complete Data Flow

```
[Investing.com]
       ↓ (Selenium crawling)
   [Crawler]
       ↓ (ORM)
 [PostgreSQL]
       ↓ (Django API)
  [JSON response]
       ↓ (fetch)
[React Frontend]
       ↓ (Rendering)
    [User]
```

---

## 💡 Key Takeaways

### Technical Learnings

**1. Complexity of Web Crawling**
- Anti-Bot bypass is an ongoing battle
- Importance of understanding jQuery UI internals
- DOM manipulation vs user sequence mimicking

**2. Django Design Principles**
- Separate separable fields (Date + Time)
- DB-level duplicate prevention with `unique_together`
- Atomic operations with `update_or_create`

**3. Value of TypeScript**
- Types as living documentation
- Runtime stability through compile-time errors

### Problem-Solving Methodology

**Divide and Conquer**:
1. Crawling only (save to TXT)
2. Test DB storage
3. Build API endpoints
4. Frontend integration

---

## 🔗 Related Concepts

- [Selenium Crawling Techniques](../../Concepts/selenium-crawling-techniques/)
- [Django Backend Patterns](../../Concepts/django-backend-patterns/)
- [React TypeScript Basics](../../Concepts/react-typescript-basics/)
- [Git Essentials](../../Concepts/git-essentials/)
- [Debugging Strategies](../../Concepts/debugging-strategies/)
- [Developer Mindset](../../Concepts/developer-mindset/)

---

## ❓ Interview Prep Questions

**Q1: How did you manipulate jQuery UI Datepicker?**
> A: Setting the value directly with JavaScript displays it visually but doesn't update the internal state. I recreated the user sequence with Selenium: "click start date → click end date → click start date again → click Apply button". The onClose event must be triggered for the date to actually apply.

**Q2: How did you bypass Anti-Bot detection?**
> A: I used undetected-chromedriver to hide the navigator.webdriver attribute and changed the User-Agent. Regular Selenium is detected with navigator.webdriver = true, but undetected-chromedriver hides this to mimic human behavior.

**Q3: Why DateField + TimeField instead of DateTimeField?**
> A: Date-based filtering was a very frequent requirement. With DateTimeField, filtering by date requires specifying a time range (00:00~23:59), but with separation, it's as simple as `filter(date='2025-10-20')`.

**Q4: Why separate the API service layer?**
> A: If components directly call fetch, every component needs modification when API URLs or logic change. With a service layer, only one place needs updating, and reusability increases.

**Q5: Why choose APScheduler?**
> A: Celery requires message brokers like Redis/RabbitMQ, complicating infrastructure. APScheduler operates as a background thread with the Django server, making it simple. Complex schedules like "last day of month at 23:00" are easily configured with CronTrigger.

---

## 📂 Project Structure

```
dipping-calendar/
├── backend/
│   ├── crawler/
│   │   ├── investing_crawler.py
│   │   └── scheduler.py
│   ├── calendar_app/
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── apps.py
│   └── manage.py
└── frontend/
    ├── src/
    │   ├── components/
    │   │   └── CalendarTab.tsx
    │   └── api/
    │       └── services/
    │           └── calendarService.ts
    └── package.json
```

---

**Current Status**: ✅ Complete (Deployment ready)
**Ownership**: Bada (Full ownership of financial calendar crawler within QuantrumAI team)

# Selenium Crawling Techniques

## 🎯 Summary

Core Selenium techniques for dynamic website crawling (Anti-Bot bypass, jQuery UI manipulation, stability)

---

## 📌 Key Points

### 1. Selenium vs BeautifulSoup

| Tool | Role | Analogy |
|------|------|---------|
| **Selenium** | Browser automation (click, wait, input) | Acting robot arm 🤖 |
| **BeautifulSoup** | HTML parsing (data extraction) | Data tweezers 🔍 |

**Why use together**:
```python
# Selenium: Load dynamic page
driver.get("https://example.com")
driver.find_element(By.ID, "loadMore").click()

# BeautifulSoup: Parse
soup = BeautifulSoup(driver.page_source, 'html.parser')
data = soup.find_all('div', class_='item')
```

---

### 2. Anti-Bot Bypass Strategies

#### A. undetected-chromedriver (Primary)

**Problem**: Regular Selenium detected via `navigator.webdriver = true`

**Solution**:
```python
import undetected_chromedriver as uc

options = uc.ChromeOptions()
driver = uc.Chrome(options=options)
```

**Effect**: Hides `navigator.webdriver` attribute → Mimics human behavior

#### B. User-Agent Modification

```python
options = webdriver.ChromeOptions()
options.add_argument('--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64)...')
```

---

### 3. Wait Strategies

| Method | Description | Recommendation |
|--------|-------------|----------------|
| **Explicit Wait** | Wait up to N seconds for condition | ✅ Recommended |
| **Implicit Wait** | Global wait N seconds if not found | 🔵 Not recommended |
| **time.sleep()** | Always wait N seconds | ❌ Worst |

#### Explicit Wait (Recommended)

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)  # Max 10 seconds

# Wait until clickable
element = wait.until(
    EC.element_to_be_clickable((By.ID, "submitBtn"))
)
element.click()

# Wait until visible
element = wait.until(
    EC.visibility_of_element_located((By.CLASS_NAME, "result"))
)
```

**Advantages**:
- Proceeds immediately when condition met (fast)
- Waits based on specific state (stable)

---

### 4. Popup Handling

#### ❌ Slow approach: Close by clicking
```python
wait.until(EC.element_to_be_clickable((By.ID, "popup")))
popup.click()  # Slow + unstable
```

#### ✅ Fast approach: Remove with JavaScript
```python
driver.execute_script("""
    document.querySelector('.popup').remove();
    document.querySelector('.general-overlay').remove();
""")
```

**Performance difference**: 200 seconds → Instant

---

### 5. jQuery UI Datepicker Manipulation (Core)

#### Problem
- jQuery UI Datepicker maintains internal state
- Setting value with JavaScript displays it but doesn't update internal state

#### ❌ Failed: Direct value setting
```python
driver.execute_script("$('#dateInput').val('2025-10-01')")
# Displays but doesn't actually apply!
```

#### ✅ Success: Mimic user sequence
```python
# 1. Open Datepicker
date_input = driver.find_element(By.ID, "dateInput")
date_input.click()
time.sleep(1)

# 2. Click start date
start_date = driver.find_element(By.XPATH, "//a[text()='1']")
start_date.click()
time.sleep(1)

# 3. Click end date
end_date = driver.find_element(By.XPATH, "//a[text()='31']")
end_date.click()
time.sleep(1)

# 4. Click start date again (confirm range)
start_date.click()
time.sleep(1)

# 5. Apply button (trigger onClose event)
apply_button = driver.find_element(By.ID, "applyBtn")
apply_button.click()
```

**Key concept**:
> "Setting value ≠ User action"
> jQuery UI only updates internal state through actual click events

---

### 6. Retry Logic & Exception Handling

#### A. Retry Logic
```python
for attempt in range(3):
    try:
        element = driver.find_element(By.ID, "button")
        element.click()
        break  # Exit on success
    except Exception as e:
        if attempt == 2:  # Last attempt
            raise
        time.sleep(2)  # Retry after 2 seconds
```

#### B. Safe Parsing
```python
try:
    event_name = element.find_element(By.CLASS_NAME, "event").text
except NoSuchElementException:
    event_name = "N/A"  # Default value
```

---

### 7. Headless Mode

```python
options = webdriver.ChromeOptions()
options.add_argument('--headless')  # Run without GUI
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')

driver = webdriver.Chrome(options=options)
```

**When to use**:
- Local development: Headless off (easier debugging)
- Server deployment: Headless on (background execution)

---

## 💡 Why I Learned This

**Dipping project** needed to crawl Investing.com:
- Bypass Anti-Bot detection
- Manipulate jQuery UI Datepicker for date filtering
- Ensure stable data collection

---

## 🔧 Practical Application

### Dipping Financial Calendar Crawler
1. Used `undetected-chromedriver` to bypass bot detection
2. Removed popups with JavaScript (saved 200 seconds)
3. Recreated jQuery UI Datepicker user sequence
4. Ensured stability with Explicit Wait + retry logic

---

## 🔗 Related Projects

- [Dipping Financial Calendar Crawler](../../Projects/QuantrumAI/dipping-calendar-crawler/): Real-world Selenium application

---

## ❓ Interview Prep Questions

**Q1: What's the difference between Selenium and BeautifulSoup?**
> A: Selenium automates the browser to perform dynamic operations like clicking and inputting, while BeautifulSoup parses HTML to extract data. For dynamic pages, we load with Selenium then parse with BeautifulSoup.

**Q2: Why use Explicit Wait?**
> A: time.sleep() always waits the full duration (slow), and Implicit Wait is a global setting with limited control. Explicit Wait proceeds immediately when specific conditions (clickable, visible) are met, making it fast and stable.

**Q3: Why was jQuery UI manipulation challenging?**
> A: Setting the input's value directly with JavaScript displays it visually but doesn't update jQuery UI's internal state. We must trigger actual click events like a real user to fire events like onClose and apply the date.

---

## 📚 Core Code Patterns

### Basic Template
```python
import undetected_chromedriver as uc
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# Driver setup
options = uc.ChromeOptions()
driver = uc.Chrome(options=options)
wait = WebDriverWait(driver, 10)

try:
    # Navigate
    driver.get("https://example.com")

    # Explicit Wait + click
    element = wait.until(EC.element_to_be_clickable((By.ID, "button")))
    element.click()

    # Extract data
    data = driver.find_element(By.CLASS_NAME, "result").text

finally:
    driver.quit()
```

---

**Core Principles**:
1. Anti-Bot: Use `undetected-chromedriver`
2. Waiting: Use Explicit Wait
3. Popups: Remove with JavaScript
4. jQuery UI: Mimic user sequence
5. Stability: Retry + exception handling

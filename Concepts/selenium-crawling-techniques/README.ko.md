# Selenium 크롤링 기술

## 🎯 한 줄 요약

동적 웹사이트 크롤링을 위한 Selenium 핵심 기술 (Anti-Bot 우회, jQuery UI 조작, 안정성 확보)

---

## 📌 핵심 내용

### 1. Selenium vs BeautifulSoup

| 도구 | 역할 | 비유 |
|------|------|------|
| **Selenium** | 브라우저 자동화 (클릭, 대기, 입력) | 행동하는 로봇 팔 🤖 |
| **BeautifulSoup** | HTML 파싱 (데이터 추출) | 데이터 족집게 🔍 |
                
**함께 사용하는 이유**:
```python
# Selenium: 동적 페이지 로드
driver.get("https://example.com")
driver.find_element(By.ID, "loadMore").click()

# BeautifulSoup: 파싱
soup = BeautifulSoup(driver.page_source, 'html.parser')
data = soup.find_all('div', class_='item')
```

---

### 2. Anti-Bot 우회 전략

#### A. undetected-chromedriver (1순위)

**문제**: 일반 Selenium은 `navigator.webdriver = true`로 탐지됨

**해결**:
```python
import undetected_chromedriver as uc

options = uc.ChromeOptions()
driver = uc.Chrome(options=options)
```

**효과**: `navigator.webdriver` 속성 숨김 → 사람처럼 위장

#### B. User-Agent 변경

```python
options = webdriver.ChromeOptions()
options.add_argument('--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64)...')
```

---

### 3. 대기 전략 (Waits)

| 방법 | 설명 | 추천도 |
|------|------|--------|
| **Explicit Wait** | 특정 조건까지 최대 N초 대기 | ✅ 권장 |
| **Implicit Wait** | 요소 없으면 N초 대기 (전역) | 🔵 비추천 |
| **time.sleep()** | 무조건 N초 대기 | ❌ 최악 |

#### Explicit Wait (권장)

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)  # 최대 10초

# 클릭 가능할 때까지 대기
element = wait.until(
    EC.element_to_be_clickable((By.ID, "submitBtn"))
)
element.click()

# 요소가 보일 때까지 대기
element = wait.until(
    EC.visibility_of_element_located((By.CLASS_NAME, "result"))
)
```

**장점**:
- 조건 만족 시 즉시 진행 (빠름)
- 특정 상태 기준 대기 (안정적)

---

### 4. 팝업 처리

#### ❌ 느린 방법: 클릭으로 닫기
```python
wait.until(EC.element_to_be_clickable((By.ID, "popup")))
popup.click()  # 느림 + 불안정
```

#### ✅ 빠른 방법: JavaScript로 제거
```python
driver.execute_script("""
    document.querySelector('.popup').remove();
    document.querySelector('.general-overlay').remove();
""")
```

**성능 차이**: 200초 → 즉시

---

### 5. jQuery UI Datepicker 조작 (핵심)

#### 문제 상황
- jQuery UI Datepicker는 내부 상태 관리
- JavaScript로 value 설정 시 화면엔 보이지만 내부 상태 업데이트 안 됨

#### ❌ 실패: 직접 값 설정
```python
driver.execute_script("$('#dateInput').val('2025-10-01')")
# 화면엔 보이지만 실제 적용 안 됨!
```

#### ✅ 성공: 사용자 시퀀스 모방
```python
# 1. Datepicker 열기
date_input = driver.find_element(By.ID, "dateInput")
date_input.click()
time.sleep(1)

# 2. 시작 날짜 클릭
start_date = driver.find_element(By.XPATH, "//a[text()='1']")
start_date.click()
time.sleep(1)

# 3. 종료 날짜 클릭
end_date = driver.find_element(By.XPATH, "//a[text()='31']")
end_date.click()
time.sleep(1)

# 4. 다시 시작 날짜 클릭 (범위 확정)
start_date.click()
time.sleep(1)

# 5. Apply 버튼 (onClose 이벤트 트리거)
apply_button = driver.find_element(By.ID, "applyBtn")
apply_button.click()
```

**핵심 개념**:
> "값 설정 ≠ 사용자 액션"
> jQuery UI는 실제 클릭 이벤트로만 내부 상태 업데이트

---

### 6. 재시도 로직 & 예외 처리

#### A. 재시도 로직
```python
for attempt in range(3):
    try:
        element = driver.find_element(By.ID, "button")
        element.click()
        break  # 성공하면 탈출
    except Exception as e:
        if attempt == 2:  # 마지막 시도
            raise
        time.sleep(2)  # 2초 후 재시도
```

#### B. 안전한 파싱
```python
try:
    event_name = element.find_element(By.CLASS_NAME, "event").text
except NoSuchElementException:
    event_name = "N/A"  # 기본값
```

---

### 7. Headless 모드

```python
options = webdriver.ChromeOptions()
options.add_argument('--headless')  # GUI 없이 실행
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')

driver = webdriver.Chrome(options=options)
```

**사용 시기**:
- 로컬 개발: Headless 끄기 (디버깅 편함)
- 서버 배포: Headless 켜기 (백그라운드 실행)

---

## 💡 왜 배웠나?

**Dipping 프로젝트**에서 Investing.com 크롤링 시:
- Anti-Bot 차단 우회 필요
- jQuery UI Datepicker 날짜 필터 조작 필요
- 안정적인 데이터 수집 필요

---

## 🔧 실제 적용

### Dipping 금융 캘린더 크롤러
1. `undetected-chromedriver`로 봇 차단 우회
2. JavaScript로 팝업 제거 (200초 절약)
3. jQuery UI Datepicker 사용자 시퀀스 재현
4. Explicit Wait + 재시도 로직으로 안정성 확보

---

## 🔗 관련 프로젝트

- [Dipping 금융 캘린더 크롤러](../../Projects/QuantrumAI/dipping-calendar-crawler/): Selenium 실전 적용

---

## ❓ 면접 예상 질문

**Q1: Selenium과 BeautifulSoup의 차이는?**
> A: Selenium은 브라우저를 자동화하여 클릭, 입력 등 동적 작업을 수행하고, BeautifulSoup은 HTML을 파싱하여 데이터를 추출합니다. 동적 페이지는 Selenium으로 로드한 후 BeautifulSoup으로 파싱합니다.

**Q2: Explicit Wait를 사용하는 이유는?**
> A: time.sleep()은 무조건 대기하여 느리고, Implicit Wait는 전역 설정이라 세밀한 제어가 어렵습니다. Explicit Wait는 특정 조건(클릭 가능, 요소 보임)을 만족하면 즉시 진행하여 빠르고 안정적입니다.

**Q3: jQuery UI 조작이 어려웠던 이유는?**
> A: JavaScript로 input의 value를 직접 설정하면 화면엔 보이지만 jQuery UI의 내부 상태가 업데이트되지 않습니다. 실제 사용자처럼 클릭 이벤트를 발생시켜야 onClose 같은 이벤트가 트리거되고 날짜가 적용됩니다.

---

## 📚 핵심 코드 패턴

### 기본 템플릿
```python
import undetected_chromedriver as uc
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# 드라이버 설정
options = uc.ChromeOptions()
driver = uc.Chrome(options=options)
wait = WebDriverWait(driver, 10)

try:
    # 페이지 이동
    driver.get("https://example.com")

    # Explicit Wait + 클릭
    element = wait.until(EC.element_to_be_clickable((By.ID, "button")))
    element.click()

    # 데이터 추출
    data = driver.find_element(By.CLASS_NAME, "result").text

finally:
    driver.quit()
```

---

**핵심 원칙**:
1. Anti-Bot: `undetected-chromedriver` 사용
2. 대기: Explicit Wait 사용
3. 팝업: JavaScript로 제거
4. jQuery UI: 사용자 시퀀스 모방
5. 안정성: 재시도 + 예외 처리

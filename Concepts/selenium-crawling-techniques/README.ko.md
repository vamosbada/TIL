# Selenium 크롤링 기술

## 🎯 한 줄 요약

동적 웹사이트 크롤링을 위한 Selenium 핵심 기술 (Anti-Bot 우회, jQuery UI 조작, 안정성 확보)

---

## 📌 핵심 내용

### 1. Selenium vs BeautifulSoup

| 도구 | 역할 |
|------|------|
| **Selenium** | 브라우저 자동화 (클릭, 대기, 입력) |
| **BeautifulSoup** | HTML 파싱 (데이터 추출) |

```python
# Selenium: 동적 페이지 로드
driver.get("https://example.com")
driver.find_element(By.ID, "loadMore").click()

# BeautifulSoup: 파싱
soup = BeautifulSoup(driver.page_source, 'html.parser')
data = soup.find_all('div', class_='item')
```

---

### 2. Anti-Bot 우회

#### undetected-chromedriver

```python
import undetected_chromedriver as uc

options = uc.ChromeOptions()
driver = uc.Chrome(options=options)
```

**효과**: `navigator.webdriver` 속성 숨김

#### User-Agent 변경

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

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)

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

```python
# ❌ 느린 방법: 클릭으로 닫기
wait.until(EC.element_to_be_clickable((By.ID, "popup")))
popup.click()

# ✅ 빠른 방법: JavaScript로 제거
driver.execute_script("""
    document.querySelector('.popup').remove();
    document.querySelector('.general-overlay').remove();
""")
```

---

### 5. jQuery UI Datepicker 조작

#### 문제
- JavaScript로 value 설정 시 화면엔 보이지만 내부 상태 업데이트 안 됨

#### ❌ 실패
```python
driver.execute_script("$('#dateInput').val('2025-10-01')")
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

# 5. Apply 버튼
apply_button = driver.find_element(By.ID, "applyBtn")
apply_button.click()
```

**핵심**: 값 설정 ≠ 사용자 액션. jQuery UI는 실제 클릭 이벤트로만 내부 상태 업데이트

---

### 6. 재시도 로직 & 예외 처리

```python
# 재시도 로직
for attempt in range(3):
    try:
        element = driver.find_element(By.ID, "button")
        element.click()
        break
    except Exception as e:
        if attempt == 2:
            raise
        time.sleep(2)

# 안전한 파싱
try:
    event_name = element.find_element(By.CLASS_NAME, "event").text
except NoSuchElementException:
    event_name = "N/A"
```

---

### 7. Headless 모드

```python
options = webdriver.ChromeOptions()
options.add_argument('--headless')
options.add_argument('--no-sandbox')
options.add_argument('--disable-dev-shm-usage')

driver = webdriver.Chrome(options=options)
```

**사용 시기**:
- 로컬 개발: Headless 끄기 (디버깅)
- 서버 배포: Headless 켜기 (백그라운드)

---

## 📚 핵심 코드 패턴

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

## ❓ 면접 질문

**Q: Selenium과 BeautifulSoup의 차이는?**
> Selenium은 브라우저를 자동화하여 동적 작업을 수행하고, BeautifulSoup은 HTML을 파싱합니다.

**Q: Explicit Wait를 사용하는 이유는?**
> 특정 조건을 만족하면 즉시 진행하여 빠르고 안정적입니다. time.sleep()은 무조건 대기하여 느립니다.

**Q: jQuery UI 조작이 어려운 이유는?**
> JavaScript로 value를 설정하면 내부 상태가 업데이트되지 않습니다. 실제 클릭 이벤트로만 상태가 업데이트됩니다.

---

## 💡 핵심 원칙

1. Anti-Bot: `undetected-chromedriver` 사용
2. 대기: Explicit Wait 사용
3. 팝업: JavaScript로 제거
4. jQuery UI: 사용자 시퀀스 모방
5. 안정성: 재시도 + 예외 처리

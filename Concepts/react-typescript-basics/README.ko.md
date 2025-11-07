# React + TypeScript 기초

## 🎯 한 줄 요약

React와 TypeScript를 함께 사용하여 타입 안전한 프론트엔드 개발하기

---

## 📌 핵심 내용

### 1. 웹 기본 개념

#### A. DOM (Document Object Model)
**개념**: HTML을 JavaScript가 이해할 수 있는 '나무 구조'로 변환

```html
<!-- HTML -->
<div id="app">
  <h1>제목</h1>
  <p>내용</p>
</div>
```

```javascript
// DOM (나무 구조)
document
  └─ div#app
      ├─ h1
      └─ p
```

#### B. AJAX (Asynchronous JavaScript and XML)
**개념**: 페이지 새로고침 없이 데이터만 받아와 UI 일부 업데이트

```typescript
// 페이지 새로고침 없이 데이터 가져오기
const response = await fetch('/api/events');
const data = await response.json();
// UI 일부만 업데이트
setEvents(data);
```

#### C. npm vs npx

| 도구 | 역할 | 비유 |
|------|------|------|
| **npm** | 라이브러리 설치 관리자 | 공구함 📦 |
| **npx** | 일회성 실행 도구 | 공구 렌탈 🔧 |

```bash
# npm: 설치 후 사용
npm install react
npm start

# npx: 설치 없이 실행
npx create-react-app my-app
```

---

### 2. TypeScript의 가치

#### A. "타입은 문서다"

**❌ JavaScript (타입 없음)**:
```javascript
function fetchEvents() {
  // 반환값이 뭔지 모름
  return fetch('/api/events').then(res => res.json());
}

// 사용할 때
const events = await fetchEvents();
console.log(events.event_name);  // 오타! (event_name이 맞나?)
```

**✅ TypeScript (타입 있음)**:
```typescript
interface EconomicEvent {
  id: number;
  date: string;
  time: string;
  event_name: string;  // 명확한 필드명
  country: string;
}

async function fetchEvents(): Promise<EconomicEvent[]> {
  const response = await fetch('/api/events');
  return response.json();
}

// 사용할 때
const events = await fetchEvents();
console.log(events[0].event_name);  // 자동완성 지원!
```

#### B. 컴파일 타임 에러 검출

```typescript
interface Event {
  date: string;
}

const event: Event = {
  dat: "2025-10-20"  // 오타!
};
// 에러: Property 'date' is missing
// 런타임 전에 에러 발견!
```

---

### 3. API 서비스 레이어 분리

#### ❌ 나쁜 방법: 컴포넌트에 직접 작성

```typescript
// CalendarTab.tsx (컴포넌트)
function CalendarTab() {
  const [events, setEvents] = useState([]);

  useEffect(() => {
    // fetch를 컴포넌트에 직접 작성
    fetch('/api/events/?country=US')
      .then(res => res.json())
      .then(data => setEvents(data));
  }, []);
}
```

**문제점**:
- API URL 변경 시 모든 컴포넌트 수정
- 재사용 불가
- 테스트 어려움

#### ✅ 좋은 방법: 서비스 레이어 분리

```typescript
// api/services/calendarService.ts (서비스 레이어)
export const fetchEconomicEvents = async (
  country?: string
): Promise<EconomicEvent[]> => {
  const url = country
    ? `/api/events/?country=${country}`
    : '/api/events/';
  const response = await fetch(url);
  return response.json();
};

// CalendarTab.tsx (컴포넌트)
import { fetchEconomicEvents } from '../api/services/calendarService';

function CalendarTab() {
  const [events, setEvents] = useState<EconomicEvent[]>([]);

  useEffect(() => {
    fetchEconomicEvents('US').then(setEvents);
  }, []);
}
```

**장점**:
- API 변경 시 한 곳만 수정
- 재사용 가능
- 테스트 용이

---

### 4. 데이터 가공

#### A. 날짜별 그룹화 (reduce)

```typescript
const events: EconomicEvent[] = [
  { date: '2025-10-20', event_name: 'GDP' },
  { date: '2025-10-20', event_name: 'CPI' },
  { date: '2025-10-21', event_name: 'FOMC' },
];

// 날짜별로 그룹화
const groupedEvents = events.reduce((acc, event) => {
  if (!acc[event.date]) {
    acc[event.date] = [];
  }
  acc[event.date].push(event);
  return acc;
}, {} as Record<string, EconomicEvent[]>);

// 결과:
// {
//   '2025-10-20': [GDP, CPI],
//   '2025-10-21': [FOMC]
// }
```

#### B. D-Day 계산

```typescript
const calculateDday = (targetDate: string): number => {
  const today = new Date();
  today.setHours(0, 0, 0, 0);  // 시간 정보 제거 필수!

  const target = new Date(targetDate);
  target.setHours(0, 0, 0, 0);

  const diff = target.getTime() - today.getTime();
  return Math.ceil(diff / (1000 * 60 * 60 * 24));
};

// 사용 예시
calculateDday('2025-10-20');  // D-5
```

**핵심**: `setHours(0, 0, 0, 0)`로 시간 정보 제거해야 정확한 날짜 차이 계산

---

### 5. React 핵심 개념

#### A. useState (상태 관리)

```typescript
const [events, setEvents] = useState<EconomicEvent[]>([]);

// events: 현재 상태
// setEvents: 상태 변경 함수
```

#### B. useEffect (부수 효과)

```typescript
useEffect(() => {
  // 컴포넌트 마운트 시 실행
  fetchEconomicEvents().then(setEvents);
}, []);  // 빈 배열: 한 번만 실행
```

#### C. 컴포넌트 구조

```typescript
function CalendarTab() {
  // 1. 상태 선언
  const [events, setEvents] = useState<EconomicEvent[]>([]);

  // 2. 부수 효과 (데이터 로딩)
  useEffect(() => {
    fetchEconomicEvents().then(setEvents);
  }, []);

  // 3. 렌더링
  return (
    <div>
      {events.map(event => (
        <div key={event.id}>
          {event.event_name}
        </div>
      ))}
    </div>
  );
}
```

---

### 6. jQuery 개념 (레거시)

**개념**: 2010년대의 '만능 도구'

```javascript
// jQuery
$('#button').click(function() {
  $.ajax({
    url: '/api/data',
    success: function(data) {
      $('#result').html(data);
    }
  });
});

// 현대 (React + fetch)
<button onClick={() => {
  fetch('/api/data')
    .then(res => res.json())
    .then(data => setResult(data));
}}>
```

**왜 배웠나?**: Investing.com 같은 기존 사이트가 jQuery 사용 (크롤링 시 이해 필요)

---

## 💡 왜 배웠나?

**Dipping 프로젝트**에서:
- 백엔드 API 데이터를 프론트에서 표시
- TypeScript로 타입 안전성 확보
- 날짜별 이벤트 그룹화 & D-Day 계산

---

## 🔧 실제 적용

### Dipping 금융 캘린더 프론트엔드
1. **타입 정의**: `EconomicEvent` interface
2. **서비스 레이어**: `calendarService.ts` 분리
3. **데이터 가공**: 날짜별 그룹화 (reduce)
4. **D-Day 계산**: 시간 정보 제거 후 계산
5. **AJAX**: fetch API로 백엔드 통신

---

## 🔗 관련 프로젝트

- [Dipping 금융 캘린더 크롤러](../../Projects/QuantrumAI/dipping-calendar-crawler/): React + TypeScript 실전 적용

---

## ❓ 면접 예상 질문

**Q1: TypeScript를 사용하는 이유는?**
> A: 타입은 살아있는 문서입니다. 컴파일 타임에 에러를 검출하여 런타임 안정성을 확보하고, IDE의 자동완성으로 개발 생산성이 높아집니다. 백엔드 API 명세와 타입을 일치시켜 데이터 불일치를 방지할 수 있습니다.

**Q2: API 서비스 레이어를 분리하는 이유는?**
> A: 컴포넌트에서 fetch를 직접 호출하면 API URL이나 로직 변경 시 모든 컴포넌트를 수정해야 합니다. 서비스 레이어로 분리하면 한 곳만 수정하면 되고, 재사용성과 테스트 용이성이 높아집니다.

**Q3: D-Day 계산 시 setHours(0, 0, 0, 0)를 사용하는 이유는?**
> A: JavaScript의 Date 객체는 시간 정보(시, 분, 초, 밀리초)를 포함합니다. 시간 정보를 제거하지 않으면 날짜 차이 계산이 부정확해집니다. 예를 들어, 오늘 15시와 내일 10시의 차이는 19시간이지만, 날짜로는 1일 차이여야 합니다.

**Q4: reduce를 사용하는 이유는?**
> A: 배열을 순회하며 누적 값을 만들 때 유용합니다. 날짜별 그룹화처럼 배열을 객체로 변환할 때 reduce가 가장 직관적이고 효율적입니다.

**Q5: AJAX의 장점은?**
> A: 페이지 전체를 새로고침하지 않고 필요한 데이터만 비동기로 가져와 UI 일부만 업데이트할 수 있습니다. 사용자 경험이 향상되고, 네트워크 트래픽이 감소합니다.

---

## 📚 핵심 코드 패턴

### 완전한 React + TypeScript 예시

```typescript
// 1. 타입 정의
interface EconomicEvent {
  id: number;
  date: string;
  event_name: string;
}

// 2. API 서비스
export const fetchEconomicEvents = async (): Promise<EconomicEvent[]> => {
  const response = await fetch('/api/events/');
  return response.json();
};

// 3. 컴포넌트
function CalendarTab() {
  const [events, setEvents] = useState<EconomicEvent[]>([]);
  const [groupedEvents, setGroupedEvents] = useState<Record<string, EconomicEvent[]>>({});

  useEffect(() => {
    // 데이터 로딩
    fetchEconomicEvents().then(data => {
      setEvents(data);

      // 날짜별 그룹화
      const grouped = data.reduce((acc, event) => {
        if (!acc[event.date]) acc[event.date] = [];
        acc[event.date].push(event);
        return acc;
      }, {} as Record<string, EconomicEvent[]>);

      setGroupedEvents(grouped);
    });
  }, []);

  return (
    <div>
      {Object.entries(groupedEvents).map(([date, events]) => (
        <div key={date}>
          <h2>{date}</h2>
          {events.map(event => (
            <div key={event.id}>{event.event_name}</div>
          ))}
        </div>
      ))}
    </div>
  );
}
```

---

**핵심 원칙**:
1. 타입: interface로 명확히 정의
2. API: 서비스 레이어 분리
3. 상태: useState로 관리
4. 부수 효과: useEffect 사용
5. 데이터 가공: reduce, map 활용

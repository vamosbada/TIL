# React Hooks

## 🎯 한 줄 요약

함수형 컴포넌트에서 상태 관리, 사이드 이펙트, DOM 조작을 가능하게 하는 React 함수들

---

## 📌 핵심 내용

### Hook이란?

**정의**: React가 제공하는 특별한 함수들
- 함수형 컴포넌트에 능력(상태 관리, 사이드 이펙트 등) 부여
- `use`로 시작하는 이름
- 함수형 컴포넌트에서만 사용 가능

---

## 🔧 주요 Hook 3개

### 1. useState - 상태 관리자

```typescript
const [count, setCount] = useState(0);
```

**역할**: 화면에 보일 값 관리
- 값 바뀌면 자동으로 화면 업데이트
- React가 알아서 DOM 업데이트 (직접 조작 X)

**언제 쓰나?**
- 사용자 입력값
- 로딩 상태
- 메시지 목록

**예시**:
```typescript
const [messages, setMessages] = useState<Message[]>([]);
const [isLoading, setIsLoading] = useState(false);
```

---

### 2. useEffect - 타이밍 관리자

```typescript
useEffect(() => {
  console.log('컴포넌트 마운트됨!');
}, []);
```

**역할**: 특정 시점에 코드 실행
- 컴포넌트 나타날 때
- 값 바뀔 때
- 컴포넌트 사라질 때

**언제 쓰나?**
- API 호출
- 타이머 설정
- 이벤트 리스너 등록

**예시**:
```typescript
// 컴포넌트 마운트 시 한 번만 실행
useEffect(() => {
  fetchData();
}, []);

// userId 변경될 때마다 실행
useEffect(() => {
  loadUserData(userId);
}, [userId]);
```

---

### 3. useRef - 직접 조작 담당자

```typescript
const inputRef = useRef(null);
inputRef.current.focus();  // DOM 직접 조작!
```

**역할**:
- **유일하게 DOM 직접 조작 가능**
- 값 저장 (리렌더링 안 일으킴)

**언제 쓰나?**
- 포커스 주기
- 스크롤 위치 조작
- 외부 라이브러리 연동
- AbortController 저장

**예시**:
```typescript
// DOM 조작
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();

// 값 저장 (리렌더링 X)
const abortControllerRef = useRef<AbortController | null>(null);
abortControllerRef.current?.abort();
```

---

## 📊 Hook 비교

| Hook | 역할 | DOM 조작? | 리렌더링? | 사용 빈도 |
|------|------|----------|----------|----------|
| `useState` | 상태 관리 | ❌ | ✅ | 95% |
| `useEffect` | 타이밍 제어 | ❌ | ❌ | 90% |
| `useRef` | 직접 조작 | ✅ | ❌ | 40% |

---

## 🎯 실전 사용 팁

### DOM 직접 조작 - 언제?

**❌ 대부분은 useState 사용** (99%):
```typescript
// React 방식 (권장)
const [text, setText] = useState('');
// React가 알아서 화면 업데이트
```

**✅ useRef는 특수한 경우만** (1%):
```typescript
// 포커스, 스크롤 등 React가 못 하는 것
inputRef.current.focus();
```

**핵심 원칙**: "React가 알아서 할게" - 직접 DOM 건드리지 말고 useState 사용!

---

### useEffect 의존성 배열

```typescript
// 1. 빈 배열: 마운트 시 한 번만
useEffect(() => {
  console.log('마운트!');
}, []);

// 2. 의존성 배열: 특정 값 변경 시
useEffect(() => {
  console.log('count 변경!');
}, [count]);

// 3. 배열 없음: 매 렌더링마다 (비추천)
useEffect(() => {
  console.log('매번 실행');
});
```

---

### useCallback으로 함수 최적화

```typescript
const sendMessage = useCallback(async (text: string) => {
  // API 호출 로직
}, [isLoading]);  // isLoading 변경될 때만 함수 재생성
```

**효과**: 불필요한 함수 재생성 방지 → 성능 향상

---

## 💡 왜 배웠나?

**Dipping Pin 챗봇 프로젝트**에서:
- `useState`로 메시지 목록, 로딩 상태 관리
- `useEffect`로 컴포넌트 마운트 시 자동 포커스
- `useRef`로 AbortController 저장 (타임아웃 처리)
- `useCallback`으로 sendMessage 함수 최적화

---

## 🔧 실제 적용

### useChatBot 커스텀 훅 (핵심 패턴)

```typescript
const useChatBot = () => {
  // useState: 상태 관리
  const [messages, setMessages] = useState<Message[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  // useRef: AbortController 저장
  const abortControllerRef = useRef<AbortController | null>(null);

  // useCallback: 함수 최적화
  const sendMessage = useCallback(async (text: string) => {
    setIsLoading(true);
    abortControllerRef.current = new AbortController();

    try {
      const response = await fetch('/api/chat/', {
        signal: abortControllerRef.current.signal
      });
      const data = await response.json();
      setMessages(prev => [...prev, data]);
    } finally {
      setIsLoading(false);
    }
  }, []);

  return { messages, sendMessage, isLoading };
};
```

**패턴**:
- ✅ **useState**: 화면 업데이트 필요한 값
- ✅ **useRef**: 리렌더링 필요 없는 값
- ✅ **useCallback**: 재사용되는 함수

---

## 🔗 관련 프로젝트

- [Dipping Pin 챗봇](../../Projects/QuantrumAI/dipping-pin-chatbot/): React Hooks 실전 적용

---

## ❓ 면접 예상 질문

**Q1: useState와 useRef의 차이는?**
> A: useState는 값이 변경되면 컴포넌트를 리렌더링하지만, useRef는 리렌더링을 일으키지 않습니다. 화면에 보여줄 값은 useState, 내부에서만 쓰는 값(예: AbortController)은 useRef를 사용합니다.

**Q2: useEffect의 의존성 배열이 왜 필요한가요?**
> A: 의존성 배열이 없으면 매 렌더링마다 실행되어 성능 문제가 생깁니다. 빈 배열 `[]`은 마운트 시 한 번만, `[count]`는 count 변경 시에만 실행하여 불필요한 실행을 방지합니다.

**Q3: DOM을 직접 조작해야 할 때는 언제인가요?**
> A: 포커스 주기, 스크롤 위치 조정 등 React가 관리하지 않는 브라우저 API를 사용할 때입니다. 대부분은 useState로 해결하고, useRef는 특수한 경우에만 사용합니다.

**Q4: useCallback은 언제 사용하나요?**
> A: 자식 컴포넌트에 함수를 props로 전달하거나, useEffect의 의존성으로 함수가 필요할 때 사용합니다. 함수가 매번 재생성되면 불필요한 리렌더링이 발생할 수 있기 때문입니다.

---

## 📚 핵심 패턴 요약

```typescript
// 화면 업데이트 필요 → useState
const [value, setValue] = useState(initialValue);

// 특정 시점 실행 → useEffect
useEffect(() => {
  // 로직
}, [dependencies]);

// DOM 직접 조작 → useRef
const ref = useRef(null);
ref.current.focus();

// 함수 최적화 → useCallback
const memoizedFunc = useCallback(() => {
  // 로직
}, [dependencies]);
```

**핵심 원칙**:
1. 화면 업데이트 필요하면 **useState**
2. 특정 타이밍 제어는 **useEffect**
3. DOM 직접 조작은 **useRef** (최소한으로)
4. 함수 재사용은 **useCallback**

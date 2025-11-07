# React Hooks

## 🎯 Summary

Special React functions that enable state management, side effects, and DOM manipulation in functional components

---

## 📌 Key Concepts

### What are Hooks?

**Definition**: Special functions provided by React
- Grant capabilities (state management, side effects, etc.) to functional components
- Names start with `use`
- Only usable in functional components

---

## 🔧 Top 3 Hooks

### 1. useState - State Manager

```typescript
const [count, setCount] = useState(0);
```

**Role**: Manage values displayed on screen
- Screen automatically updates when value changes
- React handles DOM updates (no direct manipulation needed)

**When to use?**
- User input values
- Loading states
- Message lists

**Example**:
```typescript
const [messages, setMessages] = useState<Message[]>([]);
const [isLoading, setIsLoading] = useState(false);
```

---

### 2. useEffect - Timing Manager

```typescript
useEffect(() => {
  console.log('Component mounted!');
}, []);
```

**Role**: Execute code at specific moments
- When component appears (mount)
- When values change
- When component disappears (unmount)

**When to use?**
- API calls
- Timer setup
- Event listener registration

**Example**:
```typescript
// Execute once on mount
useEffect(() => {
  fetchData();
}, []);

// Execute whenever userId changes
useEffect(() => {
  loadUserData(userId);
}, [userId]);
```

---

### 3. useRef - Direct Manipulation Handler

```typescript
const inputRef = useRef(null);
inputRef.current.focus();  // Direct DOM manipulation!
```

**Role**:
- **Only Hook that can directly manipulate DOM**
- Store values (doesn't trigger re-render)

**When to use?**
- Setting focus
- Scroll position manipulation
- External library integration
- Storing AbortController

**Example**:
```typescript
// DOM manipulation
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();

// Store values (no re-render)
const abortControllerRef = useRef<AbortController | null>(null);
abortControllerRef.current?.abort();
```

---

## 📊 Hook Comparison

| Hook | Role | DOM Manipulation? | Re-render? | Usage Frequency |
|------|------|-------------------|------------|-----------------|
| `useState` | State management | ❌ | ✅ | 95% |
| `useEffect` | Timing control | ❌ | ❌ | 90% |
| `useRef` | Direct manipulation | ✅ | ❌ | 40% |

---

## 🎯 Practical Tips

### Direct DOM Manipulation - When?

**❌ Most cases use useState** (99%):
```typescript
// React way (recommended)
const [text, setText] = useState('');
// React handles screen updates automatically
```

**✅ useRef only for special cases** (1%):
```typescript
// Things React can't do: focus, scroll, etc.
inputRef.current.focus();
```

**Core Principle**: "Let React handle it" - Use useState instead of touching DOM directly!

---

### useEffect Dependency Array

```typescript
// 1. Empty array: execute once on mount
useEffect(() => {
  console.log('Mounted!');
}, []);

// 2. Dependencies: execute when specific values change
useEffect(() => {
  console.log('Count changed!');
}, [count]);

// 3. No array: execute on every render (not recommended)
useEffect(() => {
  console.log('Runs every time');
});
```

---

### useCallback for Function Optimization

```typescript
const sendMessage = useCallback(async (text: string) => {
  // API call logic
}, [isLoading]);  // Only recreate when isLoading changes
```

**Effect**: Prevent unnecessary function recreation → Performance improvement

---

## 💡 Why I Learned This

**Dipping Pin Chatbot Project**:
- `useState` for managing message list and loading state
- `useEffect` for auto-focus on component mount
- `useRef` for storing AbortController (timeout handling)
- `useCallback` for optimizing sendMessage function

---

## 🔧 Practical Application

### useChatBot Custom Hook (Core Pattern)

```typescript
const useChatBot = () => {
  // useState: State management
  const [messages, setMessages] = useState<Message[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  // useRef: Store AbortController
  const abortControllerRef = useRef<AbortController | null>(null);

  // useCallback: Function optimization
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

**Pattern**:
- ✅ **useState**: Values that need screen updates
- ✅ **useRef**: Values that don't need re-rendering
- ✅ **useCallback**: Reusable functions

---

## 🔗 Related Projects

- [Dipping Pin Chatbot](../../Projects/QuantrumAI/dipping-pin-chatbot/): Practical application of React Hooks

---

## ❓ Interview Prep Questions

**Q1: What's the difference between useState and useRef?**
> A: useState triggers a re-render when the value changes, but useRef doesn't. Use useState for values that need to be displayed on screen, and useRef for internal values like AbortController that don't need to trigger re-renders.

**Q2: Why do we need the dependency array in useEffect?**
> A: Without a dependency array, the effect runs on every render, causing performance issues. An empty array `[]` runs once on mount, `[count]` runs when count changes. This prevents unnecessary executions.

**Q3: When should we directly manipulate the DOM?**
> A: When using browser APIs that React doesn't manage, like setting focus or adjusting scroll position. For most cases, use useState. Use useRef only for special situations.

**Q4: When should we use useCallback?**
> A: When passing functions as props to child components or when functions are needed as dependencies in useEffect. If functions are recreated every render, it can cause unnecessary re-renders.

---

## 📚 Core Pattern Summary

```typescript
// Screen updates needed → useState
const [value, setValue] = useState(initialValue);

// Execute at specific timing → useEffect
useEffect(() => {
  // logic
}, [dependencies]);

// Direct DOM manipulation → useRef
const ref = useRef(null);
ref.current.focus();

// Function optimization → useCallback
const memoizedFunc = useCallback(() => {
  // logic
}, [dependencies]);
```

**Core Principles**:
1. Need screen updates? **useState**
2. Control specific timing? **useEffect**
3. Direct DOM manipulation? **useRef** (minimize usage)
4. Function reuse? **useCallback**

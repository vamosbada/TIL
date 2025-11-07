# React + TypeScript Basics

## 🎯 Summary

Developing type-safe frontend applications with React and TypeScript

---

## 📌 Key Points

### 1. Web Fundamentals

#### A. DOM (Document Object Model)
**Concept**: Converting HTML into a 'tree structure' that JavaScript can understand

```html
<!-- HTML -->
<div id="app">
  <h1>Title</h1>
  <p>Content</p>
</div>
```

```javascript
// DOM (tree structure)
document
  └─ div#app
      ├─ h1
      └─ p
```

#### B. AJAX (Asynchronous JavaScript and XML)
**Concept**: Updating parts of UI with fetched data without full page reload

```typescript
// Fetch data without page refresh
const response = await fetch('/api/events');
const data = await response.json();
// Update only part of UI
setEvents(data);
```

#### C. npm vs npx

| Tool | Role | Analogy |
|------|------|---------|
| **npm** | Package installation manager | Toolbox 📦 |
| **npx** | One-time execution tool | Tool rental 🔧 |

```bash
# npm: Install then use
npm install react
npm start

# npx: Execute without installation
npx create-react-app my-app
```

---

### 2. Value of TypeScript

#### A. "Types are documentation"

**❌ JavaScript (no types)**:
```javascript
function fetchEvents() {
  // Unknown return type
  return fetch('/api/events').then(res => res.json());
}

// Usage
const events = await fetchEvents();
console.log(events.event_name);  // Typo! (Is event_name correct?)
```

**✅ TypeScript (with types)**:
```typescript
interface EconomicEvent {
  id: number;
  date: string;
  time: string;
  event_name: string;  // Clear field name
  country: string;
}

async function fetchEvents(): Promise<EconomicEvent[]> {
  const response = await fetch('/api/events');
  return response.json();
}

// Usage
const events = await fetchEvents();
console.log(events[0].event_name);  // Auto-completion supported!
```

#### B. Compile-time Error Detection

```typescript
interface Event {
  date: string;
}

const event: Event = {
  dat: "2025-10-20"  // Typo!
};
// Error: Property 'date' is missing
// Catch error before runtime!
```

---

### 3. API Service Layer Separation

#### ❌ Bad approach: Directly in component

```typescript
// CalendarTab.tsx (component)
function CalendarTab() {
  const [events, setEvents] = useState([]);

  useEffect(() => {
    // fetch directly in component
    fetch('/api/events/?country=US')
      .then(res => res.json())
      .then(data => setEvents(data));
  }, []);
}
```

**Problems**:
- All components need modification if API URL changes
- Not reusable
- Difficult to test

#### ✅ Good approach: Separate service layer

```typescript
// api/services/calendarService.ts (service layer)
export const fetchEconomicEvents = async (
  country?: string
): Promise<EconomicEvent[]> => {
  const url = country
    ? `/api/events/?country=${country}`
    : '/api/events/';
  const response = await fetch(url);
  return response.json();
};

// CalendarTab.tsx (component)
import { fetchEconomicEvents } from '../api/services/calendarService';

function CalendarTab() {
  const [events, setEvents] = useState<EconomicEvent[]>([]);

  useEffect(() => {
    fetchEconomicEvents('US').then(setEvents);
  }, []);
}
```

**Advantages**:
- Only one place to modify if API changes
- Reusable
- Easy to test

---

### 4. Data Processing

#### A. Grouping by date (reduce)

```typescript
const events: EconomicEvent[] = [
  { date: '2025-10-20', event_name: 'GDP' },
  { date: '2025-10-20', event_name: 'CPI' },
  { date: '2025-10-21', event_name: 'FOMC' },
];

// Group by date
const groupedEvents = events.reduce((acc, event) => {
  if (!acc[event.date]) {
    acc[event.date] = [];
  }
  acc[event.date].push(event);
  return acc;
}, {} as Record<string, EconomicEvent[]>);

// Result:
// {
//   '2025-10-20': [GDP, CPI],
//   '2025-10-21': [FOMC]
// }
```

#### B. D-Day Calculation

```typescript
const calculateDday = (targetDate: string): number => {
  const today = new Date();
  today.setHours(0, 0, 0, 0);  // Essential to remove time info!

  const target = new Date(targetDate);
  target.setHours(0, 0, 0, 0);

  const diff = target.getTime() - today.getTime();
  return Math.ceil(diff / (1000 * 60 * 60 * 24));
};

// Example usage
calculateDday('2025-10-20');  // D-5
```

**Key**: Must remove time info with `setHours(0, 0, 0, 0)` for accurate date difference

---

### 5. React Core Concepts

#### A. useState (State Management)

```typescript
const [events, setEvents] = useState<EconomicEvent[]>([]);

// events: current state
// setEvents: state update function
```

#### B. useEffect (Side Effects)

```typescript
useEffect(() => {
  // Execute on component mount
  fetchEconomicEvents().then(setEvents);
}, []);  // Empty array: run once
```

#### C. Component Structure

```typescript
function CalendarTab() {
  // 1. State declaration
  const [events, setEvents] = useState<EconomicEvent[]>([]);

  // 2. Side effects (data loading)
  useEffect(() => {
    fetchEconomicEvents().then(setEvents);
  }, []);

  // 3. Rendering
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

### 6. jQuery Concepts (Legacy)

**Concept**: The 'Swiss Army knife' of 2010s

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

// Modern (React + fetch)
<button onClick={() => {
  fetch('/api/data')
    .then(res => res.json())
    .then(data => setResult(data));
}}>
```

**Why learned**: Existing sites like Investing.com use jQuery (needed for crawling understanding)

---

## 💡 Why I Learned This

**Dipping project** needed:
- Display backend API data in frontend
- Ensure type safety with TypeScript
- Group events by date & calculate D-Day

---

## 🔧 Practical Application

### Dipping Financial Calendar Frontend
1. **Type definitions**: `EconomicEvent` interface
2. **Service layer**: Separated `calendarService.ts`
3. **Data processing**: Grouping by date (reduce)
4. **D-Day calculation**: Calculate after removing time info
5. **AJAX**: Backend communication with fetch API

---

## 🔗 Related Projects

- [Dipping Financial Calendar Crawler](../../Projects/QuantrumAI/dipping-calendar-crawler/): Real-world React + TypeScript application

---

## ❓ Interview Prep Questions

**Q1: Why use TypeScript?**
> A: Types are living documentation. Detecting errors at compile-time ensures runtime stability, and IDE auto-completion improves development productivity. Aligning types with backend API specifications prevents data mismatches.

**Q2: Why separate API service layer?**
> A: If components directly call fetch, all components need modification when API URLs or logic change. Separating into a service layer means only one place needs updating, improving reusability and testability.

**Q3: Why use setHours(0, 0, 0, 0) for D-Day calculation?**
> A: JavaScript's Date object includes time information (hours, minutes, seconds, milliseconds). Without removing time info, date difference calculations become inaccurate. For example, today at 3PM and tomorrow at 10AM is a 19-hour difference, but should be a 1-day difference in dates.

**Q4: Why use reduce?**
> A: It's useful for iterating through arrays while building accumulated values. For operations like grouping by date that transform arrays into objects, reduce is the most intuitive and efficient.

**Q5: What are AJAX's advantages?**
> A: Instead of refreshing the entire page, only necessary data is fetched asynchronously to update parts of the UI. This improves user experience and reduces network traffic.

---

## 📚 Core Code Patterns

### Complete React + TypeScript Example

```typescript
// 1. Type definition
interface EconomicEvent {
  id: number;
  date: string;
  event_name: string;
}

// 2. API service
export const fetchEconomicEvents = async (): Promise<EconomicEvent[]> => {
  const response = await fetch('/api/events/');
  return response.json();
};

// 3. Component
function CalendarTab() {
  const [events, setEvents] = useState<EconomicEvent[]>([]);
  const [groupedEvents, setGroupedEvents] = useState<Record<string, EconomicEvent[]>>({});

  useEffect(() => {
    // Data loading
    fetchEconomicEvents().then(data => {
      setEvents(data);

      // Group by date
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

**Core Principles**:
1. Types: Define clearly with interfaces
2. API: Separate service layer
3. State: Manage with useState
4. Side effects: Use useEffect
5. Data processing: Utilize reduce, map

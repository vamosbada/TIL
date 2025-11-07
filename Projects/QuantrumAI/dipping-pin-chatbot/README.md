# Dipping - Pin Chatbot System

## 🎯 Project Background

**Project Name**: Pin (Chatbot that provides pinpoint information)

**Problem**: Stock beginners struggle to understand terminology, market conditions, and news

**Solution**: AI chatbot providing customized, easy-to-understand information for beginners

**Core Value**: "Providing pinpoint information for stock beginners"

**Clear Definition**: Chatbot (Not Agent!)
- ✅ **Chatbot**: Information retrieval and delivery only
- ❌ **Agent**: Performs actions autonomously (auto-trading, etc.)
- **Financial Compliance**: No trading or action execution

---

## 🛠 Tech Stack

### LLM
- **Model**: Gemini 2.0 Flash Exp
- **Method**: External API call (Google Official)
- **Cost**: Free (1,500 requests/day)
- **Integration**: Agent/Chain configuration with LangChain

### Backend
- **Framework**: Django 5.1
- **Language**: Python 3.10
- **Virtual Environment**: conda (dipping)
- **ORM**: Django ORM + PostgreSQL

### Frontend
- **Framework**: React 19 + TypeScript
- **Build Tool**: Vite
- **State Management**: Context API

### Core Libraries
- **LangChain 0.2.16**: Agent/Chain configuration (downgraded from 1.0 for compatibility)
- **langchain-google-genai**: Gemini integration
- **PostgreSQL**: Main DB (news, sentiment analysis results)
- **Beginner's Dictionary**: JSON-based 250 terms (FAISS unnecessary)

### Data Sources
1. **Beginner's Dictionary**: 250 financial terms (JSON, simple search)
2. **Korea Investment API**: Real-time stock data
3. **News API**: Stock-related news (English)
4. **EODHD API**: Financial calendar data

---

## 💻 My Role

**100% ownership of Pin chatbot within the team**
- System architecture design
- LangChain + Gemini integration
- ReAct Agent implementation
- Django API development
- React chatbot UI development

**Role Classification**: Backend 70% + ML 30%
- **Backend Engineer (Main)**: Django API, RAG pipeline, vector DB setup
- **ML Engineer (Sub)**: Embedding model selection, prompt engineering
- **Key Insight**: RAG is increasingly absorbed into backend domain

---

## 🏗 System Architecture

### Overall Structure: One Gemini + Multiple LangChain Pipelines

```
            Gemini 2.0 Flash (Single Model)
                    ↑
    ┌───────────────┼───────────────────┐
    │               │                   │
LangChain 1     LangChain 2        LangChain 3
(Chatbot)       (Sentiment)         (Summary)
    ↑               ↑                   ↑
/api/chatbot  /api/news/sentiment  /api/market/summary
    ↑               ↑                   ↑
React Chatbot  React News Page    React Dashboard
```

**Core Concept**:
- Single Gemini model shared across multiple LangChain pipelines
- Each API has independent LangChain configuration
- Customized prompts per use case

---

## 🔧 Key Implementation

### 1. Chatbot Flow (ReAct Agent)

**ReAct**: Reasoning + Acting pattern

**Core Pattern**: Thought → Action → Observation → Answer

**Example 1: Terminology Question**
```
User: "What is PER?"
         ↓
[Thought] Question Classification
"Terminology question → Search Vector DB"
         ↓
[Action] Execute Tool
Search "PER" in Vector DB
         ↓
[Observation] Check Result
"PER = Price-to-Earnings Ratio..."
         ↓
[Thought] Decide Explanation Style
"Explain in beginner-friendly way"
         ↓
[Answer] Gemini Response Generation
"PER is Price-to-Earnings Ratio. Simply put..."
         ↓
Deliver to User
```

**Example 2: Real-time Information Question**
```
User: "What's today's economic indicator?"
         ↓
[Thought] Question Classification
"Need to query economic calendar DB"
         ↓
[Action] DB Query
Fetch today's data from PostgreSQL
         ↓
[Observation] Check Result
"CPI announcement scheduled"
         ↓
[Thought] Decide Explanation Style
"Explain in beginner-friendly way"
         ↓
[Answer] Gemini Response Generation
"Today's US Consumer Price Index..."
         ↓
Deliver to User
```

#### A. Question Classification (Thought)

```python
# Automatic question type detection
if "term" in query:
    tool = "Vector DB"  # Beginner's dictionary
elif "stock price" in query:
    tool = "Korea Investment API"  # Real-time quotes
elif "news" in query:
    tool = "News API"
```

#### B. Data Retrieval (Action)

```python
# LangChain Tool definition
from langchain.tools import Tool

tools = [
    Tool(
        name="BeginnerDictionary",
        func=vector_db_search,
        description="Stock terminology explanation"
    ),
    Tool(
        name="StockPriceQuery",
        func=stock_api_call,
        description="Real-time stock data"
    ),
    Tool(
        name="NewsSearch",
        func=news_api_call,
        description="Latest news"
    )
]
```

#### C. ReAct Agent Configuration

```python
from langchain.agents import create_react_agent
from langchain_google_genai import ChatGoogleGenerativeAI

# Gemini model
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash-exp",
    temperature=0.7
)

# Create ReAct Agent
agent = create_react_agent(
    llm=llm,
    tools=tools,
    prompt=react_prompt  # "Explain easily to beginners"
)
```

---

### 2. News Sentiment Analysis Flow

```
Collect English News (News API)
         ↓
[LangChain PromptTemplate]
"Translate this news to Korean and
 analyze positive/negative/neutral ratios"
         ↓
Send to Gemini
         ↓
Parse Result JSON
{
  "translated": "Translated content",
  "sentiment": {
    "positive": 0.7,
    "negative": 0.2,
    "neutral": 0.1
  }
}
         ↓
Store in PostgreSQL
```

#### Implementation Code

```python
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain

# Prompt template
prompt = PromptTemplate(
    input_variables=["news_text"],
    template="""
    Translate the following news to Korean and analyze sentiment:
    {news_text}

    Result in JSON format:
    {{
      "translated": "translation",
      "sentiment": {{"positive": 0.0, "negative": 0.0, "neutral": 0.0}}
    }}
    """
)

# Create LLMChain
chain = LLMChain(llm=llm, prompt=prompt)

# Execute
result = chain.run(news_text="Apple stock rises...")
```

---

### 3. Market Summary Flow

```
Collect News + Major Index Data
         ↓
[LangChain PromptTemplate]
"Summarize today's market in 3 lines
 for stock beginners to understand"
         ↓
Send to Gemini
         ↓
Generate 3-Line Summary
"1. US market rises...
 2. Semiconductor strength...
 3. Rate cut expectations..."
         ↓
Display on React Dashboard
```

#### Implementation Code

```python
prompt = PromptTemplate(
    input_variables=["news", "indices"],
    template="""
    Today's market situation:
    News: {news}
    Indices: {indices}

    Summarize in 3 lines for beginners to understand.
    """
)

chain = LLMChain(llm=llm, prompt=prompt)
summary = chain.run(news=news_data, indices=index_data)
```

---

## 🔀 3 Independent API Structure

**API 1: `/api/chatbot`** - User conversation responses (ReAct Agent)
- Vector DB (beginner's dictionary), Korea Investment API, News API integration

**API 2: `/api/news/sentiment`** - News translation + sentiment analysis
- PromptTemplate + LLMChain for positive/negative/neutral analysis

**API 3: `/api/market/summary`** - Today's market summary
- News + major indices summarized in 3 lines

---

## 📊 Data Flow

```
[User Question]
       ↓
[Django API]
       ↓
[LangChain Agent] ← Question Classification (Thought)
       ↓
[Tool Execution] ← Data Collection (Action)
  ├→ Vector DB
  ├→ Korea Investment API
  └→ News API
       ↓
[Gemini] ← Response Generation
       ↓
[Django Response]
       ↓
[React UI]
       ↓
[User]
```

---

## 🔑 Key Technical Decisions

### 0. Understanding RAG Essence

**Key Realization**: DB queries, API calls, web search → All are RAG!

**RAG = Retrieval (search) + Augmented (enhance) + Generation (create)**

```
User Question
    ↓
Search for information externally (from anywhere!)
    ↓
Pass search results to GPT
    ↓
GPT generates answer based on search results
```

**Dipping Chatbot's RAG Components**:
1. **Internal Knowledge DB** (Vector DB): 250 beginner's dictionary terms
2. **API Calls**: Korea Investment API (real-time stock), News API
3. **DB Queries**: PostgreSQL (economic calendar, sentiment analysis)

**Difference**: Only "where to search" varies - all follow RAG pattern!

**RAG + Prompt Engineering = Optimal Combination**

```python
# 1. RAG: Accurate information retrieval
stock_data = fetch_stock_price("Samsung Electronics")

# 2. Prompt: Convert to easy explanation
prompt = f"""
You are a teacher for stock beginners.

[Real-time Data]
Samsung Electronics current price: {stock_data['price']} KRW

Explain this in a way middle school students can understand.
"""

answer = llm.generate(prompt)
```

→ Not independent but **mutually complementary**!

---

### 1. LangChain Version: 0.2.16

**Issue**: LangChain 1.0 had compatibility problems

**Solution**:
```bash
pip install langchain==0.2.16
pip install langchain-google-genai
```

**Reasons**:
- Stable documentation and examples
- `create_react_agent` API support
- Smooth Gemini integration

---

### 2. Beginner's Dictionary: Simple Search (No FAISS)

**Scale**: 250 financial terms (JSON-based)

**Key Decision**: FAISS unnecessary
- Simple keyword matching is sufficient (0.001s)
- Each term includes definition + example (examples have high impact)
- Scaling path: Django ORM → Elasticsearch if needed

---

### 3. React useChatBot Custom Hook

**Core Feature**: 30-second timeout handling with AbortController

```typescript
const useChatBot = () => {
  const [messages, setMessages] = useState<Message[]>([]);
  const abortControllerRef = useRef<AbortController | null>(null);

  const sendMessage = useCallback(async (text: string) => {
    abortControllerRef.current = new AbortController();
    setTimeout(() => abortControllerRef.current?.abort(), 30000);

    const response = await fetch('/api/chat/', {
      signal: abortControllerRef.current.signal
    });
    // Loading message management + error handling
  }, []);

  return { messages, sendMessage };
};
```

**Key Patterns**:
- ✅ Function optimization with useCallback
- ✅ Store AbortController in useRef (prevent re-renders)
- ✅ Automatic 30-second timeout handling

---

### 4. Gemini API Choice (Instead of Local LLM)

**Key Decision**: Gemini API instead of local LLM (LLaMA, GPT-OSS)

**Reasons**:
- Hardware constraints (LLaMA 120B requires 22GB RAM, MacBook has 18GB)
- Fast speed + free tier (1,500 requests/day)
- Unlimited concurrent processing (local = 1 user only)
- Team collaboration friendly (reproducibility)

**Trade-off**: API dependency vs practicality → chose practicality

---

## 🎨 Frontend Architecture Patterns

### ChatBotWrapper - Separation of Concerns (SoC)

**Problem**: Repeating chatbot display conditions on every page

**❌ Without Wrapper**:
```tsx
// Duplicate on each page
{(location.pathname === '/main' ||
  location.pathname === '/stock' ||
  location.pathname === '/news') && <ChatBot />}
```

**✅ With Wrapper**:
```tsx
<ChatBotWrapper allowedPages={['/main', '/stock', '/news']} />
```

**Implementation**:
```tsx
interface ChatBotWrapperProps {
  allowedPages: string[];
}

const ChatBotWrapper: React.FC<ChatBotWrapperProps> = ({ allowedPages }) => {
  const location = useLocation();

  if (!allowedPages.includes(location.pathname)) {
    return null;  // Don't render on disallowed pages
  }

  return <ChatBot />;
};
```

**Benefits**:
- ✅ **Separation of Concerns**: ChatBot handles "conversation", Wrapper handles "routing"
- ✅ **Code Reuse**: DRY principle (Don't Repeat Yourself)
- ✅ **Easy Maintenance**: Add pages by updating allowedPages only
- ✅ **Better Readability**: Clear intent

**Analogy**:
- ChatBot = Employee (does the work)
- ChatBotWrapper = Access Control (decides where to deploy)

---

## 🏗 Django API Integration Structure

### 2-Stage URL Routing

```
POST /api/chat/
    ↓
Stage 1: config/urls.py
path('api/', include('chatbot_app.urls'))
→ "Found api/! Route to chatbot_app"
    ↓
Stage 2: chatbot_app/urls.py
path('chat/', views.chat, name='chat')
→ "Found chat/! Execute views.chat()"
    ↓
Process in views.py
```

**Core Principle**: Manage URLs in stages
- **Benefits**: Independent management per app, clean code organization
- **Using name**: Manage by alias instead of hardcoding URLs

### Singleton Pattern Implementation

**Problem**: Creating chatbot on every request → Slow

**Solution**: Create chatbot instance once and reuse

```python
# views.py
_chatbot_instance = None  # Global variable

def get_chatbot():
    global _chatbot_instance
    if _chatbot_instance is None:
        # Create only once (heavy operation)
        _chatbot_instance = create_chatbot()
    return _chatbot_instance  # Reuse

@api_view(['POST'])
def chat(request):
    chatbot = get_chatbot()  # Fast!
    response = chatbot.invoke({"input": message})
    return Response({"response": response['output']})
```

**Effects**:
- ✅ Gemini API connection once only
- ✅ Agent creation once only
- ✅ Significantly improved response speed

### Development vs Production Strategy

**Development Stage** (Current):
```python
@permission_classes([AllowAny])  # Test without login
```

**Production Stage** (Before deployment):
```python
@permission_classes([IsAuthenticated])  # Token required
```

**Rationale**:
- Development: Quick validation of chatbot logic only
- Production: Connect auth system (switch with 1 line change)

---

## 💡 Key Takeaways

### Technical Learnings

**1. Power of LangChain**
- Dynamic tool selection with Agent pattern
- Increased reusability with PromptTemplate
- Complex flow composition with Chains

**2. ReAct Pattern**
- Repeating Thought → Action → Observation
- AI selects tools autonomously
- Transparent decision-making process

**3. One Model, Multiple Uses**
- Single Gemini handles chatbot, sentiment analysis, summary
- Configure use-case pipelines with LangChain
- Cost savings + consistency

**4. Pragmatic Technical Choices**
- 250 terms don't need FAISS → simple search
- Local LLM vs API → chose API for speed and quality
- LangChain 1.0 issues → downgraded to 0.2.16
- Practicality over perfection

**5. Importance of Timeout Handling**
- 30-second timeout with AbortController
- Protect user experience from LLM response delays
- Frontend resilience against backend failures

**6. Prototype-First Strategy**
- Validate entire flow with hardcoded data first
- Connect real APIs after confirming functionality
- Top-Down approach: Small whole → Large whole
- 80% understanding is enough (learn rest in practice)

### Design Learnings

**1. API Separation**
- Separate chatbot, sentiment, summary as independent APIs
- Each API has independent LangChain configuration
- Easy maintenance

**2. Beginner-Centric Design**
- "Explain simply" prompt as core
- Build terminology dictionary Vector DB
- Provide customized information

---

## 🔗 Related Concepts

- [React Hooks](../../../Concepts/react-hooks/): useState, useEffect, useRef and other React Hooks basics
- [Django Backend Patterns](../../../Concepts/django-backend-patterns/): Django ORM and API development
- [Django REST Framework Basics](../../../Concepts/django-rest-framework-basics/): DRF ViewSet and Serializer

---

## 🔗 Next Steps

### Phase 1 (Current)
- [x] System architecture design
- [x] LangChain 0.2.16 environment setup
- [x] Beginner's dictionary data structure design (250 terms)
- [x] React useChatBot custom hook implementation
- [ ] Complete ReAct Agent implementation
- [ ] Complete beginner's dictionary embedding

### Phase 2
- [ ] Complete news sentiment analysis API
- [ ] Complete market summary API
- [ ] Integration testing of 3 APIs

### Phase 3
- [ ] React chatbot UI development
- [ ] Dashboard integration
- [ ] User feedback incorporation

---

## ❓ Interview Prep Questions

**Q1: What is ReAct Agent?**
> A: An AI pattern that repeats Reasoning and Acting. AI analyzes questions (Thought), selects and executes necessary tools (Action), checks results (Observation), then generates responses. For example, when asked "Samsung stock price," AI autonomously selects and calls the stock price API.

**Q2: Why use LangChain?**
> A: To easily integrate LLMs and compose complex flows. Agent pattern for dynamic tool selection, PromptTemplate for reusable prompt management, and Chains to connect multiple steps. More structured and maintainable than directly calling Gemini.

**Q3: How does one Gemini handle multiple functions?**
> A: By configuring use-case pipelines with LangChain. Chatbot uses ReAct Agent, sentiment analysis uses LLMChain with PromptTemplate, and summary also uses LLMChain but with different prompts calling Gemini. Same model but different prompts yield different results.

**Q4: Why is Vector DB necessary?**
> A: For semantic search by embedding stock terminology dictionary (beginner's dictionary). When users ask about "PER," Vector DB searches similar terms to provide accurate explanations. Enables meaning-based search beyond simple keyword matching.

**Q5: Why create 3 independent APIs?**
> A: To allow independent scaling of each function. Chatbot, sentiment analysis, and summary have different purposes and requirements. Putting everything in one API becomes complex and difficult to maintain. Independent APIs are easier to test and can be reused in other projects later.

---

## 📂 Project Structure

```
dipping-pin-chatbot/
├── backend/
│   ├── chatbot/
│   │   ├── agents/
│   │   │   └── react_agent.py
│   │   ├── tools/
│   │   │   ├── vector_db.py
│   │   │   ├── stock_api.py
│   │   │   └── news_api.py
│   │   └── views.py
│   ├── sentiment/
│   │   ├── chains/
│   │   │   └── sentiment_chain.py
│   │   └── views.py
│   ├── summary/
│   │   ├── chains/
│   │   │   └── summary_chain.py
│   │   └── views.py
│   └── manage.py
└── frontend/
    ├── src/
    │   ├── components/
    │   │   └── Chatbot.tsx
    │   └── api/
    │       └── services/
    │           └── chatbotService.ts
    └── package.json
```

---

**Current Status**: 🔄 Phase 1 In Progress (Architecture design complete)
**Ownership**: Bada (100% ownership of Pin chatbot within QuantrumAI team)

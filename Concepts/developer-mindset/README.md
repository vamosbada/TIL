# Developer Mindset

## 🎯 Summary

Attitudes and problem-solving approaches to adopt for future development

---

## 📌 Core Attitudes

### 1. Break Down Complex Problems

**Attitude**: "Don't try to do everything at once"

#### Bad Habit
```
"Let's crawl, save to DB, build API, and do frontend all at once!"
→ Don't know where it's stuck
→ Debugging hell
→ No sense of achievement
```

#### Good Habit
```
1. Crawling first
   → Save to TXT and verify data
   → "This step works!" ✅

2. DB storage
   → Check with queries
   → "DB saving works!" ✅

3. Build API
   → Test with Postman
   → "API works!" ✅

4. Frontend integration
   → Complete! ✅
```

**Learned from Dipping Project**:
- Broke down jQuery UI Datepicker when it didn't work
- Tested "Open Datepicker → Click date → Apply button" one by one
- Move to next step only after each succeeds

**For Future Development**:
- When facing overwhelming features → Write steps on paper
- Complete and verify one step at a time
- Progress while building momentum

---

### 2. AI is a Tool, I'm the Driver

**Attitude**: "Don't depend on AI, utilize AI"

#### Bad Habit
```
Me: "Make a crawler"
AI: (300 lines of code)
Me: (Copy-paste) → Run → Error → "Why doesn't it work?" → Ask AI again
```

#### Good Habit
```
Me: "jQuery UI Datepicker manipulation with Selenium isn't working.
     Setting value doesn't update internal state.
     Tried: 1) JS value setting 2) send_keys
     Both failed. Need step-by-step approach."

AI: (Specific answer)

Me: (Read and understand code) → (Test) → (Ask again if doesn't work)
```

**Learned from Dipping Project**:
- Blindly copy-pasting AI code → Went wrong
- Understanding and testing AI code → Success
- Habit of thinking "Why did it do this?"

**For Future Development**:
- Be specific when asking AI (problem, attempts, desired outcome)
- Always read, understand, and test AI code
- "Don't copy-paste without understanding"

---

### 3. Errors are Teachers, Not Enemies

**Attitude**: "Don't panic when errors occur"

#### Bad Habit
```
[Error occurs]
→ "Ugh, annoying" → Give up
→ "Should I ask AI?" (immediately)
```

#### Good Habit
```
[Error occurs]
→ "Okay, what's the problem?" (calm)
→ Read error message carefully
→ Google (copy error message)
→ If doesn't work, break down the problem
→ Still doesn't work, ask AI/teammates specifically
```

**Learned from Dipping Project**:
- jQuery UI failure → "Why doesn't it work?" → Understood internal workings
- Realized "Setting value ≠ User action"
- Errors led to deeper understanding

**For Future Development**:
- Read error message first
- Find similar cases through Googling
- Treat each error as a learning opportunity

---

### 4. Completion Before Perfection

**Attitude**: "Make it work first, then improve"

#### Bad Habit
```
"Need perfect design..."
→ Think for a week
→ Never start
→ Deadline approaches
```

#### Good Habit
```
Day 1: Write working code
Day 2: Refactor (separate functions, clean names)
Day 3: Optimize (improve performance)
→ Complete!
```

**Learned from Dipping Project**:
- Started with print() debugging → Later switched to logging
- Started with hardcoding → Later moved to config files
- Improvement points became visible after completion

**For Future Development**:
- Execution before design
- "Make it work" → "Make it pretty" → "Make it fast"
- Better to complete and improve than never start due to perfectionism

---

### 5. Understand the Big Picture

**Attitude**: "See the forest, then the trees"

#### Bad Habit
```
"Just need to know my code"
→ When problem occurs, don't know where it's stuck
→ Debugging takes forever
```

#### Good Habit
```
[Crawling] → [DB] → [API] → [Frontend] → [User]

Problem: "No data showing on frontend"

→ Crawling: Check TXT → ✅ Data exists
→ DB: Query → ✅ Saved
→ API: Postman → ❌ 500 error!
→ Found: serializer configuration error

→ Quick solution!
```

**Learned from Dipping Project**:
- Understanding data flow → Find problems quickly
- Know input/output of each step
- Discover optimization points

**For Future Development**:
- Draw complete flow when starting new project
- Understand "What's input, what's output?" at each step
- When problem occurs, check step by step following the flow

---

### 6. Stuck? Google → Break Down → Ask

**Attitude**: "Don't just struggle alone, get help systematically"

#### Resolution Order
```
1. Read error message carefully
   → Identify what error it is

2. Google (copy error message)
   → Find similar cases
   → StackOverflow, official docs

3. Break down the problem
   → Create minimal reproduction code
   → "Where exactly is it stuck?"

4. Ask AI/teammates
   → Problem situation + attempts + desired result
   → Be specific!
```

**Learned from Dipping Project**:
- jQuery UI stuck → Googling → Official docs → AI question
- "Stuck" → Immediately AI (X)
- "Understand before moving on" (O)

**For Future Development**:
- If stuck for 30 minutes, search
- If searching for 1 hour doesn't work, ask
- Reduce time struggling alone

---

### 7. Document What You Learn Immediately

**Attitude**: "Help your future self"

#### Bad Habit
```
"Solved jQuery UI today!"
→ Don't document
→ Same problem 3 months later
→ "How did I do it...?" (Can't remember)
```

#### Good Habit
```
"Solved jQuery UI today!"
→ Document immediately in README.md
   - Problem situation
   - Attempted methods
   - Solution
   - Why it worked

→ 3 months later, solved in 5 minutes using docs
```

**Learned from Dipping Project**:
- Forget without documentation
- Writing TIL → Interview prep too
- Documentation is a gift to future self

**For Future Development**:
- Document immediately after solving blocking issues
- Make TIL a habit
- "I'll write later" → That later never comes

---

### 8. Distinguish if/else vs try/except

**Attitude**: "Distinguish expected situations from unexpected errors"

#### if/else (Expected conditions)
```python
# Logical branching
if country == "US":
    filter_us_events()
else:
    filter_all_events()
```

#### try/except (Unexpected errors)
```python
# Operations that can fail
try:
    data = crawl_website()
except NetworkError:
    logger.error("Network error")
    retry()
```

**Learned from Dipping Project**:
- Crawling has high failure probability
- Without try/except, entire process stops on one error
- Ensure stability

**For Future Development**:
- Network, file IO, external API → try/except
- Conditional branching → if/else
- Habit of thinking "Can this fail?"

---

## 💡 Why I Learned This

**Dipping project** taught:
- jQuery UI stuck → Solved by breaking down
- AI dependency → Became proactive
- Fear of errors → Turned into learning opportunities
- Perfectionism → Prioritized completion

---

## 🔧 Practical Application

### Dipping Financial Calendar Crawler
1. **Break down**: Crawling → TXT → DB → API → Frontend
2. **Utilize AI**: Specific questions + code review
3. **Welcome errors**: jQuery UI failure → Deep understanding
4. **Completion first**: Make it work → Refactor
5. **Documentation**: Write TIL

---

## 🔗 Related Projects

- [Dipping Financial Calendar Crawler](../../Projects/QuantrumAI/dipping-calendar-crawler/): Real-world mindset application

---

## ❓ Interview Prep Questions

**Q1: How do you approach complex problems?**
> A: I break large problems into smaller steps. In the Dipping project, I divided "crawl and display on calendar" into ①crawling ②DB storage ③API ④frontend. Verifying each step narrowed the problem scope and provided sense of achievement.

**Q2: How do you utilize AI in development?**
> A: AI is a tool and I'm the driver. I ask specific questions about problems and attempts, and always understand and test AI-suggested code. Copy-pasting without understanding makes future maintenance difficult.

**Q3: What do you do when stuck?**
> A: First, I read the error message carefully and Google it. If not solved in 30 minutes, I break the problem down further and create minimal reproduction code. If still stuck, I ask specific questions. I try to reduce time struggling alone.

**Q4: How do you handle errors?**
> A: Errors are learning opportunities. jQuery UI Datepicker manipulation kept failing, but digging into the cause taught me "setting value ≠ user action." I approach errors with "Why did this happen?" rather than fear.

**Q5: Do you pursue perfect code?**
> A: Completion before perfection. In Dipping project, I started with print() debugging but made it work first, then improved to logging. Better to complete and improve than never start due to perfectionism.

---

## 📚 Future Development Checklist

### When Starting
- [ ] Broke down problem into small steps?
- [ ] Understood complete data flow?
- [ ] Starting from first step (no perfectionism)

### During Development
- [ ] Understood before copy-pasting AI code?
- [ ] Read error message carefully?
- [ ] Googled if stuck for 30 minutes?
- [ ] Ensured stability with try/except?

### After Completion
- [ ] Tested each step?
- [ ] Documented learnings? (TIL)
- [ ] Noted parts needing refactoring?

### When Stuck
- [ ] Broke problem down further?
- [ ] Created minimal reproduction code?
- [ ] Prepared specific question?

---

**One-line Summary**:
> "Break down, understand, document, and complete"

**Final Advice**:
- No perfect developer exists
- Errors are growth opportunities
- Just be slightly better tomorrow than today
- Documentation habit changes your future

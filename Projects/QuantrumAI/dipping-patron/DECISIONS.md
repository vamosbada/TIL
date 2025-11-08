# Patron - Key Decision-Making Process (Q&A)

## 📋 Table of Contents
1. Data Design Decisions
2. User Experience (UX) Decisions
3. Differentiation Strategy
4. Additional Feature Design
5. Mistakes & Confusions
6. Legal/Ethical Considerations

---

## 1. Data Design Decisions

### Q: Why is normalization necessary?

**Core Concept**:
```python
# Before normalization
Apple: $150 → $153 (small absolute value)
Tesla: $2000 → $2060 (large absolute value)
→ Completely different chart shapes!

# After normalization (0~1)
Apple: 0.0 → 0.02
Tesla: 0.0 → 0.02
→ Both recognized as "+2% pattern"
```

**Important**:
- Normalization must happen at the "numeric stage" first
- Pixel normalization after image generation is too late

**Key Learning**:
> Data preprocessing order: Normalize → Visualize → Train

---

## 2. User Experience (UX) Decisions

### Q: Input Processing - Who crops the axes?

**Decision: User crops, model handles background lines**

**Rationale**:
- Phone capture editing easily handles axis cropping
- Improves accuracy (noise removal)
- Background lines/grids can be ignored by CNN (robustness)

**UI Guidance**:
```
✅ Crop to show only candlestick chart
✅ Remove axes, dates, price numbers
✅ Background lines/grids are OK
```

**Key Learning**:
> Clearly distinguish what users can do vs what the system should do

---

### Q: How many TOP N results?

**Decision: TOP 3**

**Rationale**:
- Target audience (beginners): Too many overwhelms
- 3 allows comparison + pattern recognition
- Mobile UI consideration

---

## 3. Differentiation Strategy

### Q: How is this different from existing "stock chart pattern recognition"?

**Existing Projects**:
1. **Pattern Classification**: "This is Head & Shoulders" → So what?
2. **Price Prediction**: "Tomorrow $152" → Regulatory risk + trust issues

**Our Project (Differentiation)**:
```
Search-based + Historical Cases

Output:
"95% similar to NVIDIA 2023-05
 That pattern led to +45% rise after 3 months

 Industry: Technology - Semiconductors
 Volatility: High
 Market Correlation: 0.85"
```

**5 Key Differentiators**:
1. Search, not classify
2. Cross-stock/industry search (Tesla ↔ Netflix)
3. Concrete historical cases
4. Statistical insights (historical case analysis)
5. Beginner-friendly full-stack service

**Key Learning**:
> Clear distinction between "information provision" and "investment advice"

---

## 4. Additional Feature Design

### Volatility Score

**Definition**: Average daily price fluctuation over 12 weeks

```python
volatility = ((High - Low).mean() / Close.mean()) × 100

# Classification
Low: <2% (stable, beginner-friendly)
Medium: 2~4%
High: >4% (roller coaster, caution)
```

**Meaning**: "How much this pattern fluctuated"
- High: Caution for risk-averse beginners
- Low: Relatively stable

**Value**: Same return rate feels different with different volatility

---

### Market Correlation

**Definition**: How closely it moved with NASDAQ/S&P 500 index

```python
# Pearson correlation coefficient (-1 ~ 1)
import numpy as np
correlation = np.corrcoef(stock_returns, index_returns)[0, 1]

# Interpretation
1.0: Perfectly identical
0.8~1.0: Moves almost together (follows market) ✅
0.5~0.8: Some influence
0~0.5: Independent movement (ignores market)
```

**Beginner-friendly Expression**:
```
High: "Moved almost identically with overall market"
Low: "Moved independently regardless of market"
```

**Implementation Difficulty**: Medium (3-4 hours)

---

### Industry Tag Strategy

**Decision: Display only (no filtering)**

**Rationale**:
- Cross-industry insights = core value
- Discoveries like "Tesla similar to Netflix?" are the real fun

**Automated Collection**:
```python
import yfinance as yf

ticker = yf.Ticker("AAPL")
sector = ticker.info['sector']  # 'Technology'
industry = ticker.info['industry']  # 'Consumer Electronics'

# Zero labeling! Just 1 additional API call
```

**UI Display**:
```
1. 🔧 NVIDIA (Technology) - 95%
2. 📱 Apple (Technology) - 93%
3. 🎬 Netflix (Communication) - 92% 🌟
   💡 Found in different industry too!
```

**Key Learning**:
> Use data for "insights" not "filters"

---

## 5. Mistakes & Confusions

### Axis Removal vs Normalization

**Misconception**: "Do I not need normalization if I remove axes?"

**Truth**: Completely separate!
- Axis removal = visual image processing
- Normalization = numeric data preprocessing (essential!)

---

### Image First vs Normalization First

**Correct Order**: Normalize → Generate images → Train

**Rationale**: Normalization after image generation is too late

---

### Crop vs Padding

**Crop**: Large image → extract center only

**Padding**: Small image → add margins

---

## 6. Legal/Ethical Considerations

### Expression Guidelines

```
❌ Prohibited (Investment Advice):
"Buy this stock"
"100% will rise"

✅ Allowed (Information Provision):
"This pattern existed in the past"
"Statistically showed this tendency"
```

### Disclaimer Required

```
⚠️ Investment Notice
Past patterns do not guarantee future results.
Investment decisions are your own responsibility.
```

---

## 7. Key Lessons

### Data Science
1. **Diversity beats quality** (Include 2020 COVID)
2. **Normalization timing matters** (At numeric stage!)
3. **Separate learning from presentation** (Grayscale train, color display)

### Product Design
1. **UX > slight accuracy gain** (Require cropping)
2. **Differentiation = fresh combination** (Search+stats+cross)
3. **Think from beginner perspective** (TOP 3, intuitive expression)

### Development Strategy
1. **Quick validation > perfect pursuit**
2. **Maximize existing tools** (ResNet18 transfer learning)
3. **Automate what's automatable** (Industry tags via API)

---

**Current Status**: ✅ Phase 1 Complete (Data Preprocessing)
**Next Goal**: 🔄 Phase 2 In Progress (AI Model Training)

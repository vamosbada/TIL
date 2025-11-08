# Patron - Stock Chart Pattern Similarity Search System

## 🎯 Project Background

**Problem**: Beginner investors struggle to learn chart patterns without complex technical indicators

**Solution**: An AI-powered educational tool that finds similar historical patterns to user-uploaded stock chart images and shows subsequent price movements

**Core Value**: "Learn from similar patterns in the past"

**Educational Purpose**:
- ✅ **Information Provision**: "Similar patterns historically moved +5% on average"
- ❌ **Investment Recommendation**: "Buy this stock"
- Clear distinction for financial regulation compliance

---

## 🛠 Tech Stack

### Data Collection & Preprocessing
- **Python**: pandas, numpy
- **Data Source**: yfinance API
- **Image Processing**: Pillow, OpenCV
- **Normalization**: scikit-learn (MinMaxScaler)
- **Visualization**: matplotlib, mplfinance

### AI/ML
- **PyTorch**: ResNet18 (ImageNet transfer learning)
- **Triplet Loss**: Semi-hard Negative Mining
- **Faiss**: Vector similarity search (L2 Distance)

### Backend/Frontend (Next Phase)
- Django 5.1 + PostgreSQL
- React 19 + TypeScript

---

## 💻 My Role

**Full-stack ownership** (end-to-end development)
- Project planning and architecture design
- Data collection and preprocessing (completed)
- AI model development (in progress)
- Backend/frontend development (planned)

---

## 🏗 System Overview

### Overall Flow

```
User uploads chart
    ↓
Grayscale conversion + normalization
    ↓
ResNet18 embedding extraction (512-dim)
    ↓
Faiss similarity search
    ↓
Return TOP 3 similar patterns
    ↓
Display subsequent price movements
```

### Why CNN + Image Approach?

**CNN (Convolutional Neural Network)**:
- Recognizes charts as "images"
- Learns patterns visually, like humans viewing charts
- RNN/LSTM only see number sequences, but CNN recognizes "shapes"

**Differentiation**:
- Existing: "This is Head & Shoulders" (pattern classification)
- Patron: "95% similar to NVIDIA 2023-05, which rose +45% after 3 months" (search + concrete examples)

---

## 🔧 Key Implementation

### 1. Data Collection (yfinance API)

**Stock Selection**:
```python
NASDAQ 100 + S&P 100 → Deduplicate → 172 stocks
```

**Collection Settings**:
- **Period**: January 2020 ~ October 29, 2025
- **Interval**: Weekly (1wk)
- **Adjustment**: auto_adjust=True (automatic stock split adjustment)
- **Data**: OHLC (Open, High, Low, Close) 4 features

**Why Weekly Data?**
- ❌ Daily: Too much noise
- ✅ Weekly: Noise averaging, reduced psychological burden with weekly checks
- ❌ Monthly: Too little data

**Why auto_adjust=True?**
```python
# False (raw): Looks like crash during stock split
Aug 28: $500
Sep 1:  $125  # CNN incorrectly learns as "crash pattern" ❌

# True (adjusted): Past data also adjusted
Aug 28: $125  # Past divided by 4
Sep 1:  $125  # Naturally connected ✅
```

**Result**: 172 stocks × 305 weeks = 52,360 data points

---

### 2. Sliding Window Pattern Generation

**Window Size**: 12 weeks (approximately 3 months)

**Why 12 Weeks?**
- One quarter = familiar time unit for investors
- Not too short, not too long - appropriate period

**Sliding Method**: Overlapping with 1-week stride

```python
# Overlapping method (adopted)
for i in range(num_patterns):
    window = ohlc[i:i+12]  # Move 1 week at a time
# Result: 305 weeks → 294 patterns (305 - 12 + 1)
```

**Why Overlapping Sliding?**
- 12x more training data (294 vs 25 patterns)
- Learns pattern continuity
- Improved matching rate with user-uploaded charts

**Final Pattern Count**: 172 stocks × 294 patterns = **49,815 patterns**

---

### 3. Data Normalization

**Normalization Strategy**: Relative to first Close + MinMaxScaler hybrid

**Why Needed?**
```python
# Before normalization
Tesla $100→$200 (100% rise) → Different chart scale
Apple $1000→$1100 (10% rise) → Different chart scale

# After normalization
Both recognized as "upward pattern" identically
```

**Implementation**:
```python
# Step 1: Relative to first Close
base_price = window[0, 3]  # First week's close (C in OHLC)
relative = window / base_price

# Step 2: MinMaxScaler (0~1 range)
normalized = MinMaxScaler().fit_transform(relative)
```

**Core**: Learning "relative change rate" instead of absolute prices

---

### 4. Grayscale Chart Image Generation

**Conversion Process**:
```
(12 weeks, 4 cols) OHLC numeric array → (224, 224, 1) grayscale image
```

**Image Format**:
- Size: 224×224 (ResNet18 input size)
- Channel: Grayscale (1 channel)
- Normalization: 0~1 range

**Why Grayscale?**

Problem: User-uploaded charts have varying colors
```
Chart A: Red/green candles
Chart B: Black/white candles
Chart C: Blue/yellow candles
```

Solution: Grayscale conversion learns "pattern shapes" regardless of color

---

### 5. Large-scale Image Generation (49,815 images)

**Goal**: Convert 49,815 patterns to 224×224 grayscale images

**Why mplfinance?**
- ✅ Optimized for financial charts (candlestick dedicated)
- ✅ Superior wick rendering
- ✅ Concise code

**Contrast Enhancement Added**:
```python
from PIL import ImageEnhance

# After mplfinance chart generation
mpf.plot(data, type='candle', savefig={'dpi': 150})

# Contrast enhancement (1.5x)
enhancer = ImageEnhance.Contrast(img_resized)
img_enhanced = enhancer.enhance(1.5)
```

**Effect**: Clearer candle patterns improve CNN learning efficiency

**Long-running Task Handling**:
- Checkpoint system: Save progress every 100 patterns
- Resume from interruptions
- Real-time progress monitoring with tqdm

---

### 6. AI Model Design

**ResNet18 + Transfer Learning**:
- Uses ImageNet pre-trained model
- From scratch (100+ hours) vs transfer learning (5-6 hours)
- Extracts 512-dimensional embeddings

**Triplet Loss + Semi-hard Negative Mining**:
```python
Loss = max(0, d(anchor, positive) - d(anchor, negative) + margin)

# Goal: Similar patterns closer, different patterns farther
```

**Self-supervised Learning**:
- No manual labeling required (49,815 × 3 = 149,445 combinations)
- Automatically generates Anchor-Positive-Negative
- Learns "relative similarity"

**Faiss L2 Distance**:
- Euclidean distance measurement between 512-dim embedding vectors
- Smaller distance = more similar patterns
- Real-time search across 49,815 patterns

---

### 7. Cross-stock Matching Strategy

**Full Search (Adopted)**:
```
TSLA upload → Search across NVDA, AMD, NFLX, F...
→ "Tesla is similar to Netflix 2 years ago!"
```

**Selection Rationale**:
- Unexpected connections = true service value
- Understanding cross-industry cycles
- Maximized learning effect for beginners

> For detailed technical decisions, see [DECISIONS.md](./DECISIONS.md)

---

## 📊 Data Statistics

| Item | Value |
|------|-------|
| **Total Stocks** | 172 (NASDAQ 100 + S&P 100) |
| **Data Period** | January 2020 ~ October 29, 2025 |
| **Total Weeks** | 305 weeks |
| **Generated Patterns** | 49,815 patterns |
| **Image Size** | 224×224 grayscale |
| **Embedding Dimension** | 512-dim (ResNet18) |

---

## 💡 Lessons Learned

### Technical Learnings

**1. Importance of Data Preprocessing**
- Normalization timing: At numeric stage first!
- Normalization after image generation is too late
- Learn relative change rates, not absolute prices

**2. Power of Transfer Learning**
- ImageNet pre-training → Reduced to 5-6 hours
- From-scratch training takes 100+ hours
- 49,815 patterns provide sufficient performance

**3. Self-supervised Learning**
- Triplet Loss = Pull (similar) + Push (different)
- No manual labeling required
- Semi-hard Negative provides best learning efficiency

**4. Practical Technology Choices**
- Weekly data: Noise reduction + sufficient data
- Overlapping sliding window: 12x more training data
- mplfinance: Dedicated financial chart library
- Grayscale: Pattern learning regardless of color
- Contrast enhancement: Clearer candle patterns

### Design Learnings

**1. Beginner-centric Design**
- Weekly data = Reduced psychological burden
- Full search = Discovery fun
- Simplicity > Complex options

**2. Differentiation Strategy**
- Search, not classification
- Cross-stock/industry search
- Concrete historical examples

**3. Legal Safety**
- Clear distinction: "Information provision" vs "Investment advice"
- Historical patterns ≠ Future predictions

**4. Large-scale Task Design**
- Checkpoint system essential
- Progress monitoring
- Resume from interruptions

---

## 🔗 Next Steps

### Phase 2: AI Model Training (In Progress)
- [ ] ResNet18 backbone construction
- [ ] Triplet Loss implementation
- [ ] Semi-hard Negative Mining
- [ ] Model training and validation

### Phase 3: Similarity Search (Planned)
- [ ] Faiss index construction
- [ ] Django API development
- [ ] Search result optimization

### Phase 4: Frontend (Planned)
- [ ] React UI development
- [ ] Chart visualization
- [ ] Integration testing

---

## ❓ Interview Prep Questions

**Q1: Why choose CNN?**
> A: RNN/LSTM only see number sequences, but CNN recognizes chart "shapes" as images. Just like humans view charts visually, CNN also learns visual patterns. Since input is images, CNN is optimal.

**Q2: Why search across all stocks?**
> A: Discovering unexpected connections like "Tesla is similar to Netflix 2 years ago" is the true service value. Helps understand cross-industry cycles and maximizes learning effect for beginners.

**Q3: Why choose weekly data?**
> A: Daily data has too much noise for beginners, and monthly data has too few patterns. Weekly data secures 49,815 sufficient patterns while reducing psychological burden with weekly checks.

**Q4: Why normalize each pattern independently?**
> A: To learn relative pattern shapes (e.g., uptrend, downtrend) rather than absolute prices (e.g., AAPL $150 vs TSLA $200). Enables pattern similarity comparison across different-priced stocks.

**Q5: Why use OHLC 4 features?**
> A: Using only close prices shows only results, but using OHLC 4 features reflects open, high, low, and close in candlestick charts, capturing volatility. Chosen for more accurate pattern recognition.

**Q6: Why choose Semi-hard Negative Mining?**
> A: Easy Negative has low learning effect, and Hard Negative causes training instability. Semi-hard Negative maximizes learning efficiency with "moderately confusing" samples. Batch size 32 is sufficient with 49,815 patterns.

**Q7: Are 49,815 patterns sufficient?**
> A: Triplet Loss creates multiple combinations per batch, so actual training samples are much more. Also, starting with ImageNet transfer learning provides sufficient data volume.

---

## 📂 Project Structure

```
Patron/
├── data/
│   ├── raw/                 # yfinance raw data
│   ├── processed/           # Preprocessed patterns (49,815)
│   ├── metadata.csv         # Metadata
│   └── images/              # Grayscale chart images (224×224)
├── notebooks/
│   └── preprocessing.ipynb  # Preprocessing code
└── README.md                # Project description
```

---

**Current Status**: ✅ Phase 1 Complete (Data Preprocessing)
**Next Goal**: 🔄 Phase 2 In Progress (AI Model Training)
**Owner**: Bada (Patron lead in QuantrumAI team)

# Patron - Stock Chart Pattern Similarity Search System (Data Preprocessing)

## 🎯 Project Background

**Problem**: Beginner investors struggle to learn chart patterns without complex technical indicators

**Solution**: An AI-powered educational tool that finds similar historical patterns to user-uploaded stock chart images and shows subsequent price movements

**Core Value**: "Learn from similar patterns in the past"

---

## 🛠 Tech Stack

### Data Collection & Preprocessing
- **Python**: pandas, numpy
- **Data Source**: yfinance API
- **Image Processing**: Pillow, OpenCV
- **Normalization**: scikit-learn (MinMaxScaler)
- **Visualization**: matplotlib, mplfinance

### AI/ML (Next Phase)
- PyTorch (ResNet18)
- Triplet Loss (Semi-hard Negative Mining)
- Faiss (similarity search)

### Backend/Frontend (Next Phase)
- Django 5.1 + PostgreSQL
- React 19 + TypeScript

---

## 💻 My Role

**Full-stack ownership** (end-to-end development)
- Project planning and architecture design
- Data collection and preprocessing (completed)
- AI model development (planned)
- Backend/frontend development (planned)

---

## 🔧 Key Implementation

### 1. Data Collection (yfinance API)

**Stock Selection**:
```python
NASDAQ 100 + S&P 100 → deduplication → 172 stocks
```

**Collection Settings**:
- Period: 2020-01-01 ~ 2025-10-29 (approximately 6 years)
- Interval: weekly (1wk)
- Adjustment: auto_adjust=True (automatic stock split adjustment)

**Result**: 172 stocks × 305 weeks = 52,360 data points

### 2. Sliding Window Pattern Generation

**Window Size**: 12 weeks

**Sliding Strategy**:
```
305 weeks → 12-week sliding windows
= (305 - 12 + 1) = 294 patterns per stock
```

**Final Pattern Count**: 172 stocks × 294 patterns = **49,815 patterns**

### 3. Data Normalization

**Normalization Strategy**: Apply MinMaxScaler independently to each 12-week pattern

```python
# Normalize each pattern to 0~1 range
scaler = MinMaxScaler()
normalized_pattern = scaler.fit_transform(pattern)
```

**Rationale**: Learn relative pattern shapes rather than absolute prices

### 4. Metadata Generation

**Metadata Columns**:
```csv
ticker, pattern_id, start_date, end_date, start_price, end_price,
sector, industry, return_3m, return_6m, return_1y
```

**Future Return Calculation**:
- 3-month return (return_3m)
- 6-month return (return_6m)
- 12-month return (return_1y)

### 5. Grayscale Chart Image Generation

**Conversion Process**:
```
(12 weeks, 4 columns) numeric array → (224, 224, 1) grayscale image
```

**Image Format**:
- Size: 224×224 (ResNet18 input size)
- Channels: grayscale (1 channel)
- Normalization: 0~1 range

---

## 📊 Data Statistics

| Metric | Value |
|--------|-------|
| **Total Stocks** | 172 |
| **Data Period** | 2020-01-01 ~ 2025-10-29 (6 years) |
| **Total Weeks** | 305 |
| **Generated Patterns** | 49,815 |
| **Data Size** | ~6 MB (CSV + metadata) |

**Data Characteristics**:
- ✅ Includes 2020 COVID-19 volatility (diverse patterns)
- ✅ Stock split adjustments completed
- ⚠️ Weekly data only (no daily/minute data)
- ⚠️ US stocks only (no Korean stocks)

---

## 💡 Key Takeaways

### 1. Working with Financial Data
- Using yfinance API effectively
- Understanding stock data characteristics (OHLC, stock splits)
- Sliding window technique for time-series data

### 2. Data Preprocessing Strategies
- Importance of pattern-wise independent normalization
- Metadata design (future return calculation)
- Efficient processing of large-scale data

### 3. AI Model Design
- Understanding ResNet18 input format (224×224)
- Grayscale vs RGB trade-offs
- Data structure design for Triplet Loss

---

## 🔗 Next Steps

### Phase 2: AI Model Training (Planned)
- [ ] Build ResNet18 backbone (ImageNet transfer learning)
- [ ] Implement Triplet Loss + Semi-hard Negative Mining
- [ ] Construct training dataset (Anchor-Positive-Negative)
- [ ] Model training and validation

### Phase 3: Similarity Search (Planned)
- [ ] Build Faiss index (L2 Distance)
- [ ] Implement temporal diversity filtering algorithm
- [ ] Develop Django API endpoints

### Phase 4: Frontend (Planned)
- [ ] Develop React UI (5×3 matrix output)
- [ ] Chart visualization (Recharts)
- [ ] Integration testing

---

## ❓ Interview Prep Questions

**Q1: Why did you choose weekly data?**
> A: Daily data has excessive volume leading to high training costs, while monthly data lacks pattern diversity. Weekly data provides sufficient patterns (49,815) from 6 years of data while maintaining reasonable computational costs.

**Q2: Why normalize each pattern independently?**
> A: To learn relative pattern shapes (e.g., uptrend, downtrend) rather than absolute prices (e.g., AAPL $150 vs TSLA $200). This enables pattern similarity comparison across stocks with different price levels.

**Q3: Why Semi-hard Negative Mining?**
> A: Easy Negatives provide minimal learning signal, while Hard Negatives cause training instability. Semi-hard Negatives offer "moderately confusing" samples that maximize learning efficiency.

**Q4: Are 49,815 patterns sufficient?**
> A: Triplet Loss creates multiple combinations within each batch, effectively multiplying the training samples. Combined with ImageNet transfer learning, this dataset is sufficient for effective training.

**Q5: What's the most critical task in the next phase?**
> A: Implementing Semi-hard Negative Mining for Triplet Loss. This is crucial for the model to learn embeddings where similar patterns are close together and dissimilar patterns are far apart.

---

## 📂 Project Structure
```
Patron/
├── data/
│   ├── raw/                 # yfinance raw data
│   ├── processed/           # preprocessed patterns (49,815)
│   ├── metadata.csv         # metadata
│   └── images/              # grayscale chart images
├── notebooks/
│   └── preprocessing.ipynb  # preprocessing code
└── README.md                # project documentation
```

---

**Current Status**: ✅ Phase 1 Complete (Data Preprocessing)
**Next Goal**: 🔄 Phase 2 In Progress (AI Model Training)

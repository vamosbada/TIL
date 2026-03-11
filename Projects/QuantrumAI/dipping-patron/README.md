# Patron - Stock Chart Pattern Similarity Search System

## 🎯 Project Background

**Problem**: Beginner investors struggle to learn chart patterns without complex technical indicators

**Solution**: An AI-powered educational tool that finds the 3 most visually similar historical patterns to a user-queried stock ticker and shows subsequent price movements

**Core Value**: "Learn from similar patterns in the past"

**Educational Purpose**:
- ✅ **Information Provision**: "Similar patterns historically moved +5% on average"
- ❌ **Investment Recommendation**: "Buy this stock"
- Clear distinction for financial regulation compliance

**GitHub**: [vamosbada/patron](https://github.com/vamosbada/patron)

---

## 🛠 Tech Stack

### Data Collection & Preprocessing
- **Python**: pandas, numpy
- **Data Source**: yfinance API
- **Image Processing**: Pillow (contrast enhancement)
- **Normalization**: scikit-learn (MinMaxScaler, per-pattern)
- **Visualization**: mplfinance

### AI/ML
- **PyTorch**: ResNet18 (ImageNet transfer learning, grayscale input)
- **Triplet Loss**: Semi-hard Negative Mining
- **Faiss**: IndexFlatL2 vector similarity search (L2 distance)

### Backend (FastAPI)
- FastAPI + Uvicorn
- Real-time yfinance data fetching
- Pre-computed embeddings (50k vectors, loaded at startup)

---

## 💻 My Role

**Full-stack ownership** (end-to-end, solo development)
- Project planning and architecture design
- Data collection and preprocessing
- AI model training (ResNet18 + Triplet Loss)
- FAISS index construction and Top-3 search algorithm
- FastAPI server development and deployment preparation

---

## 🏗 System Overview

### Overall Flow

```
POST /api/patron/search (ticker input)
    ↓
yfinance: fetch latest 12-week OHLC
    ↓
MinMaxScaler normalization → 224×224 grayscale image
    ↓
ResNet18 → 512-dim L2-normalized embedding
    ↓
FAISS IndexFlatL2 search over ~50k historical patterns
    ↓
Top-3 (ticker-deduped, self-excluded) + forward returns
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
- **Period**: January 2020 ~ October 2025
- **Interval**: Weekly (1wk)
- **Adjustment**: auto_adjust=True (automatic stock split adjustment)
- **Data**: OHLC (Open, High, Low, Close) 4 features

**Why Weekly Data?**
- ❌ Daily: Too much noise
- ✅ Weekly: Noise averaging, reduced psychological burden with weekly checks
- ❌ Monthly: Too little data

**Result**: ~49,987 patterns (12-week sliding window, 1-week stride)

---

### 2. Sliding Window Pattern Generation

**Window Size**: 12 weeks (approximately 3 months)

**Why 12 Weeks?**
- One quarter = familiar time unit for investors
- Not too short, not too long - appropriate period

**Sliding Method**: Overlapping with 1-week stride

```python
for i in range(num_patterns):
    window = ohlc[i:i+12]  # Move 1 week at a time
# Result: 305 weeks → 294 patterns per stock
```

---

### 3. Data Normalization (MinMaxScaler, per-pattern)

**Why MinMaxScaler per-pattern?**
```python
# After normalization: both mapped to [0, 1]
Tesla $100→$200 (100% rise) → same visual shape as
Apple $1000→$2000 (100% rise)
```

**Implementation**:
```python
# OHLC 4 columns together → MinMaxScaler → [0, 1]
scaler = MinMaxScaler()
normalized = scaler.fit_transform(window)  # per 12-week pattern
```

**Why NOT Log normalization?** (Experiment 3)
```
Log: linear [100,110,120,130,140,150] → curved [0.0, 0.095, 0.182, ...]
MinMaxScaler: linear → linear [0.0, 0.2, 0.4, 0.6, 0.8, 1.0] ✅
```
Log normalization distorts chart shapes visually — numerically closer but visually wrong patterns returned. MinMaxScaler preserves the original shape of the chart.

---

### 4. Grayscale Chart Image Generation

**Conversion Process**:
```
(12 weeks, 4 cols) OHLC array → mplfinance candlestick → (224, 224, 1) grayscale
```

**Why Grayscale?**: Pattern shape matters, not color. Removes dependency on charting style.

**Contrast Enhancement**:
```python
enhancer = ImageEnhance.Contrast(img)
img_enhanced = enhancer.enhance(1.5)  # 1.5x contrast boost
```

---

### 5. AI Model: ResNet18 + Triplet Loss

**Architecture**:
- ImageNet pretrained ResNet18
- conv1: 3-channel → 1-channel (grayscale input)
- FC layer removed → 512-dim L2-normalized embedding output

**Triplet Loss + Semi-hard Negative Mining**:
```python
Loss = max(d(anchor, positive) - d(anchor, negative) + margin, 0)
# margin = 0.2
```

**Anchor-Positive pairs**: Same ticker, t-week and t+1, t+2, t+3 week patterns

**Training Results**:

| Epoch | Train Loss | Val Loss | Note |
|-------|-----------|---------|------|
| 1 | 0.007197 | 0.005817 | Random Negative |
| 2 | 0.004771 | **0.005174** | Random Negative — Best |
| 3 | 0.004488 | 0.005999 | Semi-hard → Early Stop |

- Total training time: 1.5h (NVIDIA A100, Mixed Precision)
- Full pipeline including image generation: 6.5h

---

### 6. FAISS Search + Top-3 Algorithm

**Index**: `faiss.IndexFlatL2(512)` — exact L2 search over ~50k vectors

**Top-3 Algorithm**:
```python
def get_top3(query_embedding, all_embeddings, metadata, query_ticker, query_date):
    # 1. Sort by L2 distance → Top-100 candidates
    distances = np.linalg.norm(all_embeddings - query_embedding, axis=1)
    top_k_indices = np.argsort(distances)[:100]

    # 2. Filter: exclude self (same ticker + within 14 days)
    # 3. Ticker deduplication: only 1 pattern per ticker
    selected = []
    seen_tickers = set()
    for idx in top_k_indices:
        ticker = metadata.loc[idx, 'ticker']
        if ticker == query_ticker and date_diff < 14:
            continue
        if ticker in seen_tickers:
            continue
        selected.append(idx)
        seen_tickers.add(ticker)
        if len(selected) == 3:
            break
    return selected
```

**Why ticker deduplication?**: Without it, the same ticker's consecutive patterns dominate Top-3 (avg 2.2 duplicates per search).

---

## 📊 Data Statistics

| Item | Value |
|------|-------|
| **Total Stocks** | 172 (NASDAQ 100 + S&P 100) |
| **Data Period** | January 2020 ~ October 2025 |
| **Generated Patterns** | ~49,987 |
| **Image Size** | 224×224 grayscale |
| **Embedding Dimension** | 512-dim (ResNet18) |
| **Training Time** | 1.5h (A100) + 5h image generation |

---

## 🧪 Experiment Log

### Experiment 1 — MinMaxScaler (Final Model)
- Val Loss: **0.005174** (Epoch 2)
- Avg duplicate tickers in Top-3: 2.20 → fixed by ticker dedup
- Visual quality: ✅ Visually similar patterns

### Experiment 2 — Normalization Analysis
Compared 4 normalization methods. Log normalization theoretically better for finance, but:

### Experiment 3 — Log Normalization (Rejected)
- Val Loss: 0.138237 — **27x worse** than Exp 1
- Avg duplicate tickers: 0.15 (better numerically)
- Visual quality: ❌ Numerically close but visually different patterns

**Key lesson**: Metric improvement ≠ Real improvement. Always visually verify.

---

## 💡 Lessons Learned

**1. Theory vs Practice Gap**
- Log normalization is theoretically standard in finance, but distorts linear trends into curves
- MinMaxScaler preserves visual chart shape → models match human visual perception

**2. Data-centric Insight**
- 49,987 patterns × Triplet Loss combinations = sufficient training signal
- ImageNet transfer learning dramatically reduces training time (6.5h vs 100h+)

**3. Algorithm Design**
- Ticker deduplication is essential for useful search results
- Pre-computed embeddings at startup: API response in <1s (vs 3min if computed per request)

**4. Large-scale Task Design**
- Checkpoint system every 100 patterns — resume from interruptions
- Mixed precision training on A100 reduces memory usage

---

## ❓ Interview Prep Questions

**Q1: Why choose CNN over RNN/LSTM?**
> A: RNN/LSTM only see number sequences. CNN recognizes chart "shapes" as images, just like humans view charts visually. Since the input is a grayscale image, CNN is the natural choice.

**Q2: Why MinMaxScaler over Log normalization?**
> A: Log normalization distorts linear trends into curves, causing the model to learn shapes that don't match human visual perception. We tested both (Experiments 1 and 3) and MinMaxScaler produced visually similar patterns despite higher numerical loss. Theory and practice diverged here.

**Q3: Why ticker deduplication in Top-3?**
> A: Without it, the same ticker's consecutive patterns dominate results — average 2.2 duplicates per search. Deduplication ensures diverse, informative Top-3 results.

**Q4: Why pre-compute embeddings?**
> A: Computing 50k embeddings per request takes ~3 minutes. Pre-loading at server startup enables <1s response time.

**Q5: Why Semi-hard Negative Mining?**
> A: Easy Negatives contribute near-zero gradient. Hard Negatives cause training instability early on. Semi-hard Negatives — within the margin but farther than the positive — maximize learning efficiency.

**Q6: Why weekly data?**
> A: Daily data has too much noise for beginners. Monthly data produces too few patterns. Weekly gives 49,987 patterns while reducing psychological burden.

---

## 📂 Project Structure

```
patron/
├── notebooks/
│   ├── 01_preprocessing.ipynb        # Data collection & preprocessing
│   ├── 02_training_v1.ipynb          # ResNet18 + Triplet Loss (final model)
│   ├── 03_faiss_search.ipynb         # FAISS index + Top-3 search
│   ├── 04_normalization_compare.ipynb # MinMaxScaler vs Log analysis
│   ├── 05_preprocessing_v2.ipynb     # Log normalization preprocessing
│   ├── 06_training_v2.ipynb          # Log normalization training (rejected)
│   ├── 07_visual_comparison.ipynb    # Visual comparison Exp 1 vs Exp 3
│   ├── 08_realtime_search.ipynb      # Live demo with yfinance
│   └── 09_embedding_precompute.ipynb # Pre-compute 50k embeddings
├── patron_fastapi/
│   ├── main.py                       # FastAPI server
│   ├── requirements.txt
│   ├── SERVER.md
│   └── data/
│       ├── metadata_all.csv          # Pattern metadata (49,987 rows)
│       └── raw/                      # 172 ticker CSVs
├── ARCHITECTURE.md                   # Full design doc & experiment log
└── README.md
```

---

**Current Status**: ✅ Complete — Data preprocessing + Model training + FAISS indexing + FastAPI server
**GitHub**: [vamosbada/patron](https://github.com/vamosbada/patron)
**Owner**: Bada Shin (QuantrumAI ML Engineer)

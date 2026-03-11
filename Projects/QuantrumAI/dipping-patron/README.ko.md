# Patron - 차트 패턴 유사도 검색 시스템

## 🎯 프로젝트 배경

**문제점**: 주식 초보자가 복잡한 기술적 지표 없이 차트 패턴을 학습하기 어려움

**해결책**: 종목 코드를 입력하면 최근 12주 차트와 가장 유사한 과거 패턴 Top-3를 찾아 이후 주가 흐름을 보여주는 AI 기반 교육 도구

**핵심 가치**: "과거의 비슷한 패턴을 보고 배우자"

**교육 목적**:
- ✅ **정보 제공**: "과거 유사 패턴에서 평균 +5% 움직였습니다"
- ❌ **투자 권유**: "이 종목을 사세요"
- 금융법 준수를 위한 명확한 구분

**GitHub**: [vamosbada/patron](https://github.com/vamosbada/patron)

---

## 🛠 기술 스택

### 데이터 수집 및 전처리
- **Python**: pandas, numpy
- **데이터 소스**: yfinance API
- **이미지 처리**: Pillow (대비 증가)
- **정규화**: scikit-learn (MinMaxScaler, 패턴별 독립 적용)
- **시각화**: mplfinance

### AI/ML
- **PyTorch**: ResNet18 (ImageNet 전이학습, 그레이스케일 입력)
- **Triplet Loss**: Semi-hard Negative Mining
- **Faiss**: IndexFlatL2 벡터 유사도 검색 (L2 거리)

### 백엔드 (FastAPI)
- FastAPI + Uvicorn
- yfinance 실시간 데이터 조회
- 사전 계산된 임베딩 (50k 벡터, 서버 시작 시 로드)

---

## 💻 내 역할

**전체 프로젝트 전담** (1부터 100까지, 단독 개발)
- 프로젝트 기획 및 아키텍처 설계
- 데이터 수집 및 전처리
- AI 모델 학습 (ResNet18 + Triplet Loss)
- FAISS 인덱스 구축 및 Top-3 검색 알고리즘
- FastAPI 서버 개발 및 배포 준비

---

## 🏗 시스템 개요

### 전체 플로우

```
POST /api/patron/search (종목 코드 입력)
    ↓
yfinance: 최근 12주 OHLC 실시간 조회
    ↓
MinMaxScaler 정규화 → 224×224 그레이스케일 이미지
    ↓
ResNet18 → 512차원 L2 정규화 임베딩
    ↓
FAISS IndexFlatL2로 ~50k 과거 패턴 검색
    ↓
Top-3 반환 (종목 중복 제거, 본인 제외) + 3·6·12개월 수익률
```

### 왜 CNN + 이미지 방식인가?

**CNN (Convolutional Neural Network)**:
- 차트를 "이미지"로 인식
- 사람이 차트를 시각적으로 보듯이 패턴 학습
- RNN/LSTM은 숫자 시퀀스만 보지만, CNN은 "모양" 인식

**차별점**:
- 기존: "이건 Head & Shoulders입니다" (패턴 분류)
- Patron: "NVIDIA 2023-05와 95% 유사, 그때는 3개월 후 +45% 상승" (검색 + 구체적 사례)

---

## 🔧 핵심 구현 내용

### 1. 데이터 수집 (yfinance API)

**종목 선정**:
```python
NASDAQ 100 + S&P 100 → 중복 제거 → 172개 종목
```

**수집 설정**:
- **기간**: 2020년 01월 ~ 2025년 10월
- **간격**: 주봉 (1wk)
- **보정**: auto_adjust=True (액면분할 자동 보정)
- **데이터**: OHLC (Open, High, Low, Close) 4개 특성

**주봉 선택 이유**:
- ❌ 일봉: 노이즈 많음
- ✅ 주봉: 노이즈 평균화, 주단위 체크로 심리적 부담 감소
- ❌ 월봉: 데이터 너무 적음

**결과**: 약 49,987개 패턴 (12주 슬라이딩 윈도우, 1주 stride)

---

### 2. 슬라이딩 윈도우 패턴 생성

**윈도우 크기**: 12주 (약 3개월)

**12주 선택 이유**:
- 한 분기 = 투자자들이 익숙한 시간 단위
- 너무 짧지도, 길지도 않은 적절한 기간

**슬라이딩 방식**: 1주씩 이동하는 겹치는 방식

```python
for i in range(num_patterns):
    window = ohlc[i:i+12]  # 1주씩 이동
# 결과: 305주 → 종목당 294개 패턴
```

---

### 3. 데이터 정규화 (MinMaxScaler, 패턴별)

**왜 패턴별 MinMaxScaler인가?**
```python
# 정규화 후: 둘 다 [0, 1]로 매핑
테슬라 $100→$200 (100% 상승) → 애플 $1000→$2000 (100% 상승)과 동일한 시각적 형태
```

**구현**:
```python
# OHLC 4열 동시 → MinMaxScaler → [0, 1]
scaler = MinMaxScaler()
normalized = scaler.fit_transform(window)  # 12주 패턴별 독립 적용
```

**왜 Log 정규화 안 쓰나?** (실험 3에서 기각)
```
Log: 직선 상승 [100,110,120,130,140,150] → 곡선 [0.0, 0.095, 0.182, ...]
MinMaxScaler: 직선 → 직선 [0.0, 0.2, 0.4, 0.6, 0.8, 1.0] ✅
```
Log 정규화는 차트 모양을 왜곡함 — 숫자상으론 가깝지만 시각적으로 다른 패턴을 검색하는 문제 발생. MinMaxScaler가 원래 차트 형태를 보존.

---

### 4. 그레이스케일 차트 이미지 생성

**변환 과정**:
```
(12주, 4열) OHLC 배열 → mplfinance 캔들차트 → (224, 224, 1) 그레이스케일
```

**왜 그레이스케일?**: 색상이 아닌 패턴 모양이 중요. 차트 스타일 의존성 제거.

**대비 증가**:
```python
enhancer = ImageEnhance.Contrast(img)
img_enhanced = enhancer.enhance(1.5)  # 1.5배 대비 강화
```

---

### 5. AI 모델: ResNet18 + Triplet Loss

**아키텍처**:
- ImageNet 사전학습 ResNet18
- conv1: 3채널 → 1채널 (그레이스케일 입력)
- FC 레이어 제거 → 512차원 L2 정규화 임베딩 출력

**Triplet Loss + Semi-hard Negative Mining**:
```python
Loss = max(d(anchor, positive) - d(anchor, negative) + margin, 0)
# margin = 0.2
```

**Anchor-Positive 쌍**: 같은 종목의 t주차 패턴과 t+1, t+2, t+3주차 패턴

**학습 결과**:

| Epoch | Train Loss | Val Loss | 비고 |
|-------|-----------|---------|------|
| 1 | 0.007197 | 0.005817 | Random Negative |
| 2 | 0.004771 | **0.005174** | Random Negative — Best |
| 3 | 0.004488 | 0.005999 | Semi-hard → Early Stop |

- 학습 시간: 1.5시간 (NVIDIA A100, Mixed Precision)
- 이미지 생성 포함 전체: 6.5시간

---

### 6. FAISS 검색 + Top-3 알고리즘

**인덱스**: `faiss.IndexFlatL2(512)` — ~50k 벡터 정확 L2 검색

**Top-3 알고리즘**:
```python
def get_top3(query_embedding, all_embeddings, metadata, query_ticker, query_date):
    # 1. L2 거리 정렬 → Top-100 후보
    distances = np.linalg.norm(all_embeddings - query_embedding, axis=1)
    top_k_indices = np.argsort(distances)[:100]

    # 2. 본인 제외 (같은 ticker + 14일 이내)
    # 3. 종목 중복 제거: 종목당 가장 유사한 1개만
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

**왜 종목 중복 제거?**: 없으면 같은 종목의 연속된 패턴이 Top-3 독점 — 평균 2.2개 중복.

---

## 📊 데이터 통계

| 항목 | 값 |
|------|-----|
| **총 종목 수** | 172개 (NASDAQ 100 + S&P 100) |
| **데이터 기간** | 2020년 01월 ~ 2025년 10월 |
| **생성된 패턴 수** | 약 49,987개 |
| **이미지 크기** | 224×224 그레이스케일 |
| **임베딩 차원** | 512차원 (ResNet18) |
| **학습 시간** | 1.5시간 (A100) + 5시간 이미지 생성 |

---

## 🧪 실험 로그

### 실험 1 — MinMaxScaler (최종 채택)
- Val Loss: **0.005174** (Epoch 2)
- Top-3 평균 중복 종목: 2.20개 → 종목 중복 제거로 해결
- 시각적 품질: ✅ 눈으로 봐도 유사한 패턴

### 실험 2 — 정규화 방법 비교
4가지 정규화 방법 비교 분석. Log 정규화가 금융 이론상 더 우수하지만:

### 실험 3 — Log 정규화 (기각)
- Val Loss: 0.138237 — **실험 1 대비 27배 높음**
- 평균 중복 종목: 0.15개 (수치상 더 좋음)
- 시각적 품질: ❌ 숫자는 가깝지만 시각적으로 다른 패턴 반환

**핵심 교훈**: 지표 개선 ≠ 실제 개선. 항상 시각적으로 검증해야 함.

---

## 💡 배운 점

**1. 이론과 실제의 괴리**
- Log 정규화는 금융 이론상 표준이지만 직선 추세를 곡선으로 왜곡
- MinMaxScaler가 시각적 차트 형태를 보존 → 모델이 사람의 시각 인식과 일치

**2. 데이터 중심적 사고**
- 49,987개 패턴 × Triplet Loss 조합 = 충분한 학습 신호
- ImageNet 전이학습으로 학습 시간 대폭 단축 (6.5시간 vs 100시간+)

**3. 알고리즘 설계**
- 종목 중복 제거는 유용한 검색 결과를 위해 필수
- 임베딩 사전 계산: API 응답 <1초 (요청마다 계산 시 3분 소요)

**4. 대규모 작업 설계**
- 100개마다 체크포인트 저장 — 중단 후 재개 가능
- A100 Mixed Precision으로 메모리 사용량 절감

---

## ❓ 면접 예상 질문

**Q1: RNN/LSTM 대신 CNN을 선택한 이유는?**
> A: RNN/LSTM은 숫자 시퀀스만 보지만, CNN은 차트의 "모양"을 이미지로 인식합니다. 사람이 차트를 시각적으로 보듯이 CNN도 시각적 패턴을 학습합니다. 입력이 이미지이므로 CNN이 최적입니다.

**Q2: Log 정규화 대신 MinMaxScaler를 선택한 이유는?**
> A: Log 정규화는 직선 추세를 곡선으로 왜곡해서, 모델이 사람 눈에 유사하지 않은 패턴을 검색하는 문제가 발생했습니다. 실험 1과 3을 직접 비교한 결과, Val Loss는 Log가 27배 나빴고 시각적 품질도 떨어졌습니다. 이론과 실제가 다른 대표적 사례입니다.

**Q3: Top-3에서 종목 중복 제거를 하는 이유는?**
> A: 제거 없이는 같은 종목의 연속된 패턴이 Top-3를 독점합니다 (평균 2.2개 중복). 중복 제거로 다양하고 유용한 결과를 반환합니다.

**Q4: 임베딩을 왜 사전 계산하나요?**
> A: 50,000개 임베딩을 요청마다 계산하면 약 3분이 걸립니다. 서버 시작 시 미리 로드하면 응답 시간 <1초로 단축됩니다.

**Q5: Semi-hard Negative Mining을 선택한 이유는?**
> A: Easy Negative는 gradient가 거의 0에 가까워 학습 기여가 없고, Hard Negative는 초반 학습을 불안정하게 만듭니다. Semi-hard Negative는 margin 내에 있지만 positive보다 먼 샘플로, 학습 효율을 최대화합니다.

**Q6: 주봉 데이터를 선택한 이유는?**
> A: 일봉은 주린이에게 노이즈가 많아 부담이고, 월봉은 패턴 수가 너무 적습니다. 주봉은 49,987개의 충분한 패턴을 확보하면서도 심리적 부담을 줄입니다.

---

## 📂 프로젝트 구조

```
patron/
├── notebooks/
│   ├── 01_preprocessing.ipynb        # 데이터 수집 및 전처리
│   ├── 02_training_v1.ipynb          # ResNet18 + Triplet Loss (최종 모델)
│   ├── 03_faiss_search.ipynb         # FAISS 인덱스 + Top-3 검색
│   ├── 04_normalization_compare.ipynb # MinMaxScaler vs Log 분석
│   ├── 05_preprocessing_v2.ipynb     # Log 정규화 전처리
│   ├── 06_training_v2.ipynb          # Log 정규화 학습 (기각)
│   ├── 07_visual_comparison.ipynb    # 실험 1 vs 실험 3 시각적 비교
│   ├── 08_realtime_search.ipynb      # yfinance 실시간 데모
│   └── 09_embedding_precompute.ipynb # 50k 임베딩 사전 계산
├── patron_fastapi/
│   ├── main.py                       # FastAPI 서버
│   ├── requirements.txt
│   ├── SERVER.md
│   └── data/
│       ├── metadata_all.csv          # 패턴 메타데이터 (49,987행)
│       └── raw/                      # 172개 종목 CSV
├── ARCHITECTURE.md                   # 전체 설계 문서 및 실험 로그
└── README.md
```

---

**현재 상태**: ✅ 완료 — 데이터 전처리 + 모델 학습 + FAISS 인덱싱 + FastAPI 서버
**GitHub**: [vamosbada/patron](https://github.com/vamosbada/patron)
**담당**: 신바다 (QuantrumAI ML Engineer)

# Django REST Framework 기초

## 🎯 한 줄 요약

DRF의 핵심 개념과 각 기능의 차이점 (@api_view, APIView, Generics)

---

## 📌 핵심 내용

### 1. API의 본질: JSON 상하차 시스템

**백엔드의 핵심 = JSON 상하차**

```
프론트엔드 ←→ [JSON 📦] ←→ Serializer ←→ 데이터베이스
```

#### 직렬화 (Serialization / 상차)
```python
# DB 데이터 → JSON으로 포장
book = Book.objects.get(id=1)
serializer = BookSerializer(book)
json_data = serializer.data  # 프론트로 전송
```

#### 역직렬화 (Deserialization / 하차)
```python
# JSON → DB에 저장
data = request.data  # 프론트에서 받은 JSON
serializer = BookSerializer(data=data)
if serializer.is_valid():
    serializer.save()  # DB에 저장
```

---

### 2. Serializer vs ModelSerializer

#### A. Serializer (수동)

**특징**: 모든 것을 직접 정의

```python
class BookSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=200)
    author = serializers.CharField(max_length=100)
    published_date = serializers.DateField()

    def create(self, validated_data):
        return Book.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.title = validated_data.get('title', instance.title)
        instance.author = validated_data.get('author', instance.author)
        instance.save()
        return instance
```

**언제 쓰나**: 모델과 무관한 데이터, 복잡한 커스터마이징

#### B. ModelSerializer (자동)

**특징**: 모델 기반 자동 생성

```python
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = '__all__'  # 또는 ['title', 'author']
        # create(), update() 자동 구현됨!
```

**언제 쓰나**: 모델 기반 일반적인 CRUD (대부분의 경우)

**차이점 요약**:
- Serializer: 필드, create(), update() 모두 직접 작성
- ModelSerializer: 모델만 지정하면 자동 생성

---

### 3. @api_view (함수 기반 뷰)

**특징**: 함수에 DRF 기능 추가

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(['GET', 'POST'])
def book_list(request):
    if request.method == 'GET':
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)
```

**제공 기능**:
- `request.data`: JSON 자동 파싱
- `Response`: 자동 JSON 변환 + 브라우저블 API
- 자동 에러 처리

**장점**:
- ✅ 간단한 구조
- ✅ 빠른 작성

**단점**:
- ❌ if/elif로 HTTP 메서드 구분
- ❌ 코드가 길어지면 복잡

**언제 쓰나**: 간단한 API, 커스텀 로직 많을 때

---

### 4. APIView (클래스 기반 뷰)

**특징**: HTTP 메서드별로 함수 분리

```python
from rest_framework.views import APIView
from rest_framework.response import Response

class BookList(APIView):
    def get(self, request):
        """목록 조회"""
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

    def post(self, request):
        """생성"""
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)

class BookDetail(APIView):
    def get_object(self, pk):
        """재사용 메서드"""
        return Book.objects.get(pk=pk)

    def get(self, request, pk):
        """상세 조회"""
        book = self.get_object(pk)
        serializer = BookSerializer(book)
        return Response(serializer.data)

    def put(self, request, pk):
        """수정"""
        book = self.get_object(pk)
        serializer = BookSerializer(book, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=400)

    def delete(self, request, pk):
        """삭제"""
        book = self.get_object(pk)
        book.delete()
        return Response(status=204)
```

**장점**:
- ✅ HTTP 메서드 명확히 분리 (get, post, put, delete)
- ✅ 코드 구조화 (get_object 같은 재사용 메서드)
- ✅ 클래스 상속으로 확장 가능

**단점**:
- ❌ 여전히 반복 코드 많음
- ❌ CRUD 로직 직접 작성

**언제 쓰나**: 복잡한 커스터마이징, 코드 구조화 필요할 때

---

### 5. Generics (자동화된 뷰)

**특징**: CRUD 로직 자동 제공

```python
from rest_framework import generics

class BookList(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

class BookDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

**제공되는 Generic Views**:

| 클래스 | 제공 기능 | HTTP 메서드 |
|--------|----------|------------|
| **ListAPIView** | 목록 조회 | GET |
| **CreateAPIView** | 생성 | POST |
| **ListCreateAPIView** | 목록 조회 + 생성 | GET, POST |
| **RetrieveAPIView** | 상세 조회 | GET |
| **UpdateAPIView** | 수정 | PUT, PATCH |
| **DestroyAPIView** | 삭제 | DELETE |
| **RetrieveUpdateDestroyAPIView** | 상세 + 수정 + 삭제 | GET, PUT, PATCH, DELETE |

**장점**:
- ✅ 최소한의 코드 (2줄!)
- ✅ 표준 CRUD 자동 제공
- ✅ 페이지네이션, 필터링 자동
- ✅ 검증된 로직

**단점**:
- ❌ 커스터마이징 제한적 (필요 시 메서드 오버라이딩)

**언제 쓰나**: 일반적인 CRUD API (가장 많이 씀)

**커스터마이징 예시**:
```python
class BookList(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get_queryset(self):
        """필터링 커스터마이징"""
        queryset = super().get_queryset()
        author = self.request.query_params.get('author')
        if author:
            queryset = queryset.filter(author=author)
        return queryset
```

---

## 🔀 언제 무엇을 쓸까?

| 상황 | 추천 방법 | 이유 |
|------|----------|------|
| **간단한 API** | @api_view | 빠르고 직관적 |
| **표준 CRUD** | Generics | 코드 최소화 |
| **복잡한 로직** | APIView | 유연한 커스터마이징 |
| **특수한 경우** | APIView 상속 | Generics 베이스 + 커스터마이징 |

### 실전 예시

```python
# 1. 간단한 통계 API → @api_view
@api_view(['GET'])
def book_statistics(request):
    total = Book.objects.count()
    return Response({'total': total})

# 2. 일반 CRUD → Generics
class BookList(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# 3. 복잡한 검색 API → APIView
class BookSearch(APIView):
    def post(self, request):
        # 복잡한 검색 로직
        filters = request.data.get('filters', {})
        books = Book.objects.filter(**filters)
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)
```

---

## 💡 왜 배웠나?

**개인 학습**으로:
- DRF의 핵심 개념 이해
- API 개발 도구 선택 능력
- 실전 프로젝트 적용 준비

---

## 🔧 핵심 도구 비교

| 도구 | 코드량 | 유연성 | 사용 빈도 |
|------|--------|--------|----------|
| **@api_view** | 중간 | 높음 | ⭐⭐⭐ |
| **APIView** | 많음 | 매우 높음 | ⭐⭐ |
| **Generics** | 적음 | 중간 | ⭐⭐⭐⭐⭐ |

**비유**:
- **@api_view**: 🔧 만능 공구 (다재다능)
- **APIView**: 🏗 건물 골조 (구조적)
- **Generics**: 🏢 완성된 건물 (즉시 사용)

---

## 🔗 관련 개념

- [Django 백엔드 패턴](../django-backend-patterns/): Django ORM, API, 스케줄링

---

## ❓ 면접 예상 질문

**Q1: @api_view와 APIView의 차이는?**
> A: @api_view는 함수 기반으로 간단하지만 if/elif로 HTTP 메서드를 구분합니다. APIView는 클래스 기반으로 get(), post() 등 메서드별로 명확히 분리되어 구조적입니다. 간단한 API는 @api_view, 복잡한 API는 APIView가 적합합니다.

**Q2: Generics를 사용하는 이유는?**
> A: 표준 CRUD 작업을 자동화합니다. queryset과 serializer_class만 지정하면 목록 조회, 생성, 수정, 삭제 등이 자동으로 구현됩니다. 코드량이 최소화되고 검증된 로직을 사용할 수 있습니다.

**Q3: 언제 APIView를 직접 사용하나요?**
> A: Generics로 해결 안 되는 복잡한 로직이 필요할 때입니다. 여러 모델 조합, 복잡한 필터링, 특수한 권한 체크 등이 필요하면 APIView를 상속하여 커스터마이징합니다.

**Q4: Serializer와 ModelSerializer의 차이는?**
> A: Serializer는 모든 필드와 create/update 메서드를 수동으로 정의합니다. ModelSerializer는 Django 모델을 기반으로 필드와 메서드를 자동 생성합니다. 실전에서는 대부분 ModelSerializer를 사용합니다.

**Q5: 직렬화와 역직렬화란?**
> A: 직렬화는 Python 객체를 JSON으로 변환하여 프론트엔드로 보내는 것이고, 역직렬화는 프론트엔드에서 받은 JSON을 Python 객체로 변환하여 DB에 저장하는 것입니다. Serializer가 양방향 변환을 담당합니다.

---

## 📚 핵심 개념 정리

### 1. API = JSON 상하차
```
프론트 → JSON → Serializer → DB (역직렬화)
DB → Serializer → JSON → 프론트 (직렬화)
```

### 2. Serializer
- **Serializer**: 수동 정의
- **ModelSerializer**: 자동 생성 (실전 추천)

### 3. View 3가지
- **@api_view**: 함수 기반, 간단한 API
- **APIView**: 클래스 기반, 구조화
- **Generics**: 자동화, 표준 CRUD

### 4. 선택 기준
```
표준 CRUD → Generics (1순위)
간단한 커스텀 → @api_view
복잡한 커스텀 → APIView
```

---

**핵심 원칙**:
1. **실전**: Generics 먼저 고려
2. **커스터마이징**: 필요할 때만 APIView
3. **간단한 API**: @api_view
4. **Serializer**: 거의 항상 ModelSerializer

---

**배운 점**:
- DRF는 JSON 상하차 자동화 도구
- Generics가 가장 효율적 (코드 최소화)
- 상황에 맞게 도구 선택 중요
- 복잡도와 유연성은 트레이드오프

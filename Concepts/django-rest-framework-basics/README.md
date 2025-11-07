# Django REST Framework Basics

## 🎯 Summary

Core DRF concepts and differences between features (@api_view, APIView, Generics)

---

## 📌 Key Points

### 1. API Essence: JSON Loading/Unloading System

**Backend Core = JSON Loading/Unloading**

```
Frontend ←→ [JSON 📦] ←→ Serializer ←→ Database
```

#### Serialization (Loading)
```python
# DB data → Package as JSON
book = Book.objects.get(id=1)
serializer = BookSerializer(book)
json_data = serializer.data  # Send to frontend
```

#### Deserialization (Unloading)
```python
# JSON → Save to DB
data = request.data  # JSON from frontend
serializer = BookSerializer(data=data)
if serializer.is_valid():
    serializer.save()  # Save to DB
```

---

### 2. Serializer vs ModelSerializer

#### A. Serializer (Manual)

**Feature**: Define everything manually

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

**When to use**: Non-model data, complex customization

#### B. ModelSerializer (Automatic)

**Feature**: Auto-generate based on model

```python
class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = '__all__'  # or ['title', 'author']
        # create(), update() auto-implemented!
```

**When to use**: Model-based general CRUD (most cases)

**Difference Summary**:
- Serializer: Write fields, create(), update() manually
- ModelSerializer: Auto-generate by specifying model only

---

### 3. @api_view (Function-Based View)

**Feature**: Add DRF functionality to functions

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

**Provided Features**:
- `request.data`: Auto JSON parsing
- `Response`: Auto JSON conversion + Browsable API
- Automatic error handling

**Pros**:
- ✅ Simple structure
- ✅ Quick to write

**Cons**:
- ❌ HTTP methods separated by if/elif
- ❌ Complex when code grows

**When to use**: Simple APIs, lots of custom logic

---

### 4. APIView (Class-Based View)

**Feature**: Separate functions per HTTP method

```python
from rest_framework.views import APIView
from rest_framework.response import Response

class BookList(APIView):
    def get(self, request):
        """List view"""
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

    def post(self, request):
        """Create"""
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)

class BookDetail(APIView):
    def get_object(self, pk):
        """Reusable method"""
        return Book.objects.get(pk=pk)

    def get(self, request, pk):
        """Detail view"""
        book = self.get_object(pk)
        serializer = BookSerializer(book)
        return Response(serializer.data)

    def put(self, request, pk):
        """Update"""
        book = self.get_object(pk)
        serializer = BookSerializer(book, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=400)

    def delete(self, request, pk):
        """Delete"""
        book = self.get_object(pk)
        book.delete()
        return Response(status=204)
```

**Pros**:
- ✅ Clear HTTP method separation (get, post, put, delete)
- ✅ Code structure (reusable methods like get_object)
- ✅ Extensible via class inheritance

**Cons**:
- ❌ Still repetitive code
- ❌ Must write CRUD logic manually

**When to use**: Complex customization, need code structure

---

### 5. Generics (Automated Views)

**Feature**: Auto-provide CRUD logic

```python
from rest_framework import generics

class BookList(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

class BookDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

**Available Generic Views**:

| Class | Features | HTTP Methods |
|-------|----------|--------------|
| **ListAPIView** | List view | GET |
| **CreateAPIView** | Create | POST |
| **ListCreateAPIView** | List + Create | GET, POST |
| **RetrieveAPIView** | Detail view | GET |
| **UpdateAPIView** | Update | PUT, PATCH |
| **DestroyAPIView** | Delete | DELETE |
| **RetrieveUpdateDestroyAPIView** | Detail + Update + Delete | GET, PUT, PATCH, DELETE |

**Pros**:
- ✅ Minimal code (2 lines!)
- ✅ Standard CRUD auto-provided
- ✅ Pagination, filtering automatic
- ✅ Proven logic

**Cons**:
- ❌ Limited customization (override methods if needed)

**When to use**: General CRUD APIs (most common)

**Customization Example**:
```python
class BookList(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get_queryset(self):
        """Customize filtering"""
        queryset = super().get_queryset()
        author = self.request.query_params.get('author')
        if author:
            queryset = queryset.filter(author=author)
        return queryset
```

---

## 🔀 When to Use What?

| Situation | Recommended | Reason |
|-----------|------------|--------|
| **Simple API** | @api_view | Fast and intuitive |
| **Standard CRUD** | Generics | Minimize code |
| **Complex logic** | APIView | Flexible customization |
| **Special cases** | APIView inheritance | Generics base + customization |

### Practical Examples

```python
# 1. Simple statistics API → @api_view
@api_view(['GET'])
def book_statistics(request):
    total = Book.objects.count()
    return Response({'total': total})

# 2. General CRUD → Generics
class BookList(generics.ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# 3. Complex search API → APIView
class BookSearch(APIView):
    def post(self, request):
        # Complex search logic
        filters = request.data.get('filters', {})
        books = Book.objects.filter(**filters)
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)
```

---

## 💡 Why I Learned This

**Personal learning** to:
- Understand core DRF concepts
- Develop API tool selection skills
- Prepare for real-world project application

---

## 🔧 Core Tool Comparison

| Tool | Code Amount | Flexibility | Usage Frequency |
|------|-------------|-------------|-----------------|
| **@api_view** | Medium | High | ⭐⭐⭐ |
| **APIView** | Large | Very High | ⭐⭐ |
| **Generics** | Small | Medium | ⭐⭐⭐⭐⭐ |

**Analogies**:
- **@api_view**: 🔧 Multi-tool (versatile)
- **APIView**: 🏗 Building frame (structural)
- **Generics**: 🏢 Completed building (ready to use)

---

## 🔗 Related Concepts

- [Django Backend Patterns](../django-backend-patterns/): Django ORM, API, Scheduling

---

## ❓ Interview Prep Questions

**Q1: What's the difference between @api_view and APIView?**
> A: @api_view is function-based and simple but uses if/elif to distinguish HTTP methods. APIView is class-based with clear separation via get(), post(), etc., making it more structural. Use @api_view for simple APIs, APIView for complex ones.

**Q2: Why use Generics?**
> A: They automate standard CRUD operations. Just specify queryset and serializer_class, and list view, create, update, delete are automatically implemented. Minimizes code and uses proven logic.

**Q3: When to directly use APIView?**
> A: When complex logic can't be solved by Generics. For combining multiple models, complex filtering, or special permission checks, inherit APIView for customization.

**Q4: What's the difference between Serializer and ModelSerializer?**
> A: Serializer requires manually defining all fields and create/update methods. ModelSerializer auto-generates fields and methods based on Django models. ModelSerializer is used in most real-world cases.

**Q5: What are serialization and deserialization?**
> A: Serialization converts Python objects to JSON for sending to frontend. Deserialization converts JSON received from frontend to Python objects for DB storage. Serializer handles both conversions.

---

## 📚 Core Concept Summary

### 1. API = JSON Loading/Unloading
```
Frontend → JSON → Serializer → DB (Deserialization)
DB → Serializer → JSON → Frontend (Serialization)
```

### 2. Serializer
- **Serializer**: Manual definition
- **ModelSerializer**: Auto-generation (recommended for practice)

### 3. Three View Types
- **@api_view**: Function-based, simple APIs
- **APIView**: Class-based, structured
- **Generics**: Automated, standard CRUD

### 4. Selection Criteria
```
Standard CRUD → Generics (1st choice)
Simple custom → @api_view
Complex custom → APIView
```

---

**Core Principles**:
1. **Practice**: Consider Generics first
2. **Customization**: APIView only when needed
3. **Simple APIs**: @api_view
4. **Serializer**: Almost always ModelSerializer

---

**Learnings**:
- DRF is a JSON loading/unloading automation tool
- Generics are most efficient (minimize code)
- Tool selection based on situation is important
- Complexity and flexibility are trade-offs

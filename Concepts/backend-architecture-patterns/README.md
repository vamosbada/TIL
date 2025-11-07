# Backend Architecture Patterns

## 🎯 Summary

Backend code structure design methods: Understanding layered, microservices, and other major patterns

---

## 📌 Key Points

### 1. What is Architecture Design?

**Concept**: Like drawing blueprints before building a house, planning how to organize and structure code in advance

**Why necessary?**
- ✅ Easier maintenance
- ✅ Better team collaboration
- ✅ Stable structure during scaling
- ✅ Fewer bugs (reduced side effects through separation of concerns)

---

## 🏗 5 Major Architecture Patterns

### 1. Layered Architecture

**Analogy**: 🏢 Apartment Building

**Structure**: Stacking layers vertically
```
4th Floor: UI (Controller)
3rd Floor: Business Logic (Service)
2nd Floor: Data Access (Repository)
1st Floor: Database
```

**Core Rule**: Access only from upper to lower layers
- ✅ 4th → 3rd → 2nd → 1st
- ❌ 4th → 1st (direct access prohibited!)

**When to use**:
- Most web services (90%+)
- Startups, small-to-medium companies
- Personal projects, portfolios

---

### 2. Client-Server

**Analogy**: 🍽 Restaurant

```
Customer (Client)
  ↓ "Bulgogi please" (HTTP Request)
Order Counter (API)
  ↓
Kitchen (Server)
  ↓ "Bulgogi ready!" (HTTP Response)
Deliver to Customer
```

**Feature**: Customer doesn't need to know how the kitchen operates

**When to use**:
- Web/mobile apps
- REST API, GraphQL

---

### 3. Microservices

**Analogy**: 🚚 Food Truck Village

```
Kimbap Truck (Kimbap only)    Taco Truck (Tacos only)    Coffee Truck (Coffee only)
       ↓                             ↓                           ↓
Each operates independently    If Kimbap truck breaks    Taco truck still works!
```

**Features**:
- Each service is independent
- One failure doesn't affect others
- Separate teams per service possible

**When to use**:
- Netflix, Amazon, Google
- Team size 100+
- Massive traffic
- ⚠️ 10x+ cost

---

### 4. BFF (Backend for Frontend)

**Analogy**: 🏬 Department Store Information Desk

```
Web Customer      → Web Desk      → Customized Service
Mobile Customer   → Mobile Desk   → Customized Service
VIP Customer      → VIP Desk      → Customized Service
```

**Feature**: Different APIs for different client types

**When to use**:
- Web/mobile simultaneous service
- Different requirements per platform

---

### 5. Orchestration

**Analogy**: 💒 Wedding Planner

```
Bride & Groom: "Plan our wedding"
         ↓
  Wedding Planner (Coordinator)
    ├→ Book band
    ├→ Contact caterer
    ├→ Reserve photographer
    ├→ Order flowers
    └→ Rent limousine
```

**Feature**: Coordinates and manages multiple services

**When to use**:
- Complex business flows
- Multiple service integration needed

---

## 🏢 Layered Architecture in Detail

**Most commonly used pattern (90%+)**

### A. 4-Layer Structure

```
┌─────────────────────────────────┐
│  4th: Controller                 │  ← Receives user requests
│  "Receive & respond only"       │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│  3rd: Service                    │  ← Actual business logic
│  "Does the actual work"         │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│  2nd: Repository                 │  ← DB access only
│  "DB specialist"                 │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│  1st: Database                   │  ← Actual data
└─────────────────────────────────┘
```

### B. Role of Each Layer

#### Controller (4th Floor)
```python
@app.route('/users', methods=['GET'])
def get_users():
    # Receive request
    # Call Service
    users = user_service.get_all_users()
    # Return response
    return jsonify(users)
```

**Responsibilities**:
- Receive HTTP requests
- Call Service
- Return HTTP responses
- ❌ No business logic!

#### Service (3rd Floor)
```python
class UserService:
    def get_all_users(self):
        # Business logic
        users = user_repository.find_all()
        # Data processing
        return [self.format_user(u) for u in users]
```

**Responsibilities**:
- Actual business logic
- Data processing/validation
- Call Repository
- ❌ No direct DB access!

#### Repository (2nd Floor)
```python
class UserRepository:
    def find_all(self):
        # DB access only
        return db.session.query(User).all()

    def find_by_id(self, user_id):
        return db.session.query(User).filter_by(id=user_id).first()
```

**Responsibilities**:
- DB access only (SQL queries)
- CRUD operations
- ❌ No business logic!

#### Database (1st Floor)
```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100))
    email = db.Column(db.String(100))
```

**Responsibility**: Store actual data

---

### C. Advantages of Layered Architecture

**1. Clear Responsibilities**
```
"Where's the user list?"
→ Check Controller!

"Where's the business logic?"
→ Check Service!

"Where's the DB query?"
→ Check Repository!
```

**2. Easy Testing**
```python
# Service testing (using Repository mock)
def test_get_users():
    mock_repo = Mock()
    service = UserService(mock_repo)
    # Test Service only!
```

**3. Easy Maintenance**
```
DB Change:
✅ Modify Repository only (1 file)

vs

Without layers:
❌ Modify 10~15 files
```

**4. Reusability**
```python
# Service can be used by multiple Controllers
user_service.get_all_users()  # REST API
user_service.get_all_users()  # GraphQL
user_service.get_all_users()  # CLI
```

---

## 🔀 Architecture Selection Criteria

| Criteria | Layered | Microservices |
|----------|---------|---------------|
| **Team Size** | 1~10 people | 100+ people |
| **Service Complexity** | Simple~Medium | Very Complex |
| **Traffic** | ~Thousands | Tens of thousands+ |
| **Budget** | Low | High (10x+) |
| **Development Speed** | Fast (MVP) | Slow (initial setup) |
| **Scalability** | Medium | Very High |

**Real-world Examples**:
- **Netflix**: Early layered → Current microservices
- **Facebook**: Early layered → Growth phase client-server → Current microservices + BFF
- **Toss**: Payments (microservices) + Internal admin (layered)

---

## 💡 Why I Learned This

**Personal learning** to:
- Discover structural issues in actual project code
- Realize necessity of layered architecture
- Set refactoring direction

---

## 🔧 Practical Experience: Current Code Issues

### ❌ Problem: Layer Jumping

```python
# routes.py (Controller)
@app.route('/users')
def get_users():
    # Jump from 4th floor to 1st floor!
    users = db.session.query(User).all()  # ❌
    return jsonify(users)
```

**Problems**:
1. **Code Duplication**: Same query copy-pasted in 10~15 files
2. **Maintenance Hell**: When DB schema changes, modify 10~15 files
3. **Testing Difficulty**: Controller and DB are tightly coupled
4. **Mixed Responsibilities**: All logic in one file

### ✅ Solution: Layer Separation

```python
# controller.py (4th Floor)
@app.route('/users')
def get_users():
    users = user_service.get_all_users()  # Call Service
    return jsonify(users)

# service.py (3rd Floor)
class UserService:
    def get_all_users(self):
        return user_repository.find_all()  # Call Repository

# repository.py (2nd Floor)
class UserRepository:
    def find_all(self):
        return db.session.query(User).all()  # DB access
```

**Effects**:
- ✅ Remove duplication: Query in one place only
- ✅ Maintenance: DB change requires modifying 1 file only
- ✅ Testing: Each layer independently testable
- ✅ Clear responsibilities: Each has clear role

---

## 🔗 Related Concepts

- [Django Backend Patterns](../django-backend-patterns/): Actual implementation in Django

---

## ❓ Interview Prep Questions

**Q1: What is layered architecture?**
> A: A structure that divides code into 4 layers: Controller, Service, Repository, Database. Access is only from upper to lower layers, with clear responsibilities for each. Controller handles requests/responses only, Service handles business logic, and Repository handles DB access only.

**Q2: Why shouldn't Controller directly access DB?**
> A: Responsibilities get mixed and maintenance becomes difficult. When DB schema changes, all Controllers must be modified, and the same query gets duplicated everywhere. Separating into Repository means only one place needs modification when DB changes.

**Q3: What's the difference between microservices and layered architecture?**
> A: Layered architecture divides code into layers within a single application, while microservices separate into completely independent applications by functionality. Microservices have higher scalability but 10x+ cost and complexity.

**Q4: When to use layered vs microservices?**
> A: Use layered for teams under 10 people with moderate traffic, microservices for teams of 100+ with massive traffic. Even companies like Netflix started with layered architecture. Over 90% of services are sufficient with layered architecture.

**Q5: Why is Service layer necessary?**
> A: To separate business logic from DB access. Putting business logic in Controller makes testing difficult, and putting it in Repository makes reuse difficult. Service combines multiple Repositories to handle complex business logic and can be reused by multiple Controllers.

---

## 📚 Key Summary

### 1. Architecture is Blueprint
```
Plan structure before writing code
→ Easy maintenance
→ Better team collaboration
→ Fewer bugs
```

### 2. Layered is Dominant (90%+)
```
Controller (request/response)
    ↓
Service (business logic)
    ↓
Repository (DB access)
    ↓
Database (data)
```

### 3. Core Rules
```
✅ Access upper → lower only
❌ No layer jumping
✅ Clear responsibilities per layer
```

### 4. Selection Criteria
```
Simple project → Layered
Complex enterprise → Microservices
No need to be perfect from start (can change later)
```

---

**Core Principles**:
1. **Separation**: Separation of Concerns
2. **Responsibility**: Single Responsibility Principle
3. **Dependency**: Upper to lower only (Dependency Direction)
4. **Pragmatism**: Choose based on situation

---

**Learnings**:
- Architecture isn't complexity, it's "organization"
- Seems tedious at first but much easier later
- Most projects are sufficient with layered
- Must experience actual problems to understand necessity

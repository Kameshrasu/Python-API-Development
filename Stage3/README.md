# 📍 Stage 3: Building APIs in Python

## 1️⃣ Introduction to Web Frameworks

### What is a Web Framework?

A **web framework** is a collection of tools, libraries, and conventions that help you build web applications and APIs more easily. Instead of writing everything from scratch, frameworks provide:

- **Routing** → Map URLs to functions
- **Request handling** → Process incoming HTTP requests
- **Response formatting** → Send back JSON, HTML, etc.
- **Middleware** → Add functionality between request and response
- **Error handling** → Manage exceptions gracefully

### Flask vs FastAPI

| Aspect | Flask | FastAPI |
|--------|-------|---------|
| **Release Year** | 2010 | 2018 |
| **Philosophy** | Minimal, flexible | Modern, high-performance |
| **Learning Curve** | Easy to start | Medium complexity |
| **Type Hints** | Optional | Required/Recommended |
| **Performance** | Good | Excellent (async) |
| **Documentation** | Manual setup | Auto-generated |
| **Data Validation** | Manual/Extensions | Built-in (Pydantic) |
| **Async Support** | Limited | Native |
| **Community** | Large, mature | Growing rapidly |

### When to Choose Flask

**Choose Flask when:**
- You're **learning** web development
- Building **simple REST APIs**
- Need **maximum flexibility**
- Working on **small to medium** projects
- Want to understand **how things work** under the hood
- Team prefers **minimal dependencies**

**Example Use Cases:**
- Learning projects
- Prototypes
- Simple CRUD APIs
- Microservices with basic requirements

### When to Choose FastAPI

**Choose FastAPI when:**
- Building **production APIs**
- Need **high performance** (async/await)
- Want **automatic documentation** (Swagger/OpenAPI)
- Working with **modern Python** (3.6+)
- Need **strong data validation**
- Building **complex APIs** with many endpoints

**Example Use Cases:**
- Production web APIs
- Machine learning APIs
- Real-time applications
- APIs with complex data validation
- Microservices in modern architecture

---

## 2️⃣ Creating Endpoints

### What is an Endpoint?

An **endpoint** is a specific URL where your API can be accessed. It combines:
- **URL path** (e.g., `/users`)
- **HTTP method** (GET, POST, etc.)
- **Function** that handles the request

### Understanding Routes

**Route** = URL pattern that maps to a function

```
URL: /users/123
├── /users (base path)
├── /123 (path parameter)
└── Function that handles this request
```

### Path Parameters

**Path parameters** are part of the URL itself:

```
/users/123        → user_id = 123
/posts/456/edit   → post_id = 456, action = edit
/api/v1/books/789 → version = v1, book_id = 789
```

**Characteristics:**
- **Required** → Must be provided in URL
- **Part of URL structure**
- **Usually represent resource IDs**

### Query Parameters

**Query parameters** come after `?` in the URL:

```
/users?limit=10&page=2&sort=name
├── limit=10 (how many items)
├── page=2 (which page)
└── sort=name (sort order)
```

**Characteristics:**
- **Optional** → Can have default values
- **Come after `?`** in URL
- **Separated by `&`**
- **Used for filtering, pagination, sorting**

### JSON Responses

APIs typically return data in **JSON format** because:
- **Language independent** → Works with any programming language
- **Lightweight** → Less data transfer than XML
- **Easy to parse** → Most languages have built-in JSON support
- **Human readable** → Easy to debug

**Good API Response Structure:**
```json
{
  "success": true,
  "data": [...],
  "message": "Operation completed successfully",
  "meta": {
    "total": 100,
    "page": 1,
    "per_page": 10
  }
}
```

---

## 3️⃣ Handling HTTP Methods

### Understanding CRUD Operations

**CRUD** = Create, Read, Update, Delete

| CRUD | HTTP Method | Purpose | Example |
|------|-------------|---------|---------|
| **Create** | POST | Add new data | Create new user |
| **Read** | GET | Retrieve data | Get user info |
| **Update** | PUT/PATCH | Modify data | Update user details |
| **Delete** | DELETE | Remove data | Delete user |

### GET - Fetch Data

**Purpose:** Retrieve information without changing anything

**Characteristics:**
- **Safe** → Doesn't modify server state
- **Idempotent** → Multiple identical requests have same effect
- **Cacheable** → Responses can be cached
- **No request body** → Data passed via URL parameters

**Examples:**
```
GET /users           → Get all users
GET /users/123       → Get specific user
GET /users?page=2    → Get users with pagination
```

### POST - Create Data

**Purpose:** Create new resources

**Characteristics:**
- **Not safe** → Changes server state
- **Not idempotent** → Multiple requests create multiple resources
- **Has request body** → Data sent in JSON format
- **Returns 201** → "Created" status code

**Examples:**
```
POST /users          → Create new user
POST /posts          → Create new blog post
POST /orders         → Place new order
```

### PUT - Replace Data

**Purpose:** Replace entire resource

**Characteristics:**
- **Not safe** → Changes server state
- **Idempotent** → Multiple identical requests have same effect
- **Complete replacement** → All fields must be provided
- **Returns 200** → "OK" status code

**Example:**
```
PUT /users/123
{
  "name": "John Smith",
  "email": "john@example.com",
  "age": 30
}
```
*All fields replaced, missing fields become null/empty*

### PATCH - Update Data

**Purpose:** Partially update resource

**Characteristics:**
- **Not safe** → Changes server state
- **Can be idempotent** → Depends on implementation
- **Partial update** → Only provided fields are changed
- **Returns 200** → "OK" status code

**Example:**
```
PATCH /users/123
{
  "email": "newemail@example.com"
}
```
*Only email updated, other fields unchanged*

### DELETE - Remove Data

**Purpose:** Remove resources

**Characteristics:**
- **Not safe** → Changes server state
- **Idempotent** → Multiple deletes have same effect
- **Usually no body** → Resource ID in URL
- **Returns 200/204** → "OK" or "No Content"

**Examples:**
```
DELETE /users/123    → Delete specific user
DELETE /posts/456    → Delete blog post
```

---

## 4️⃣ Request Validation

### Why Validate Input?

**Security reasons:**
- Prevent **injection attacks**
- Avoid **malformed data** breaking your app
- Ensure **data integrity**

**User experience:**
- Provide **clear error messages**
- **Fail fast** → Don't process invalid data
- **Consistent responses**

### Types of Validation

**1. Data Type Validation**
```
age: must be integer
email: must be valid email format
name: must be string
```

**2. Range Validation**
```
age: must be between 0 and 150
password: minimum 8 characters
price: must be positive number
```

**3. Format Validation**
```
email: user@domain.com format
phone: +1-234-567-8900 format
date: YYYY-MM-DD format
```

**4. Business Rule Validation**
```
username: must be unique
order_quantity: can't exceed stock
start_date: must be before end_date
```

### Flask Validation Approaches

**Manual Validation:**
```python
if not data or 'name' not in data:
    return {"error": "Name required"}, 400

if len(data['name']) < 2:
    return {"error": "Name too short"}, 400
```

**With Marshmallow Library:**
- Define **schemas** for data structure
- **Automatic validation** based on schema
- **Serialization/Deserialization**
- **Custom validators**

### FastAPI Validation (Pydantic)

**Built-in validation** with Pydantic models:
- **Type checking** → Automatic based on type hints
- **Custom validators** → Write your own rules
- **Nested validation** → Validate complex data structures
- **Automatic error messages** → User-friendly responses

### Handling Invalid Requests

**What to return when validation fails:**

**Status Codes:**
- `400 Bad Request` → Client sent invalid data
- `422 Unprocessable Entity` → Data format correct but validation failed

**Response Format:**
```json
{
  "error": "Validation Error",
  "details": {
    "name": ["Name is required"],
    "email": ["Invalid email format"],
    "age": ["Must be between 0 and 150"]
  }
}
```

---

## 5️⃣ Middleware & Logging

### What is Middleware?

**Middleware** is code that runs **between** receiving a request and sending a response.

**Request Flow with Middleware:**
```
Client Request
    ↓
Middleware 1 (e.g., Authentication)
    ↓
Middleware 2 (e.g., Logging)
    ↓
Your Route Function
    ↓
Middleware 2 (Response processing)
    ↓
Middleware 1 (Response processing)
    ↓
Client Response
```

### Common Middleware Use Cases

**1. Authentication/Authorization**
- Check if user is logged in
- Verify API keys
- Role-based access control

**2. Logging**
- Log all incoming requests
- Track response times
- Monitor API usage

**3. CORS (Cross-Origin Resource Sharing)**
- Allow requests from web browsers
- Configure allowed origins
- Handle preflight requests

**4. Rate Limiting**
- Prevent API abuse
- Limit requests per user/IP
- Throttle heavy users

**5. Request/Response Modification**
- Add custom headers
- Transform request data
- Modify response format

### Why Logging is Important

**Debugging:**
- **Track errors** → See what went wrong
- **Monitor performance** → Find slow endpoints
- **Understand usage** → Which endpoints are popular

**Security:**
- **Audit trail** → Who accessed what and when
- **Detect attacks** → Unusual request patterns
- **Compliance** → Some industries require logs

**Operations:**
- **Health monitoring** → Is the API working?
- **Capacity planning** → Do we need more servers?
- **User behavior** → How do users interact with API?

### What to Log

**Request Information:**
- HTTP method and URL
- Request headers (be careful with sensitive data)
- Request body (for POST/PUT requests)
- User/API key (for authentication)
- Timestamp

**Response Information:**
- HTTP status code
- Response time
- Response size
- Any errors that occurred

**Example Log Entry:**
```
2025-08-12 10:30:15 - INFO - [req-123] POST /users - 201 - 45ms
2025-08-12 10:30:16 - ERROR - [req-124] GET /users/999 - 404 - User not found
```

### Log Levels

**DEBUG** → Detailed info for developers
```
Processing user data: {"name": "John", "email": "john@example.com"}
```

**INFO** → General operational messages
```
User 123 created successfully
API request: GET /users - 200 OK
```

**WARNING** → Something unusual but not an error
```
Slow query detected: took 2.5 seconds
Rate limit approaching for user 456
```

**ERROR** → Something went wrong
```
Database connection failed
Invalid API key used
```

**CRITICAL** → Serious problems
```
Database server down
API completely unavailable
```

### Middleware Best Practices

**1. Order Matters**
- Authentication before authorization
- Logging early in the chain
- CORS before other processing

**2. Performance**
- Keep middleware lightweight
- Avoid heavy processing
- Use async when possible

**3. Error Handling**
- Middleware should handle its own errors
- Don't break the request chain
- Log middleware errors

**4. Security**
- Don't log sensitive data (passwords, tokens)
- Validate middleware inputs
- Use HTTPS in production

---

## 🎯 Key Concepts Summary

### Framework Choice Decision Tree
```
Need to learn basics? → Flask
Need high performance? → FastAPI
Building simple API? → Flask
Need auto-documentation? → FastAPI
Team new to Python? → Flask
Building production API? → FastAPI
```

### HTTP Methods Quick Reference
```
GET    → Read data (safe, idempotent)
POST   → Create data (not safe, not idempotent)
PUT    → Replace data (not safe, idempotent)
PATCH  → Update data (not safe, maybe idempotent)
DELETE → Remove data (not safe, idempotent)
```

### Validation Strategy
```
1. Validate early → Before processing
2. Fail fast → Return errors immediately
3. Clear messages → Help users fix issues
4. Consistent format → Same error structure
```

### Logging Strategy
```
1. Log requests → Method, URL, timestamp
2. Log responses → Status code, response time
3. Log errors → What went wrong
4. Log security → Authentication attempts
5. Don't log secrets → Passwords, tokens
```

**Next:** We'll implement these concepts with practical code examples! 🚀

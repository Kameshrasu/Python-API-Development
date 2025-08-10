# 📍 Fundamentals of APIs

## 1️⃣ What is an API?

**API** = **Application Programming Interface**

An API is **not** software or a website — it's a **set of rules** that define how two applications communicate with each other.

### 🥗 Real-World Analogy

Think of an API like ordering food at a restaurant:

- **Menu** → API documentation (tells you what's available)
- **Waiter** → API (takes your request & delivers the result)
- **Kitchen** → Server or database (processes the request)
- **Customer** → You (the client making the request)

### 🖥 Key Terms

- **Client** → The program making requests (your app, browser, mobile app)
- **Server** → The program providing responses (database, web service)
- **Endpoint** → A specific URL where you can access an API (e.g., `/users/1`)

---

## 2️⃣ Difference between REST, SOAP, and GraphQL

### REST API
- **Most common** type of API
- Uses **HTTP methods** (`GET`, `POST`, `PUT`, `DELETE`)
- Returns data in **JSON** format
- **Stateless** → Each request is independent
- Simple and widely supported

```http
GET /users/1
Response: {"id": 1, "name": "John", "email": "john@example.com"}
```

### SOAP API
- **XML-based** communication
- More **complex** but very **secure**
- Used in **banking** and **enterprise** systems
- Has built-in error handling

```xml
<soap:Envelope>
  <soap:Body>
    <GetUserRequest>
      <UserId>1</UserId>
    </GetUserRequest>
  </soap:Body>
</soap:Envelope>
```

### GraphQL API
- **Single endpoint** for all operations
- Client specifies **exactly what data** it needs
- No over-fetching or under-fetching
- More flexible than REST

```graphql
query {
  user(id: 1) {
    name
    email
  }
}
```

**Quick Comparison:**

| Feature | REST | SOAP | GraphQL |
|---------|------|------|---------|
| Data Format | JSON | XML | JSON |
| Learning Curve | Easy | Hard | Medium |
| Flexibility | Medium | Low | High |
| Use Case | Web APIs | Enterprise | Modern Apps |

---

## 3️⃣ HTTP Methods

HTTP methods tell the server **what action** you want to perform:

| Method | Purpose | Example | Description |
|--------|---------|---------|-------------|
| `GET` | **Read** data | `GET /users` | Get list of all users |
| `POST` | **Create** new data | `POST /users` | Create a new user |
| `PUT` | **Replace** entire resource | `PUT /users/5` | Replace user 5 completely |
| `PATCH` | **Update** part of resource | `PATCH /users/5` | Update only email of user 5 |
| `DELETE` | **Remove** data | `DELETE /users/5` | Delete user 5 |

### Examples:

**GET - Retrieve data:**
```http
GET /users/1
Response: {"id": 1, "name": "John", "role": "Developer"}
```

**POST - Create new data:**
```http
POST /users
Content-Type: application/json

{
  "name": "Alice",
  "role": "Designer"
}

Response: {"id": 2, "name": "Alice", "role": "Designer"}
```

**PUT - Replace entire resource:**
```http
PUT /users/1
Content-Type: application/json

{
  "name": "John Smith",
  "role": "Senior Developer"
}
```

**PATCH - Update partially:**
```http
PATCH /users/1
Content-Type: application/json

{
  "role": "Senior Developer"
}
```

**DELETE - Remove data:**
```http
DELETE /users/1
Response: {"message": "User deleted successfully"}
```

---

## 4️⃣ Understanding HTTP Status Codes

Status codes tell you **what happened** with your request:

### 2xx - Success ✅
| Code | Meaning | When It Happens |
|------|---------|-----------------|
| `200` | OK | GET request successful |
| `201` | Created | POST request created new resource |

### 4xx - Client Errors ❌ (Your mistake)
| Code | Meaning | When It Happens |
|------|---------|-----------------|
| `400` | Bad Request | Invalid data sent |
| `401` | Unauthorized | Missing or invalid authentication |
| `404` | Not Found | Resource doesn't exist |

### 5xx - Server Errors 🔥 (Server's problem)
| Code | Meaning | When It Happens |
|------|---------|-----------------|
| `500` | Internal Server Error | Something broke on the server |

### Examples:

**Success (200):**
```http
GET /users/1
HTTP/1.1 200 OK
{"id": 1, "name": "John"}
```

**Not Found (404):**
```http
GET /users/999
HTTP/1.1 404 Not Found
{"error": "User not found"}
```

**Bad Request (400):**
```http
POST /users
HTTP/1.1 400 Bad Request
{"error": "Name is required"}
```

---

## 5️⃣ JSON Basics (Python dict ↔ JSON)

**JSON** = **JavaScript Object Notation** (but used everywhere, not just JavaScript)

### Python Dictionary → JSON

**Python dict:**
```python
user_data = {
    "id": 1,
    "name": "John",
    "age": 30,
    "skills": ["Python", "JavaScript"],
    "active": True
}
```

**Converts to JSON:**
```json
{
  "id": 1,
  "name": "John",
  "age": 30,
  "skills": ["Python", "JavaScript"],
  "active": true
}
```

### Data Type Conversion:

| Python | JSON |
|--------|------|
| `dict` | `object` |
| `list` | `array` |
| `str` | `string` |
| `int`, `float` | `number` |
| `True` | `true` |
| `False` | `false` |
| `None` | `null` |

### Working with JSON in Python:

```python
import json

# Python dict to JSON string
user_dict = {"name": "John", "age": 30}
json_string = json.dumps(user_dict)
print(json_string)  # {"name": "John", "age": 30}

# JSON string to Python dict
json_data = '{"name": "Alice", "age": 25}'
python_dict = json.loads(json_data)
print(python_dict)  # {'name': 'Alice', 'age': 25}
```

---

## 6️⃣ How API Clients Interact with Endpoints

### What is an API Client?
An **API client** is any tool or program that makes requests to an API endpoint.

### Popular API Clients:

#### 1. **Postman** (GUI Tool)
- User-friendly visual interface
- Great for testing and exploring APIs
- Can save requests for later use

**Steps in Postman:**
1. Choose HTTP method (`GET`, `POST`, etc.)
2. Enter API URL (e.g., `https://api.example.com/users`)
3. Add headers if needed (e.g., `Content-Type: application/json`)
4. Add body data for `POST`/`PUT` requests
5. Click **Send**
6. View response

#### 2. **curl** (Command Line Tool)
Command-line tool for making HTTP requests:

**GET request:**
```bash
curl https://api.github.com/users/octocat
```

**POST request with data:**
```bash
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "age": 30}'
```

**GET with headers:**
```bash
curl -H "Authorization: Bearer your-token" \
     https://api.example.com/protected-data
```

#### 3. **Python with requests library:**
```python
import requests

# GET request
response = requests.get('https://api.github.com/users/octocat')
data = response.json()
print(data['name'])

# POST request
user_data = {"name": "John", "email": "john@example.com"}
response = requests.post('https://httpbin.org/post', json=user_data)
print(response.status_code)  # 200
```

### How Client-Server Communication Works:

```
1. Client sends REQUEST to API endpoint
   ↓
2. Server processes the request
   ↓
3. Server sends RESPONSE back to client
   ↓
4. Client processes the response data
```

### Complete Example:

**API Endpoint:** `https://jsonplaceholder.typicode.com/users/1`

**Using curl:**
```bash
curl https://jsonplaceholder.typicode.com/users/1
```

**Response:**
```json
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
  "address": {
    "street": "Kulas Light",
    "city": "Gwenborough"
  }
}
```

**Using Python:**
```python
import requests

response = requests.get('https://jsonplaceholder.typicode.com/users/1')
user_data = response.json()

print(f"Name: {user_data['name']}")
print(f"Email: {user_data['email']}")
print(f"City: {user_data['address']['city']}")
```

---

## 🎯 Practice APIs for Learning

Try these free APIs to practice:

1. **JSONPlaceholder** → `https://jsonplaceholder.typicode.com/`
   - Fake REST API for testing
   - No authentication needed

2. **GitHub API** → `https://api.github.com/`
   - Real API with lots of data
   - Try: `https://api.github.com/users/YOUR_USERNAME`

3. **HTTPBin** → `https://httpbin.org/`
   - Perfect for testing different HTTP methods
   - Shows you exactly what you sent

---

## ✅ Key Takeaways

1. **API** = Set of rules for applications to communicate
2. **REST** is most common, **SOAP** is for enterprise, **GraphQL** is flexible
3. **HTTP methods** define the action (`GET`=read, `POST`=create, etc.)
4. **Status codes** tell you what happened (`200`=success, `404`=not found, `500`=server error)
5. **JSON** is like Python dictionaries but in text format
6. **API clients** like Postman and curl help you test APIs

**Next:** Start practicing with real APIs using Postman or curl!

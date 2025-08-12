# 📍 FastAPI Practice - Stage 3 Implementation

## Setup & Installation

### Installing FastAPI

```bash
# Install FastAPI and UVICORN server
pip install fastapi uvicorn

# Optional: Install email validation support
pip install "fastapi[standard]"

# For development
pip install python-multipart  # For form data handling
```

### Basic FastAPI Project Structure

```
my_fastapi_project/
├── main.py              # Main application file
├── models.py            # Pydantic models
├── database.py          # Database simulation
├── middleware.py        # Custom middleware
├── requirements.txt     # Dependencies
└── logs/               # Log files directory
```

---

## 1️⃣ Setting up FastAPI Project

### Basic FastAPI Application

**main.py**
```python
from fastapi import FastAPI

# Create FastAPI app instance
app = FastAPI(
    title="My Learning API",
    description="A comprehensive API for learning FastAPI",
    version="1.0.0"
)

@app.get("/")
def read_root():
    return {"message": "Welcome to FastAPI Learning API!"}

@app.get("/health")
def health_check():
    return {"status": "healthy", "message": "API is running"}

# Run with: uvicorn main:app --reload
```

**Run the application:**
```bash
uvicorn main:app --reload
```

**Access your API:**
- API: http://127.0.0.1:8000
- Auto Documentation: http://127.0.0.1:8000/docs
- Alternative Docs: http://127.0.0.1:8000/redoc

---

## 2️⃣ Creating Endpoints

### Simple GET Endpoints

```python
from fastapi import FastAPI

app = FastAPI(title="User Management API")

# In-memory database simulation
users_db = [
    {"id": 1, "name": "John Doe", "email": "john@example.com", "age": 30},
    {"id": 2, "name": "Jane Smith", "email": "jane@example.com", "age": 25},
    {"id": 3, "name": "Bob Johnson", "email": "bob@example.com", "age": 35}
]

@app.get("/users")
def get_all_users():
    """Get all users"""
    return {
        "success": True,
        "data": users_db,
        "total": len(users_db)
    }

@app.get("/users/count")
def get_users_count():
    """Get total number of users"""
    return {"total_users": len(users_db)}

@app.get("/users/names")
def get_user_names():
    """Get only user names"""
    names = [user["name"] for user in users_db]
    return {"names": names}
```

### Path Parameters

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    """Get user by ID"""
    user = next((u for u in users_db if u["id"] == user_id), None)
    
    if not user:
        raise HTTPException(status_code=404, detail=f"User with ID {user_id} not found")
    
    return {
        "success": True,
        "data": user
    }

@app.get("/users/{user_id}/profile")
def get_user_profile(user_id: int):
    """Get user profile information"""
    user = next((u for u in users_db if u["id"] == user_id), None)
    
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Return profile without sensitive information
    profile = {
        "name": user["name"],
        "age": user["age"],
        "profile_complete": bool(user.get("email"))
    }
    
    return {"profile": profile}

# String path parameters
@app.get("/categories/{category_name}")
def get_category(category_name: str):
    """Get category by name"""
    return {
        "category": category_name,
        "description": f"This is the {category_name} category"
    }

# Multiple path parameters
@app.get("/users/{user_id}/posts/{post_id}")
def get_user_post(user_id: int, post_id: int):
    """Get specific post by user"""
    return {
        "user_id": user_id,
        "post_id": post_id,
        "title": f"Post {post_id} by User {user_id}"
    }
```

### Query Parameters

```python
from fastapi import FastAPI, Query
from typing import Optional

app = FastAPI()

@app.get("/users")
def get_users_with_filters(
    limit: int = 10,
    page: int = 1,
    age_min: Optional[int] = None,
    age_max: Optional[int] = None,
    name: Optional[str] = None
):
    """Get users with filtering and pagination"""
    
    # Start with all users
    filtered_users = users_db.copy()
    
    # Apply filters
    if age_min is not None:
        filtered_users = [u for u in filtered_users if u["age"] >= age_min]
    
    if age_max is not None:
        filtered_users = [u for u in filtered_users if u["age"] <= age_max]
    
    if name:
        filtered_users = [u for u in filtered_users if name.lower() in u["name"].lower()]
    
    # Apply pagination
    start = (page - 1) * limit
    end = start + limit
    paginated_users = filtered_users[start:end]
    
    return {
        "data": paginated_users,
        "pagination": {
            "page": page,
            "limit": limit,
            "total": len(filtered_users),
            "total_pages": (len(filtered_users) + limit - 1) // limit
        },
        "filters_applied": {
            "age_min": age_min,
            "age_max": age_max,
            "name": name
        }
    }

# Advanced query parameters with validation
@app.get("/search")
def search_users(
    q: str = Query(..., min_length=2, max_length=50, description="Search query"),
    sort: str = Query("name", regex="^(name|age|email)$", description="Sort field"),
    order: str = Query("asc", regex="^(asc|desc)$", description="Sort order")
):
    """Search users with sorting"""
    
    # Search in names and emails
    results = [
        user for user in users_db 
        if q.lower() in user["name"].lower() or q.lower() in user["email"].lower()
    ]
    
    # Sort results
    reverse = (order == "desc")
    results.sort(key=lambda x: x[sort], reverse=reverse)
    
    return {
        "query": q,
        "sort": f"{sort} {order}",
        "results": results,
        "count": len(results)
    }
```

---

## 3️⃣ Handling HTTP Methods (CRUD Operations)

### Pydantic Models for Request/Response

**models.py**
```python
from pydantic import BaseModel, EmailStr, validator
from typing import Optional
from datetime import datetime

class UserBase(BaseModel):
    name: str
    email: EmailStr
    age: int

class UserCreate(UserBase):
    """Model for creating new users"""
    
    @validator('name')
    def validate_name(cls, v):
        if len(v.strip()) < 2:
            raise ValueError('Name must be at least 2 characters long')
        if len(v) > 100:
            raise ValueError('Name must be less than 100 characters')
        return v.strip().title()  # Capitalize properly
    
    @validator('age')
    def validate_age(cls, v):
        if v < 0:
            raise ValueError('Age cannot be negative')
        if v > 150:
            raise ValueError('Age cannot be more than 150')
        return v

class UserUpdate(BaseModel):
    """Model for updating users"""
    name: Optional[str] = None
    email: Optional[EmailStr] = None
    age: Optional[int] = None
    
    @validator('name')
    def validate_name(cls, v):
        if v is not None:
            if len(v.strip()) < 2:
                raise ValueError('Name must be at least 2 characters long')
            return v.strip().title()
        return v
    
    @validator('age')
    def validate_age(cls, v):
        if v is not None:
            if v < 0 or v > 150:
                raise ValueError('Age must be between 0 and 150')
        return v

class User(UserBase):
    """Model for returning user data"""
    id: int
    created_at: datetime
    
    class Config:
        # This allows the model to work with SQLAlchemy objects
        from_attributes = True

class UserResponse(BaseModel):
    """Standardized response model"""
    success: bool
    message: str
    data: Optional[User] = None
```

### Complete CRUD Implementation

**main.py**
```python
from fastapi import FastAPI, HTTPException, status
from typing import List
from datetime import datetime
from models import User, UserCreate, UserUpdate, UserResponse

app = FastAPI(title="Complete CRUD API", version="1.0.0")

# In-memory database with more realistic data
users_db = [
    {
        "id": 1, 
        "name": "John Doe", 
        "email": "john@example.com", 
        "age": 30,
        "created_at": datetime.now()
    },
    {
        "id": 2, 
        "name": "Jane Smith", 
        "email": "jane@example.com", 
        "age": 25,
        "created_at": datetime.now()
    }
]
next_id = 3

# CREATE - POST Method
@app.post("/users", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
def create_user(user_data: UserCreate):
    """Create a new user"""
    global next_id
    
    # Check if email already exists
    existing_user = next((u for u in users_db if u["email"] == user_data.email), None)
    if existing_user:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Email already registered"
        )
    
    # Create new user
    new_user = {
        "id": next_id,
        "name": user_data.name,
        "email": user_data.email,
        "age": user_data.age,
        "created_at": datetime.now()
    }
    
    users_db.append(new_user)
    next_id += 1
    
    return UserResponse(
        success=True,
        message="User created successfully",
        data=new_user
    )

# READ - GET Methods
@app.get("/users", response_model=List[User])
def get_all_users(limit: int = 10, skip: int = 0):
    """Get all users with pagination"""
    return users_db[skip:skip + limit]

@app.get("/users/{user_id}", response_model=UserResponse)
def get_user(user_id: int):
    """Get user by ID"""
    user = next((u for u in users_db if u["id"] == user_id), None)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with ID {user_id} not found"
        )
    
    return UserResponse(
        success=True,
        message="User found",
        data=user
    )

# UPDATE - PUT Method (Complete replacement)
@app.put("/users/{user_id}", response_model=UserResponse)
def update_user_complete(user_id: int, user_data: UserCreate):
    """Completely replace user data"""
    user = next((u for u in users_db if u["id"] == user_id), None)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with ID {user_id} not found"
        )
    
    # Check if new email conflicts with existing users (except current user)
    existing_user = next((u for u in users_db if u["email"] == user_data.email and u["id"] != user_id), None)
    if existing_user:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Email already registered"
        )
    
    # Update user completely
    user["name"] = user_data.name
    user["email"] = user_data.email
    user["age"] = user_data.age
    
    return UserResponse(
        success=True,
        message="User updated successfully",
        data=user
    )

# UPDATE - PATCH Method (Partial update)
@app.patch("/users/{user_id}", response_model=UserResponse)
def update_user_partial(user_id: int, user_data: UserUpdate):
    """Partially update user data"""
    user = next((u for u in users_db if u["id"] == user_id), None)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with ID {user_id} not found"
        )
    
    # Get only the fields that were provided
    update_data = user_data.dict(exclude_unset=True)
    
    # Check email conflicts if email is being updated
    if "email" in update_data:
        existing_user = next((u for u in users_db if u["email"] == update_data["email"] and u["id"] != user_id), None)
        if existing_user:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Email already registered"
            )
    
    # Update only provided fields
    for field, value in update_data.items():
        user[field] = value
    
    return UserResponse(
        success=True,
        message="User updated successfully",
        data=user
    )

# DELETE Method
@app.delete("/users/{user_id}")
def delete_user(user_id: int):
    """Delete a user"""
    global users_db
    
    user = next((u for u in users_db if u["id"] == user_id), None)
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User with ID {user_id} not found"
        )
    
    # Remove user from database
    users_db = [u for u in users_db if u["id"] != user_id]
    
    return {
        "success": True,
        "message": f"User {user['name']} deleted successfully"
    }

# Bulk operations
@app.post("/users/bulk", response_model=List[User])
def create_multiple_users(users_data: List[UserCreate]):
    """Create multiple users at once"""
    global next_id
    created_users = []
    
    for user_data in users_data:
        # Check if email already exists
        existing_user = next((u for u in users_db if u["email"] == user_data.email), None)
        if existing_user:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail=f"Email {user_data.email} already registered"
            )
        
        new_user = {
            "id": next_id,
            "name": user_data.name,
            "email": user_data.email,
            "age": user_data.age,
            "created_at": datetime.now()
        }
        
        users_db.append(new_user)
        created_users.append(new_user)
        next_id += 1
    
    return created_users

@app.delete("/users")
def delete_all_users():
    """Delete all users (use with caution!)"""
    global users_db
    count = len(users_db)
    users_db = []
    
    return {
        "success": True,
        "message": f"Deleted {count} users"
    }
```

---

## 4️⃣ Request Validation with Pydantic

### Advanced Validation Examples

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, validator, EmailStr, Field
from typing import Optional, List
from datetime import datetime, date
import re

app = FastAPI()

class Address(BaseModel):
    """Nested model for address validation"""
    street: str = Field(..., min_length=5, max_length=100)
    city: str = Field(..., min_length=2, max_length=50)
    postal_code: str = Field(..., regex=r"^\d{5}(-\d{4})?$")
    country: str = Field(default="USA", max_length=3)

class UserProfile(BaseModel):
    """Advanced user model with complex validation"""
    
    # Basic fields with validation
    name: str = Field(..., min_length=2, max_length=50, description="User's full name")
    email: EmailStr = Field(..., description="Valid email address")
    age: int = Field(..., ge=0, le=150, description="Age in years")
    phone: Optional[str] = Field(None, regex=r"^\+?1?\d{9,15}$")
    
    # Date fields
    birth_date: Optional[date] = None
    
    # Nested validation
    address: Optional[Address] = None
    
    # List validation
    interests: List[str] = Field(default=[], max_items=10)
    skills: List[str] = Field(default=[])
    
    # Enum-like validation
    status: str = Field(default="active", regex="^(active|inactive|pending)$")
    
    # Custom validators
    @validator('name')
    def validate_name(cls, v):
        # No numbers in name
        if any(char.isdigit() for char in v):
            raise ValueError('Name cannot contain numbers')
        
        # No special characters except spaces, hyphens, apostrophes
        if not re.match(r"^[a-zA-Z\s\-']+$", v):
            raise ValueError('Name contains invalid characters')
        
        return v.strip().title()
    
    @validator('birth_date')
    def validate_birth_date(cls, v):
        if v and v > date.today():
            raise ValueError('Birth date cannot be in the future')
        return v
    
    @validator('age', always=True)
    def validate_age_with_birth_date(cls, v, values):
        """Cross-field validation: age should match birth_date"""
        birth_date = values.get('birth_date')
        if birth_date and v:
            calculated_age = (date.today() - birth_date).days // 365
            if abs(calculated_age - v) > 1:  # Allow 1 year tolerance
                raise ValueError('Age does not match birth date')
        return v
    
    @validator('interests', 'skills')
    def validate_string_lists(cls, v):
        # Remove empty strings and duplicates
        cleaned = list(set([item.strip() for item in v if item.strip()]))
        return cleaned
    
    @validator('phone')
    def validate_phone(cls, v):
        if v:
            # Remove all non-digit characters for validation
            digits_only = re.sub(r'[^\d]', '', v)
            if len(digits_only) < 10:
                raise ValueError('Phone number too short')
        return v

# Advanced endpoint with validation
@app.post("/profiles", response_model=UserProfile)
def create_user_profile(profile: UserProfile):
    """Create user profile with advanced validation"""
    
    # Additional business logic validation
    if profile.age and profile.age < 13:
        if not profile.address:
            raise HTTPException(
                status_code=422,
                detail="Address required for users under 13"
            )
    
    return profile

# File upload validation
from fastapi import File, UploadFile

class ImageMetadata(BaseModel):
    filename: str
    size: int
    content_type: str
    
    @validator('content_type')
    def validate_content_type(cls, v):
        allowed_types = ['image/jpeg', 'image/png', 'image/gif']
        if v not in allowed_types:
            raise ValueError(f'Content type must be one of: {allowed_types}')
        return v
    
    @validator('size')
    def validate_size(cls, v):
        max_size = 5 * 1024 * 1024  # 5MB
        if v > max_size:
            raise ValueError('File size cannot exceed 5MB')
        return v

@app.post("/upload-image")
async def upload_image(file: UploadFile = File(...)):
    """Upload image with validation"""
    
    # Validate file
    metadata = ImageMetadata(
        filename=file.filename,
        size=len(await file.read()),
        content_type=file.content_type
    )
    
    # Reset file position after reading
    await file.seek(0)
    
    return {
        "message": "File uploaded successfully",
        "metadata": metadata
    }
```

### Custom Error Handling for Validation

```python
from fastapi import FastAPI, Request, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import ValidationError

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    """Custom handler for validation errors"""
    
    errors = []
    for error in exc.errors():
        field = " -> ".join([str(loc) for loc in error["loc"][1:]])  # Skip 'body'
        message = error["msg"]
        error_type = error["type"]
        
        errors.append({
            "field": field,
            "message": message,
            "type": error_type,
            "input": error.get("input")
        })
    
    return JSONResponse(
        status_code=422,
        content={
            "error": "Validation Error",
            "message": "The provided data is invalid",
            "details": errors,
            "total_errors": len(errors)
        }
    )

@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    """Handle custom validation errors"""
    return JSONResponse(
        status_code=400,
        content={
            "error": "Invalid Value",
            "message": str(exc)
        }
    )
```

---

## 5️⃣ Middleware & Logging

### Custom Middleware Implementation

**middleware.py**
```python
from fastapi import Request, Response
from fastapi.middleware.base import BaseHTTPMiddleware
import time
import uuid
import logging
from datetime import datetime

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('logs/api.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger("api_middleware")

class RequestLoggingMiddleware(BaseHTTPMiddleware):
    """Comprehensive request/response logging middleware"""
    
    async def dispatch(self, request: Request, call_next):
        # Generate unique request ID
        request_id = str(uuid.uuid4())[:8]
        start_time = time.time()
        
        # Log request
        logger.info(f"[{request_id}] START - {request.method} {request.url}")
        
        # Add request ID to request state
        request.state.request_id = request_id
        
        # Log request headers (be careful with sensitive data)
        headers_to_log = {
            k: v for k, v in request.headers.items() 
            if k.lower() not in ['authorization', 'cookie']
        }
        logger.info(f"[{request_id}] Headers: {headers_to_log}")
        
        # Log request body for POST/PUT/PATCH
        if request.method in ["POST", "PUT", "PATCH"]:
            try:
                body = await request.body()
                if body:
                    # Don't log if body is too large
                    if len(body) < 1000:
                        logger.info(f"[{request_id}] Body: {body.decode()}")
                    else:
                        logger.info(f"[{request_id}] Body: <large payload {len(body)} bytes>")
            except Exception as e:
                logger.error(f"[{request_id}] Error reading body: {e}")
        
        try:
            # Process the request
            response = await call_next(request)
            
            # Calculate response time
            duration = round((time.time() - start_time) * 1000, 2)
            
            # Log response
            logger.info(f"[{request_id}] END - {response.status_code} - {duration}ms")
            
            # Add custom headers
            response.headers["X-Request-ID"] = request_id
            response.headers["X-Response-Time"] = f"{duration}ms"
            
            return response
            
        except Exception as e:
            duration = round((time.time() - start_time) * 1000, 2)
            logger.error(f"[{request_id}] ERROR - {str(e)} - {duration}ms")
            raise

class SecurityMiddleware(BaseHTTPMiddleware):
    """Basic security middleware"""
    
    async def dispatch(self, request: Request, call_next):
        # Check for required headers
        user_agent = request.headers.get("user-agent")
        if not user_agent:
            logger.warning(f"Request without User-Agent from {request.client.host}")
        
        # Process request
        response = await call_next(request)
        
        # Add security headers
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        
        return response

class PerformanceMiddleware(BaseHTTPMiddleware):
    """Monitor API performance"""
    
    def __init__(self, app):
        super().__init__(app)
        self.slow_request_threshold = 1.0  # 1 second
    
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        
        response = await call_next(request)
        
        duration = time.time() - start_time
        
        # Log slow requests
        if duration > self.slow_request_threshold:
            logger.warning(
                f"SLOW REQUEST: {request.method} {request.url} "
                f"took {duration:.2f}s"
            )
        
        return response
```

### Complete Application with Middleware

**main.py**
```python
from fastapi import FastAPI, Request, Depends
from fastapi.middleware.cors import CORSMiddleware
from middleware import RequestLoggingMiddleware, SecurityMiddleware, PerformanceMiddleware
import logging
import os
from datetime import datetime

# Create logs directory
os.makedirs("logs", exist_ok=True)

# Setup logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('logs/app.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger("main_app")

# Create FastAPI app
app = FastAPI(
    title="Production-Ready API",
    description="API with comprehensive middleware and logging",
    version="1.0.0"
)

# Add CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # In production, specify exact origins
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Add custom middleware (order matters!)
app.add_middleware(RequestLoggingMiddleware)
app.add_middleware(SecurityMiddleware)
app.add_middleware(PerformanceMiddleware)

# Sample data
users_db = [
    {"id": 1, "name": "John Doe", "email": "john@example.com"},
    {"id": 2, "name": "Jane Smith", "email": "jane@example.com"}
]

# Dependency to get request ID
async def get_request_id(request: Request):
    return getattr(request.state, 'request_id', 'unknown')

@app.get("/")
async def root(request_id: str = Depends(get_request_id)):
    logger.info(f"[{request_id}] Root endpoint accessed")
    return {
        "message": "Welcome to Production-Ready API",
        "timestamp": datetime.now(),
        "request_id": request_id
    }

@app.get("/users")
async def get_users(request_id: str = Depends(get_request_id)):
    logger.info(f"[{request_id}] Fetching all users")
    return {
        "users": users_db,
        "count": len(users_db),
        "request_id": request_id
    }

@app.get("/slow-endpoint")
async def slow_endpoint():
    """Simulates a slow endpoint for testing performance monitoring"""
    import asyncio
    await asyncio.sleep(2)  # Sleep for 2 seconds
    return {"message": "This was a slow operation"}

@app.get("/health")
async def health_check():
    """Health check endpoint"""
    return {
        "status": "healthy",
        "timestamp": datetime.now(),
        "version": "1.0.0"
    }

# Error handling
@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception):
    request_id = getattr(request.state, 'request_id', 'unknown')
    logger.error(f"[{request_id}] Unhandled exception: {str(exc)}")
    
    return {
        "error": "Internal server error",
        "message": "Something went wrong",
        "request_id": request_id
    }

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

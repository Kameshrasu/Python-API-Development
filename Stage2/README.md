# 📍 Stage 2: Working with APIs in Python

## 1️⃣ Consuming APIs

### Installing the requests library

First, install the most popular Python library for making HTTP requests:

```bash
pip install requests
```

### Using `requests` library

The `requests` library makes API calls simple and readable:

```python
import requests

# Basic GET request
response = requests.get('https://api.github.com/users/octocat')
print(response.status_code)  # 200
print(response.json())       # JSON data as Python dict
```

### Sending GET Requests

**Basic GET request:**
```python
import requests

# Get user information
response = requests.get('https://jsonplaceholder.typicode.com/users/1')

if response.status_code == 200:
    user_data = response.json()
    print(f"Name: {user_data['name']}")
    print(f"Email: {user_data['email']}")
else:
    print(f"Error: {response.status_code}")
```

**GET with headers:**
```python
import requests

headers = {
    'User-Agent': 'My-App/1.0',
    'Accept': 'application/json'
}

response = requests.get('https://api.github.com/users/octocat', headers=headers)
print(response.json()['name'])
```

### Sending POST Requests

**POST with JSON data:**
```python
import requests

# Data to send
new_user = {
    "name": "John Doe",
    "username": "johndoe",
    "email": "john@example.com"
}

# POST request
response = requests.post(
    'https://jsonplaceholder.typicode.com/users',
    json=new_user  # Automatically sets Content-Type: application/json
)

if response.status_code == 201:
    print("User created successfully!")
    print(response.json())
else:
    print(f"Failed to create user: {response.status_code}")
```

**POST with form data:**
```python
import requests

# Form data
form_data = {
    'username': 'johndoe',
    'password': 'secret123'
}

response = requests.post('https://httpbin.org/post', data=form_data)
print(response.json())
```

### Handling Query Parameters

**Method 1: In URL string**
```python
import requests

url = 'https://jsonplaceholder.typicode.com/posts?userId=1&_limit=5'
response = requests.get(url)
posts = response.json()
print(f"Found {len(posts)} posts")
```

**Method 2: Using params parameter (Recommended)**
```python
import requests

params = {
    'userId': 1,
    '_limit': 5,
    '_sort': 'title'
}

response = requests.get('https://jsonplaceholder.typicode.com/posts', params=params)
posts = response.json()

for post in posts:
    print(f"- {post['title']}")
```

### Working with JSON Payloads

**Sending complex JSON data:**
```python
import requests

# Complex data structure
blog_post = {
    "title": "My First API Post",
    "body": "This is the content of my blog post",
    "userId": 1,
    "tags": ["python", "api", "tutorial"],
    "metadata": {
        "category": "programming",
        "difficulty": "beginner"
    }
}

response = requests.post(
    'https://jsonplaceholder.typicode.com/posts',
    json=blog_post
)

if response.status_code == 201:
    created_post = response.json()
    print(f"Post created with ID: {created_post['id']}")
```

### Working with API Authentication

#### 1. API Keys

**API Key in headers:**
```python
import requests

headers = {
    'X-API-Key': 'your-api-key-here',
    'Content-Type': 'application/json'
}

response = requests.get('https://api.example.com/data', headers=headers)
```

**API Key in query parameters:**
```python
import requests

params = {
    'api_key': 'your-api-key-here',
    'format': 'json'
}

response = requests.get('https://api.example.com/data', params=params)
```

#### 2. Bearer Tokens (JWT)

```python
import requests

# Token (usually obtained from login)
token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

headers = {
    'Authorization': f'Bearer {token}',
    'Content-Type': 'application/json'
}

response = requests.get('https://api.example.com/protected-data', headers=headers)

if response.status_code == 200:
    data = response.json()
    print("Protected data accessed successfully!")
else:
    print(f"Authentication failed: {response.status_code}")
```

#### 3. Basic Authentication

```python
import requests
from requests.auth import HTTPBasicAuth

# Method 1: Using auth parameter
response = requests.get(
    'https://api.example.com/data',
    auth=HTTPBasicAuth('username', 'password')
)

# Method 2: Using auth shorthand
response = requests.get(
    'https://api.example.com/data',
    auth=('username', 'password')
)
```

---

## 2️⃣ Parsing API Responses

### Reading JSON Data in Python

**Basic JSON parsing:**
```python
import requests

response = requests.get('https://jsonplaceholder.typicode.com/users/1')

# Get JSON data
user_data = response.json()

# Access data like a Python dictionary
print(f"Name: {user_data['name']}")
print(f"Email: {user_data['email']}")
print(f"Company: {user_data['company']['name']}")

# Access nested data safely
phone = user_data.get('phone', 'Not available')
print(f"Phone: {phone}")
```

**Working with arrays:**
```python
import requests

response = requests.get('https://jsonplaceholder.typicode.com/users')
users = response.json()

# Loop through users
for user in users:
    print(f"{user['id']}: {user['name']} ({user['email']})")

# Filter users
developers = [user for user in users if 'dev' in user.get('website', '').lower()]
print(f"Found {len(developers)} developers")
```

**Extracting specific data:**
```python
import requests

response = requests.get('https://jsonplaceholder.typicode.com/posts')
posts = response.json()

# Extract just titles
titles = [post['title'] for post in posts]

# Get posts by specific user
user_posts = [post for post in posts if post['userId'] == 1]

print(f"Total posts: {len(posts)}")
print(f"Posts by user 1: {len(user_posts)}")
```

### Handling Errors and Exceptions

**Check status codes:**
```python
import requests

def get_user(user_id):
    response = requests.get(f'https://jsonplaceholder.typicode.com/users/{user_id}')
    
    if response.status_code == 200:
        return response.json()
    elif response.status_code == 404:
        print(f"User {user_id} not found")
        return None
    else:
        print(f"Error: {response.status_code}")
        return None

# Usage
user = get_user(1)
if user:
    print(f"User found: {user['name']}")
```

**Handle network exceptions:**
```python
import requests
from requests.exceptions import RequestException, Timeout, ConnectionError

def safe_api_call(url):
    try:
        response = requests.get(url, timeout=5)  # 5 second timeout
        response.raise_for_status()  # Raises exception for 4xx/5xx status codes
        return response.json()
    
    except ConnectionError:
        print("Error: Unable to connect to the API")
        return None
    
    except Timeout:
        print("Error: Request timed out")
        return None
    
    except requests.exceptions.HTTPError as e:
        print(f"HTTP Error: {e}")
        return None
    
    except RequestException as e:
        print(f"Request failed: {e}")
        return None
    
    except ValueError:  # JSON decode error
        print("Error: Invalid JSON response")
        return None

# Usage
data = safe_api_call('https://jsonplaceholder.typicode.com/users/1')
if data:
    print(f"User: {data['name']}")
```

**Complete error handling example:**
```python
import requests
import json

def api_request(url, method='GET', data=None, headers=None):
    """
    Make API request with comprehensive error handling
    """
    try:
        if method.upper() == 'GET':
            response = requests.get(url, headers=headers, timeout=10)
        elif method.upper() == 'POST':
            response = requests.post(url, json=data, headers=headers, timeout=10)
        else:
            raise ValueError(f"Unsupported method: {method}")
        
        # Check if request was successful
        response.raise_for_status()
        
        return {
            'success': True,
            'data': response.json(),
            'status_code': response.status_code
        }
    
    except requests.exceptions.Timeout:
        return {'success': False, 'error': 'Request timeout'}
    
    except requests.exceptions.ConnectionError:
        return {'success': False, 'error': 'Connection error'}
    
    except requests.exceptions.HTTPError as e:
        return {
            'success': False,
            'error': f'HTTP error: {e}',
            'status_code': response.status_code
        }
    
    except json.JSONDecodeError:
        return {'success': False, 'error': 'Invalid JSON response'}
    
    except Exception as e:
        return {'success': False, 'error': f'Unexpected error: {str(e)}'}

# Usage
result = api_request('https://jsonplaceholder.typicode.com/users/1')

if result['success']:
    user_data = result['data']
    print(f"User: {user_data['name']}")
else:
    print(f"API call failed: {result['error']}")
```

### Logging API Responses

**Basic logging:**
```python
import requests
import logging

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

def log_api_call(url, method='GET'):
    logger.info(f"Making {method} request to: {url}")
    
    response = requests.get(url)
    
    logger.info(f"Response status: {response.status_code}")
    logger.info(f"Response size: {len(response.content)} bytes")
    
    if response.status_code == 200:
        logger.info("Request successful")
        return response.json()
    else:
        logger.error(f"Request failed with status {response.status_code}")
        return None

# Usage
data = log_api_call('https://jsonplaceholder.typicode.com/users/1')
```

**Advanced logging with response details:**
```python
import requests
import logging
import json
from datetime import datetime

# Setup logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger('api_client')

class APIClient:
    def __init__(self):
        self.session = requests.Session()
        
    def make_request(self, url, method='GET', **kwargs):
        start_time = datetime.now()
        
        logger.info(f"Starting {method} request to {url}")
        
        try:
            response = self.session.request(method, url, **kwargs)
            end_time = datetime.now()
            duration = (end_time - start_time).total_seconds()
            
            logger.info(f"Request completed in {duration:.2f}s")
            logger.info(f"Status: {response.status_code}")
            
            if response.status_code >= 400:
                logger.error(f"Error response: {response.text}")
            
            return response
            
        except Exception as e:
            logger.error(f"Request failed: {str(e)}")
            raise

# Usage
client = APIClient()
response = client.make_request('https://jsonplaceholder.typicode.com/users/1')
```

---

## 3️⃣ Real API Practice

### GitHub API

**Get user information:**
```python
import requests

def get_github_user(username):
    url = f'https://api.github.com/users/{username}'
    
    response = requests.get(url)
    
    if response.status_code == 200:
        user = response.json()
        
        print(f"Name: {user['name']}")
        print(f"Bio: {user.get('bio', 'No bio available')}")
        print(f"Public Repos: {user['public_repos']}")
        print(f"Followers: {user['followers']}")
        print(f"Following: {user['following']}")
        
        return user
    else:
        print(f"User '{username}' not found")
        return None

# Usage
get_github_user('octocat')
```

**Get user repositories:**
```python
import requests

def get_user_repos(username, per_page=5):
    url = f'https://api.github.com/users/{username}/repos'
    
    params = {
        'sort': 'updated',
        'per_page': per_page
    }
    
    response = requests.get(url, params=params)
    
    if response.status_code == 200:
        repos = response.json()
        
        print(f"\n{username}'s repositories:")
        for repo in repos:
            print(f"- {repo['name']}")
            print(f"  Description: {repo.get('description', 'No description')}")
            print(f"  Language: {repo.get('language', 'Unknown')}")
            print(f"  Stars: {repo['stargazers_count']}")
            print()
    else:
        print(f"Could not fetch repositories for {username}")

# Usage
get_user_repos('octocat')
```

### OpenWeatherMap API

First, sign up at [OpenWeatherMap](https://openweathermap.org/api) to get a free API key.

**Get current weather:**
```python
import requests

def get_weather(city, api_key):
    url = 'https://api.openweathermap.org/data/2.5/weather'
    
    params = {
        'q': city,
        'appid': api_key,
        'units': 'metric'  # Celsius
    }
    
    response = requests.get(url, params=params)
    
    if response.status_code == 200:
        weather = response.json()
        
        print(f"Weather in {weather['name']}, {weather['sys']['country']}:")
        print(f"Temperature: {weather['main']['temp']}°C")
        print(f"Feels like: {weather['main']['feels_like']}°C")
        print(f"Description: {weather['weather'][0]['description']}")
        print(f"Humidity: {weather['main']['humidity']}%")
        print(f"Wind Speed: {weather['wind']['speed']} m/s")
        
        return weather
    else:
        print(f"Could not get weather for {city}")
        print(f"Error: {response.json().get('message', 'Unknown error')}")
        return None

# Usage (replace with your API key)
api_key = 'your-openweathermap-api-key'
get_weather('London', api_key)
```

**Get 5-day forecast:**
```python
import requests
from datetime import datetime

def get_forecast(city, api_key):
    url = 'https://api.openweathermap.org/data/2.5/forecast'
    
    params = {
        'q': city,
        'appid': api_key,
        'units': 'metric'
    }
    
    response = requests.get(url, params=params)
    
    if response.status_code == 200:
        forecast = response.json()
        
        print(f"5-day forecast for {forecast['city']['name']}:")
        
        for item in forecast['list'][:5]:  # First 5 forecasts
            dt = datetime.fromtimestamp(item['dt'])
            temp = item['main']['temp']
            desc = item['weather'][0]['description']
            
            print(f"{dt.strftime('%Y-%m-%d %H:%M')}: {temp}°C, {desc}")
    else:
        print(f"Could not get forecast for {city}")

# Usage
get_forecast('Paris', api_key)
```

### Public Dummy APIs (jsonplaceholder.typicode.com)

**Complete CRUD operations:**
```python
import requests

BASE_URL = 'https://jsonplaceholder.typicode.com'

class PostManager:
    def __init__(self):
        self.base_url = BASE_URL
    
    def get_all_posts(self):
        """Get all posts"""
        response = requests.get(f'{self.base_url}/posts')
        
        if response.status_code == 200:
            return response.json()
        return []
    
    def get_post(self, post_id):
        """Get single post"""
        response = requests.get(f'{self.base_url}/posts/{post_id}')
        
        if response.status_code == 200:
            return response.json()
        return None
    
    def create_post(self, title, body, user_id):
        """Create new post"""
        new_post = {
            'title': title,
            'body': body,
            'userId': user_id
        }
        
        response = requests.post(f'{self.base_url}/posts', json=new_post)
        
        if response.status_code == 201:
            return response.json()
        return None
    
    def update_post(self, post_id, title, body, user_id):
        """Update existing post"""
        updated_post = {
            'id': post_id,
            'title': title,
            'body': body,
            'userId': user_id
        }
        
        response = requests.put(f'{self.base_url}/posts/{post_id}', json=updated_post)
        
        if response.status_code == 200:
            return response.json()
        return None
    
    def delete_post(self, post_id):
        """Delete post"""
        response = requests.delete(f'{self.base_url}/posts/{post_id}')
        
        return response.status_code == 200

# Usage example
post_manager = PostManager()

# Get all posts
posts = post_manager.get_all_posts()
print(f"Total posts: {len(posts)}")

# Get single post
post = post_manager.get_post(1)
if post:
    print(f"Post title: {post['title']}")

# Create new post
new_post = post_manager.create_post(
    title="My Python API Post",
    body="This post was created using Python!",
    user_id=1
)
if new_post:
    print(f"Created post with ID: {new_post['id']}")

# Update post
updated_post = post_manager.update_post(
    post_id=1,
    title="Updated Title",
    body="Updated content",
    user_id=1
)

# Delete post
if post_manager.delete_post(1):
    print("Post deleted successfully")
```

**Working with related data:**
```python
import requests

def get_user_posts_and_comments():
    """Get user info, their posts, and comments on posts"""
    
    # Get user info
    user_response = requests.get('https://jsonplaceholder.typicode.com/users/1')
    user = user_response.json()
    print(f"User: {user['name']} ({user['email']})")
    
    # Get user's posts
    posts_response = requests.get('https://jsonplaceholder.typicode.com/posts', 
                                  params={'userId': 1})
    posts = posts_response.json()
    print(f"\nPosts by {user['name']} ({len(posts)} posts):")
    
    for post in posts[:3]:  # Show first 3 posts
        print(f"\n- {post['title']}")
        
        # Get comments for this post
        comments_response = requests.get(
            f'https://jsonplaceholder.typicode.com/posts/{post["id"]}/comments'
        )
        comments = comments_response.json()
        
        print(f"  Comments ({len(comments)}):")
        for comment in comments[:2]:  # Show first 2 comments
            print(f"    * {comment['name']}: {comment['body'][:50]}...")

# Usage
get_user_posts_and_comments()
```

---

## 🎯 Practice Exercises

Try these exercises to practice what you've learned:

1. **GitHub User Analyzer**
   - Get user info and repositories
   - Find their most popular repository
   - List programming languages they use

2. **Weather Dashboard**
   - Get weather for multiple cities
   - Compare temperatures
   - Get weather alerts if temperature is extreme

3. **Blog Post Manager**
   - Use JSONPlaceholder API
   - Create a simple blog post CRUD system
   - Add error handling and logging

4. **API Response Cache**
   - Cache API responses to avoid repeated requests
   - Implement expiry time for cache
   - Compare performance with and without cache

---

## ✅ Key Takeaways

1. **`requests` library** makes API calls simple in Python
2. **Always handle errors** - network issues, HTTP errors, JSON parsing errors
3. **Use proper authentication** - API keys, Bearer tokens, Basic auth
4. **Log your API calls** for debugging and monitoring
5. **Practice with real APIs** to understand different response formats
6. **Handle different HTTP methods** - GET for reading, POST for creating

**Next Stage:** Building your own APIs with Flask/FastAPI! 🚀

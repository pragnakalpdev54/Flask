# Level 0: Complete Beginner - Python & Web Fundamentals for Flask

> **Prerequisites**: None. This is where your journey begins!
> **Next Level**: [Level 1: Foundations](02_LEVEL_1_FOUNDATIONS.md)

> [!NOTE]
> **No OOP Required**: You do *not* need to know about Python Classes or Objects to start this level. We use simple functions.

## Goal

Before diving into Flask, you need to understand the fundamentals. This level covers Python basics for web development, web concepts, and environment setup. By the end, you'll be ready to start building your first Flask API.

## Table of Contents

1. [Why Learn This?](#why-learn-this)
2. [Python Basics for Web Development](#python-basics-for-web-development)
3. [Understanding the Web](#understanding-the-web)
4. [What is an API?](#what-is-an-api)
5. [Environment Setup Explained](#environment-setup-explained)
6. [Hello World: Your First Flask API](#hello-world-your-first-flask-api)
7. [Practice Problems](#practice-problems)
8. [Trivia](#trivia)

## Why Learn This?

### What is Flask?

**Flask** is a lightweight WSGI web application framework. It is designed to make getting started quick and easy, with the ability to scale up to complex applications. It began as a simple wrapper around Werkzeug and Jinja and has become one of the most popular Python web application frameworks.

### Why Use Flask?

- **Flexible**: It gives you control over how you structure your application.
- **Minimalistic**: Core is simple and extensible.
- **Great for APIs**: Perfect for building RESTful services (our focus).
- **Industry Standard**: Used by companies like Netflix, Lyft, and Reddit.

## Python Basics for Web Development

### Variables and JSON Structures

**Why this matters**: We will be building APIs that speak JSON. Python dictionaries are the closest relative to JSON.

```python
# Dictionaries - The core of our API responses
user = {
    "id": 1,
    "username": "dev_user",
    "is_active": True,
    "roles": ["admin", "editor"]
}

# Lists - Collections of resources
users = [
    {"id": 1, "username": "dev_user"},
    {"id": 2, "username": "test_user"}
]
```

### Functions as Endpoints

**Why this matters**: In Flask, a "route" is just a function decorated with a URL path.

```python
# A normal function
def get_user_data(user_id):
    return {"id": user_id, "name": "Alice"}

# In Flask, it looks very similar:
# @app.route("/users/<int:user_id>")
# def get_user(user_id):
#     return {"id": user_id, "name": "Alice"}
```

## Understanding the Web

### Client-Server Model

```
┌─────────┐         HTTP Request          ┌─────────┐
│ Client  │ ────────────────────────────> │ Server │
│(Browser)│                                │(Flask) │
└─────────┘ <────────────────────────────  └─────────┘
            HTTP Response (JSON)
```

**Key Concept**: In this course, we are building **APIs**. We will not be sending HTML pages (templates). We will send **JSON data**. The Client (Front-end or Mobile App) will take that JSON and display it.

## What is an API?

**API (Application Programming Interface)** is the messenger that takes requests and tells a system what you want to do and then returns the response back to you.

### REST API Basics

- **Resource**: The "thing" you are interacting with (e.g., `User`, `Product`).
- **Endpoint**: The URL where the resource lives (e.g., `/api/users`).
- **Method**: The action (GET, POST, PUT, DELETE).

## Environment Setup Explained

### Step 1: Create Project Folder

```bash
mkdir flask_learning
cd flask_learning
```

### Step 2: Create Virtual Environment

Always use a virtual environment to isolate dependencies.

**Windows**:

```bash
python -m venv venv
venv\Scripts\activate
```

**Linux/Mac**:

```bash
python3 -m venv venv
source venv/bin/activate
```

### Step 3: Install Flask

```bash
pip install flask
```

## Hello World: Your First Flask API

Let's create a minimal Flask application that returns JSON.

1. Create a file named `app.py`.
2. Add the following code:

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/")
def home():
    """
    Root endpoint.
    Returns a simple JSON message.
    """
    return jsonify({
        "message": "Welcome to the Flask API!",
        "status": "success"
    })

if __name__ == "__main__":
    app.run(debug=True)
```

> [!TIP]
> **Magic Box – Flask & `jsonify`**: You do not need to understand how `Flask`, `@app.route`, or `jsonify` work internally yet. Treat this pattern as a reusable starting point and focus on the JSON you return.

## Understanding Flask Basics

Before we run the application, let's understand what's happening in those last two lines.

### Why `if __name__ == "__main__":`?

This is a standard Python pattern. Here's what it means:

```python
if __name__ == "__main__":
    app.run(debug=True)
```

**Explanation:**

- `__name__` is a special Python variable. When you run a file directly (e.g., `python app.py`), Python sets `__name__` to `"__main__"`.
- If you **import** this file from another file (e.g., `from app import app`), then `__name__` will be the module name (`"app"`), not `"__main__"`.

**Why does this matter?**

We only want to start the Flask development server when we run the file directly, **not** when we import it. This is important because:

1. **Testing**: When you write tests, you import your app without running the server.
2. **Production**: Production servers (like Gunicorn or uWSGI) import your app and run it their own way.

**In simple terms:** This line says "Only start the development server if this file is being run directly."

### Understanding `app.run()` Parameters

The `app.run()` method starts Flask's built-in development server. Here are the most important parameters:

```python
app.run(
    host='127.0.0.1',  # What network interface to bind to
    port=5000,         # What port to listen on
    debug=True         # Enable/disable debug mode
)
```

#### 1. `host` - Network Interface

**Default:** `'127.0.0.1'` (localhost only)

```python
# Only accessible from your own computer
app.run(host='127.0.0.1')  # or just 'localhost'

# Accessible from other devices on your network (e.g., testing on your phone)
app.run(host='0.0.0.0')
```

> [!WARNING]
> **Security**: Only use `host='0.0.0.0'` in development on trusted networks. Never in production!

**When to use each:**

- `127.0.0.1` (default): Development on your own machine only
- `0.0.0.0`: Testing from other devices (phone, tablet) on your local network

#### 2. `port` - Port Number

**Default:** `5000`

```python
# Use default port 5000
app.run()  # Accessible at http://127.0.0.1:5000

# Use custom port
app.run(port=8080)  # Accessible at http://127.0.0.1:8080
```

**Why change the port?**

- Port 5000 is already in use by another application
- Your company/project has a standard port number
- You're running multiple Flask apps simultaneously

#### 3. `debug` - Debug Mode

**Default:** `False` (disabled)

```python
# Development: Enable debug mode
app.run(debug=True)

# Production: NEVER enable debug mode
app.run(debug=False)
```

**What does `debug=True` do?**

| Feature | Behavior |
|---------|----------|
| **Auto-reload** | Server restarts automatically when you change code |
| **Error Details** | Shows detailed error pages with stack traces in browser |
| **Interactive Debugger** | Allows you to inspect variables in the browser when errors occur |

> [!CAUTION]
> **NEVER use `debug=True` in production!** It exposes sensitive information and allows code execution through the browser debugger. This is a major security risk.

### Complete Example with All Parameters

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/")
def home():
    return jsonify({"message": "Hello, Flask!"})

if __name__ == "__main__":
    # Development configuration
    app.run(
        host='127.0.0.1',  # Localhost only
        port=5000,          # Default Flask port
        debug=True          # Enable auto-reload and detailed errors
    )
```

### Development vs Production

**Development (what we use in this course):**

```python
if __name__ == "__main__":
    app.run(debug=True)
```

**Production (real-world deployment):**

```bash
# Use a production server like Gunicorn
gunicorn -w 4 -b 0.0.0.0:8000 app:app
```

> [!NOTE]
> Flask's built-in server (`app.run()`) is **only for development**. For production, you need a proper WSGI server like Gunicorn, uWSGI, or Waitress. We'll cover this in later levels.

### Run it

```bash
python app.py
```

### Test it

Open your browser or Postman and go to `http://127.0.0.1:5000/`.

**You should see:**

```json
{
  "message": "Welcome to the Flask API!",
  "status": "success"
}
```

> **Note**: Notice we used `jsonify`. This is a helper that converts Python dictionaries to proper JSON responses with the correct Content-Type header (`application/json`).  
> **Rule**: Always return JSON. Never return raw strings or HTML for APIs.

## Practice Problems

### Problem 1: Setup

1. Create a new virtual environment.
2. Install Flask.
3. Freeze requirements to `requirements.txt`.

### Problem 2: New Endpoint

1. Add a new route `/ping` to your `app.py`.
2. It should return `{"pong": true}`.

## Trivia

### Question 1

**What function do we use to return JSON in Flask?**

- [ ] `json.dumps()`
- [x] `jsonify()`
- [ ] `return_json()`
- [ ] `flask.json()`

**Explanation**: `jsonify` serializes data to JSON and sets the generic Content-Type header to `application/json`.

### Question 2

**Why do we not use HTML templates in this course?**

- [ ] HTML is too hard
- [x] We are building an API that serves data (JSON) to any client (Web, Mobile, etc), decoupling the UI from the Logic.
- [ ] Flask cannot handle HTML

### Question 3

**What decorates a function to make it a route?**

- [ ] `@app.endpoint`
- [ ] `@flask.route`
- [x] `@app.route`
- [ ] `@url_path`

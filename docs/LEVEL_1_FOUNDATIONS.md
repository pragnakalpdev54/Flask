# Level 1: Flask Foundations - Routing & Requests

## Goal

In this level, we will learn how to handle different types of requests, work with data sent by clients, and organize our application using Blueprints.

**Core Rules for this Level:**

1. **API First**: We only speak JSON.
2. **Thin Routes**: Routes should only parse input and return output.
3. **No Jinja**: We serve static HTML/JS if we need a UI.

## Table of Contents

1. [Routing Basics](#routing-basics)
2. [Handling Requests (The Input)](#handling-requests-the-input)
3. [Building Responses (The Output)](#building-responses-the-output)
4. [Structuring with Blueprints](#structuring-with-blueprints)
5. [Serving Static Files](#serving-static-files)
6. [Practice Problems](#practice-problems)

## Routing Basics

### Methods

By default, a route only answers to GET. You must specify others.

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route("/books", methods=["GET"])
def get_books():
    return jsonify({"message": "Get all books"})

@app.route("/books", methods=["POST"])
def create_book():
    return jsonify({"message": "Create a book"}), 201
```

### Dynamic URLs

Capture values from the URL.

```python
@app.route("/books/<int:book_id>", methods=["GET"])
def get_book(book_id):
    # book_id is passed as an integer argument
    return jsonify({"id": book_id, "title": "Sample Book"})
```

*Converters*: `string` (default), `int`, `float`, `path`, `uuid`.

## Handling Requests (The Input)

In an API, we mostly care about **JSON Body** and **Query Parameters**.

### 1. JSON Body (POST/PUT)

Used when creating or updating resources.

```python
@app.route("/users", methods=["POST"])
def create_user():
    # request.get_json() parses the JSON body
    data = request.get_json()
    
    username = data.get("username")
    email = data.get("email")
    
    # In real app: Validate and Save to DB
    
    return jsonify({"status": "created", "username": username}), 201
```

### 2. Query Parameters (GET)

Used for filtering or pagination (e.g., `/users?page=2&role=admin`).

```python
@app.route("/users", methods=["GET"])
def list_users():
    # request.args is a dictionary-like object
    page = request.args.get("page", 1, type=int)
    role = request.args.get("role")
    
    return jsonify({"page": page, "filter_role": role})
```

## Building Responses (The Output)

Always return `jsonify` and a proper Status Code.

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Standard success |
| 201 | Created | resource created successfully |
| 400 | Bad Request | Client sent invalid JSON or missing fields |
| 404 | Not Found | Resource does not exist |
| 500 | Server Error | Something broke (Hide details from user!) |

```python
@app.route("/orders", methods=["POST"])
def create_order():
    data = request.get_json()
    
    if not data or "item" not in data:
        # 400 Bad Request
        return jsonify({"error": "Missing 'item' field"}), 400
        
    return jsonify({"id": 123, "status": "pending"}), 201
```

## Structuring with Blueprints

As your app grows, you cannot keep everything in `app.py`. **Blueprints** allow you to organize your routes into separate files (modules).

### Directory Structure

```
project/
├── app.py
└── routes/
    ├── __init__.py
    └── user_routes.py
```

### 1. Create a Blueprint (`routes/user_routes.py`)

```python
from flask import Blueprint, jsonify

# Define the blueprint
user_bp = Blueprint("users", __name__, url_prefix="/users")

@user_bp.route("/", methods=["GET"])
def get_users():
    return jsonify({"users": []})

@user_bp.route("/<int:id>", methods=["GET"])
def get_user(id):
    return jsonify({"id": id})
```

### 2. Register it in `app.py`

```python
from flask import Flask
from routes.user_routes import user_bp

app = Flask(__name__)

# Register the blueprint
app.register_blueprint(user_bp)
# Now routes are accessible at /users and /users/<id>

if __name__ == "__main__":
    app.run(debug=True)
```

## Serving Static Files

Since we **do not use Jinja templates**, we handle UI by serving static HTML/JS files that consume our JSON API.

Place files in a `static` folder:

```
project/
├── static/
│   ├── index.html
│   └── app.js
└── app.py
```

### Serving the Entry Point

```python
@app.route("/")
def index():
    # Send the static index.html file
    return app.send_static_file("index.html")
```

Your `index.html` should use JavaScript (`fetch` or `axios`) to call your API endpoints (e.g., `/users`) and display data.

## Practice Problems

### Problem 1: Calculator API

Create an endpoint `/calculate` that accepts a POST request with JSON:
`{"operation": "add", "x": 10, "y": 5}`
And returns: `{"result": 15}`.
Support add, subtract, multiply, divide. Return 400 for invalid operations.

### Problem 2: Blueprint Organization

Refactor Problem 1 into a blueprint structure with a URL prefix `/math`.

### Problem 3: Query Params

Create a route `/search` that takes a query param `q`. If `q` is missing, return error 400. If present, return `{"query": q}`.

## Trivia

### Question 1

**How do you access the JSON body sent by a client?**

- [ ] `request.body`
- [x] `request.get_json()`
- [ ] `request.json_data`
- [ ] `json.load(request)`

### Question 2

**What is the purpose of a Blueprint?**

- [ ] To design the database
- [ ] To create HTML templates
- [x] To organize routes into modular components
- [ ] To handle authentication

### Question 3

**If a client sends invalid data, what status code should you return?**

- [ ] 200 OK
- [ ] 500 Server Error
- [x] 400 Bad Request
- [ ] 404 Not Found

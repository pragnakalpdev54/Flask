# Level 1: Flask Foundations - Routing & Requests

> **Prerequisites**: [Level 0: Beginner](01_LEVEL_0_BEGINNER.md)
> **Next Step**: [Test Task 1](03_TEST_TASK_1.md)

> [!NOTE]
> **No OOP Required**: We still use standard Python functions here. No classes needed!

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
4. [Serving Static Files](#serving-static-files)
5. [Pagination, Filtering, and Versioning](#pagination-filtering-and-versioning)
6. [Practice Problems](#practice-problems)

## Routing Basics

### 1. HTTP & API Basics

Before writing code, let's understand the request types and status codes we handle.

#### HTTP Methods

* **GET**: Retrieve data (Read). Browsers send this by default. **Never change data with a GET.**
* **POST**: Create new data (Create). Used for submitting forms or JSON.
* **PUT**: Update an existing resource completely (Replace).
* **PATCH**: Update parts of an existing resource (Modify).
* **DELETE**: Remove data (Delete).

#### HTTP Status Codes (The Response)

Always return the correct status code so the client knows what happened.

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Standard success |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Client sent invalid JSON or missing fields |
| 404 | Not Found | Resource does not exist |
| 500 | Server Error | Something broke (Hide details from user!) |

### 2. How to Test (The Browser Trap)

> [!WARNING]
> **You cannot test POST requests in your browser address bar!**

When you type a URL in Chrome or Firefox, it **always** sends a `GET` request. If you try to visit a `POST`-only route, you will get a `405 Method Not Allowed` error.

**Tools you need:**

1. **Postman** (Recommended): A GUI app to send any type of HTTP request.
2. **cURL**: A command-line tool. Example: `curl -X POST http://localhost:5000/books`

### 3. Defining Routes

By default, a route only answers to GET. You must explicitly specify other methods.

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

> [!TIP]
> **Magic Box – Flask routing & `request`**: Treat `Flask(__name__)`, `@app.route`, `jsonify(...)`, and the `request` object as Magic Boxes. You mainly need to know *where* to read data (`request.get_json()`, `request.args`) and *what* JSON to return.

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

### 3. The `request` Object Summary

Confused? Here is a cheat sheet:

| Data Source | Content-Type | Usage | Flask Accessor |
|-------------|--------------|-------|----------------|
| **JSON** | `application/json` | Modern APIs (POST/PUT) | `request.get_json()` |
| **URL Params** | N/A | Filtering/Sorting (GET) | `request.args.get('key')` |
| **Form Data** | `application/x-www-form-urlencoded` | HTML Forms | `request.form.get('key')` |
| **Path Vars** | N/A | Resource IDs | Function Argument |

### 4. Query Parameters (GET)

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

Always return `jsonify` and a proper Status Code (see the table in [HTTP & API Basics](#1-http--api-basics)).

```python
@app.route("/orders", methods=["POST"])
def create_order():
    data = request.get_json()
    
    if not data or "item" not in data:
        # 400 Bad Request
        return jsonify({"error": "Missing 'item' field"}), 400
        
    return jsonify({"id": 123, "status": "pending"}), 201
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

## Pagination, Filtering, and Versioning

At some point your API will outgrow simple “return all rows” endpoints. You need a consistent way to **page**, **filter**, and **version** your APIs.

### 1. Basic pagination pattern

Use `page` and `page_size` (or `limit` and `offset`) as query parameters:

```python
@app.route("/items", methods=["GET"])
def list_items():
    page = request.args.get("page", 1, type=int)
    page_size = request.args.get("page_size", 20, type=int)

    # pretend this comes from your DB
    items = [
        {"id": 1, "name": "Item 1"},
        {"id": 2, "name": "Item 2"},
    ]

    return jsonify({
        "items": items,
        "page": page,
        "page_size": page_size,
        "total": len(items),
    })
```

Recommended response envelope:

* `items`: list of resources
* `page`: current page
* `page_size`: how many per page
* `total`: total number of items

### 2. Filtering and searching

Reuse query params for filters instead of inventing custom URLs:

* `/users?role=admin`
* `/books?author_id=1&price_min=10&price_max=50`

Example:

```python
@app.route("/books", methods=["GET"])
def list_books():
    author_id = request.args.get("author_id", type=int)
    price_min = request.args.get("price_min", type=float)
    price_max = request.args.get("price_max", type=float)

    return jsonify({
        "author_id": author_id,
        "price_min": price_min,
        "price_max": price_max,
    })
```

### 3. API versioning

When you need to introduce **breaking changes**, do not silently change existing URLs. Add a version prefix:

* `/api/v1/books`
* `/api/v2/books`

Later levels and Test Task 3 use prefixes like `/api/v1/authors` and `/api/v1/books`. You should follow the same idea in your own projects.

## Practice Problems

### Problem 1: Calculator API

Create an endpoint `/calculate` that accepts a POST request with JSON:
`{"operation": "add", "x": 10, "y": 5}`
And returns: `{"result": 15}`.
Support add, subtract, multiply, divide. Return 400 for invalid operations.

### Problem 3: Query Params

Create a route `/search` that takes a query param `q`. If `q` is missing, return error 400. If present, return `{"query": q}`.

## Trivia

### Question 1

**How do you access the JSON body sent by a client?**

* [ ] `request.body`
* [x] `request.get_json()`
* [ ] `request.json_data`
* [ ] `json.load(request)`

### Question 3

**If a client sends invalid data, what status code should you return?**

* [ ] 200 OK
* [ ] 500 Server Error
* [x] 400 Bad Request
* [ ] 404 Not Found

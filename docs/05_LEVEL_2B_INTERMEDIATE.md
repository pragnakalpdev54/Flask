# Level 2B: Intermediate - Serialization & Error Handling

> **Prerequisites**: [Level 2A: Databases](04_LEVEL_2A_INTERMEDIATE.md)
> **Next Level**: [Level 3: Authentication](06_LEVEL_3_AUTHENTICATION.md)

> [!NOTE]
> **OOP Level**: Low. Schemas are classes, but they are just **templates for data shapes**. You declare fields, and Marshmallow does the magic. You do not need to understand advanced OOP concepts to use them.

## Goal

In this level, we make our API robust. We will use **Marshmallow** for validation (incoming data) and serialization (outgoing data). We will also strictly implement **Global Error Handling** so users never see standard internal server errors.

**Strict Rules:**

1. **Never return Model objects directly**: Always serialize them to JSON.
2. **Never expose Stack Traces**: Catch all errors and return generic JSON messages.
3. **Validate Inputs**: Trust no one.

## Table of Contents

1. [Why Marshmallow?](#why-marshmallow)
2. [Defining Schemas](#defining-schemas)
3. [Validation (Input)](#validation-input)
4. [Serialization (Output)](#serialization-output)
5. [Global Error Handling](#global-error-handling)
6. [Practice Problems](#practice-problems)

## Why Marshmallow?

SQLAlchemy Models are Python Objects. APIs need JSON. Marshmallow bridges this gap.

- **Serialization**: Object -> JSON
- **Deserialization**: JSON -> Object (with Validation!)

Install it:

```bash
pip install flask-marshmallow marshmallow-sqlalchemy
```

## Defining Schemas

Schemas usually sit near your models or in a `schemas/` folder.

```python
from flask_marshmallow import Marshmallow
from marshmallow import ValidationError
from models import User

ma = Marshmallow()

class UserSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User
        load_instance = True  # Optional: deserialize to model instances
        exclude = ("password_hash",)  # Never expose password hashes

# Initialize schemas
user_schema = UserSchema()
users_schema = UserSchema(many=True)
```

> [!TIP]
> **Magic Box – Schemas & `ma`**: Treat `ma = Marshmallow()` and `class UserSchema(...)` as Magic Boxes that know how to turn your models into JSON and validate input. Focus on which fields you declare and what errors you return, not on how Marshmallow works internally.

## Validation (Input)

Use the schema to validate incoming JSON before it reaches your Service Layer.

```python
@app.route('/users', methods=['POST'])
def create_user():
    json_data = request.get_json()
    
    try:
        # Validate data
        data = user_schema.load(json_data)
    except ValidationError as err:
        return jsonify(err.messages), 400
        
    # Proceed to service...
    user = UserService.create_user(
        username=data["username"],
        email=data["email"],
        password_hash=data["password_hash"],
    )
```

## Serialization (Output)

Never manually build dictionaries like `{'id': user.id, ...}`. Use the schema.

```python
@app.route('/users', methods=['GET'])
def get_users():
    users = UserService.get_all_users()
    
    # Dump list of users
    result = users_schema.dump(users)
    return jsonify(result), 200

@app.route('/users/<int:id>', methods=['GET'])
def get_user(id):
    user = UserService.get_user_by_id(id)
    
    # Dump single user
    result = user_schema.dump(user)
    return jsonify(result), 200
```

## Global Error Handling

**Rule**: User should not be able to see the internal server errors on UI.

Instead of `try-except` in every route, use `errorhandler`.

### 1. Create a Custom Exception

```python
# errors.py

class AppError(Exception):
    def __init__(self, message, status_code=400):
        super().__init__(message)
        self.message = message
        self.status_code = status_code
```

### 2. Update Service to raise it

```python
# services/user_service.py
from errors import AppError

def get_user_by_id(user_id):
    user = User.query.get(user_id)
    if not user:
        raise AppError("User not found", 404)
    return user
```

### 3. Catch it Globally in `app.py`

```python
from errors import AppError

def create_app():
    app = Flask(__name__)
    # ... config ...

    @app.errorhandler(AppError)
    def handle_app_error(e):
        return jsonify({
            "error": e.message,
            "code": e.status_code
        }), e.status_code

    @app.errorhandler(500)
    def handle_500(e):
        # Log the real error to a file/logging service
        # app.logger.error(e)
        return jsonify({"error": "Internal Server Error"}), 500

    @app.errorhandler(404)
    def handle_404(e):
        return jsonify({"error": "Resource not found"}), 404
        
    return app
```

**Result**: Even if your code crashes, the user sees `{"error": "Internal Server Error"}` (JSON), not a scary HTML stack trace.

## Practice Problems

### Problem 1: Book Schema

Create a `BookSchema` for your Book model. Configure it to auto-generate from the SQLAlchemy model.

### Problem 2: Serialize User List

Create an endpoint `/api/users` that returns all users as JSON using Marshmallow.

**Starter code**:

```python
from flask import Flask, jsonify
from models import User
from schemas import UserSchema

app = Flask(__name__)

user_schema = UserSchema()
users_schema = UserSchema(many=True)

@app.route('/api/users', methods=['GET'])
def get_users():
    users = User.query.all()
    # Serialize the users
    result = users_schema.load(users)  # Is this correct?
    return jsonify(result), 200
```

### Problem 3: Validate Implementation

Update your `create_book` route to use `book_schema.load()`. Try sending invalid data (missing title) and confirm you get a 400 JSON response with validation errors.

### Problem 4: 404 Handler

Implement a global 404 handler that returns `{"error": "This endpoint does not exist"}`. Use Postman to trigger it.

## Common Pitfalls Quiz

Test your understanding of serialization and validation:

### Pitfall 1: Exposing Sensitive Fields

**Question**: What's wrong with this schema?

```python
from marshmallow import Schema, fields

class UserSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User
        load_instance = True
        # No exclusions!
```

<details>
<summary>Click to see answer</summary>

**Answer**: This exposes **password_hash** in API responses! Anyone can see hashed passwords.

**Fix**:

```python
class UserSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User
        load_instance = True
        exclude = ("password_hash",)  # Never expose passwords!
```

**Critical security rule**: NEVER expose:

- `password_hash`
- `password`
- API keys
- Tokens
- Internal IDs you don't want users to know

</details>

### Pitfall 2: Confusing load() vs dump()

**Question**: Why does this crash?

```python
@app.route('/users', methods=['GET'])
def get_users():
    users = User.query.all()
    result = user_schema.load(users)  # Wrong method!
    return jsonify(result), 200
```

<details>
<summary>Click to see answer</summary>

**Answer**: `load()` is for **input** (JSON → Object), `dump()` is for **output** (Object → JSON). You're trying to load when you should dump!

**Fix**:

```python
@app.route('/users', methods=['GET'])
def get_users():
    users = User.query.all()
    result = users_schema.dump(users)  # dump = serialize for output
    return jsonify(result), 200
```

**Remember**:

- `schema.load(json_data)` - Validate input, deserialize
- `schema.dump(model_object)` - Serialize for output

</details>

### Pitfall 3: Not Catching ValidationError

**Question**: What happens when a user sends invalid data?

```python
from marshmallow import ValidationError

@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    validated = user_schema.load(data)  # What if validation fails?
    # ... create user ...
    return jsonify({"message": "Created"}), 201
```

<details>
<summary>Click to see answer</summary>

**Answer**: If validation fails, Marshmallow raises `ValidationError`, which crashes your app with a 500 error instead of returning proper 400.

**Fix**:

```python
from marshmallow import ValidationError

@ app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    
    try:
        validated = user_schema.load(data)
    except ValidationError as err:
        return jsonify(err.messages), 400  # Return validation errors
    
    # ... create user ...
    return jsonify({"message": "Created"}), 201
```

**Better with global handler**:

```python
@app.errorhandler(ValidationError)
def handle_validation_error(e):
    return jsonify(e.messages), 400
```

</details>

### Pitfall 4: Exposing Internal Errors

**Question**: What's wrong with this error handler?

```python
@app.errorhandler(500)
def handle_500(e):
    return jsonify({"error": str(e)}), 500  # Exposing internal details!
```

<details>
<summary>Click to see answer</summary>

**Answer**: **NEVER expose internal errors to users!** This leaks:

- Database schema details
- File paths
- Stack traces
- Library versions

Attackers use this information!

**Fix**:

```python
@app.errorhandler(500)
def handle_500(e):
    # Log internally
    app.logger.error(f"Internal error: {e}")
    
    # Return generic message to user
    return jsonify({"error": "Internal server error"}), 500
```

</details>

### Pitfall 5: Wrong Schema for Different Operations

**Question**: How do you handle different requirements for registration vs login?

```python
# Registration needs: username, email, password
# Login needs: email, password (no username)
# Using same schema for both?

class UserSchema(ma.Schema):
    username = fields.String(required=True)
    email = fields.String(required=True)
    password = fields.String(required=True)
```

<details>
<summary>Click to see answer</summary>

**Answer**: Use **different schemas** for different operations!

**Fix**:

```python
class UserRegistrationSchema(ma.Schema):
    username = fields.String(required=True)
    email = fields.String(required=True)
    password = fields.String(required=True)

class UserLoginSchema(ma.Schema):
    email = fields.String(required=True)
    password = fields.String(required=True)
    # No username needed for login

class UserResponseSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User
        exclude = ("password_hash",)  # Never return password

# Use appropriate schema
@app.route('/register', methods=['POST'])
def register():
    data = UserRegistrationSchema().load(request.get_json())
    # ...

@app.route('/login', methods=['POST'])
def login():
    data = UserLoginSchema().load(request.get_json())
    # ...
```

</details>

## Trivia

### Question 1

**What is the method to convert an Object to JSON using Marshmallow?**

- [ ] `schema.load()`
- [x] `schema.dump()`
- [ ] `schema.json()`
- [ ] `schema.serialize()`

### Question 2

**What allows us to catch exceptions globally?**

- [ ] `@app.route`
- [ ] `@app.exception`
- [x] `@app.errorhandler`
- [ ] `try...except` block in `main`

### Question 3

**If a validation error occurs during `schema.load()`, what exception is raised?**

- [ ] `ValueError`
- [x] `ValidationError`
- [ ] `TypeError`
- [ ] `JsonError`

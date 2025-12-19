# Level 3: Authentication & Security

> **Prerequisites**: [Level 2B: Serialization](05_LEVEL_2B_INTERMEDIATE.md)
> **Next Step**: [Test Task 2](07_TEST_TASK_2.md)

> [!NOTE]
> **OOP Level**: Medium. We use a class for `AuthService` as a **namespace for auth functions**, and decorators like `@jwt_required` which are advanced functions (not OOP). The complexity in this level is **security**, not object-oriented design.

## Goal

In this level, we secure our API. Since we are building a stateless JSON API, we will use **JWT (JSON Web Tokens)** instead of Session-based auth (Flask-Login). We will also implement a strict **Authentication Service**.

**Strict Rules:**

1. **No Sessions**: Use JWTs for API authentication.
2. **Passwords**: Never store plain passwords. Use Bcrypt.
3. **Auth Service**: Crypto and Token logic belongs in a service, not the route.

## Table of Contents

1. [Password Hashing](#password-hashing)
2. [JWT Basics](#jwt-basics)
3. [The Auth Service](#the-auth-service)
4. [Protecting Routes](#protecting-routes)
5. [Practice Problems](#practice-problems)

## Password Hashing

We need `flask-bcrypt` to hash passwords.

### Setup

```bash
pip install flask-bcrypt
```

In `extensions.py`:

```python
from flask_bcrypt import Bcrypt
bcrypt = Bcrypt()
```

### Usage

```python
# Hashing (Registration)
pw_hash = bcrypt.generate_password_hash('s3cret').decode('utf-8')

# Checking (Login)
is_valid = bcrypt.check_password_hash(pw_hash, 's3cret')
```

> [!TIP]
> **Magic Box – Password Hashing (`bcrypt`)**: `bcrypt.generate_password_hash(...)` and `bcrypt.check_password_hash(...)` are crypto Magic Boxes. You only need to know *when* to call them (on registration and login), not how hashing works internally.

## JWT Basics

We use `flask-jwt-extended` for handling tokens.

### Setup

```bash
pip install flask-jwt-extended
```

In `extensions.py`:

```python
from flask_jwt_extended import JWTManager
jwt = JWTManager()
```

In `app.py` (Factory):

```python
def create_app():
    # ...
    app.config["JWT_SECRET_KEY"] = "super-secret-key"  # Change this!
    jwt.init_app(app)
    # ...
```

> [!TIP]
> **Magic Box – JWTs**: `JWTManager`, `create_access_token`, and `@jwt_required()` are Magic Boxes that handle token creation and verification. Focus on which routes require a token and what payload you store in it, not on the cryptography details.

## The Auth Service

We must isolate authentication logic.

### `services/auth_service.py`

```python
from models import User, db
from extensions import bcrypt
from flask_jwt_extended import create_access_token
from errors import AppError

class AuthService:
    @staticmethod
    def register_user(username, email, password):
        # 1. Check if user exists
        if User.query.filter_by(email=email).first():
            raise AppError("Email already exists", 400)
            
        # 2. Hash password
        password_hash = bcrypt.generate_password_hash(password).decode('utf-8')
        
        # 3. Create User
        user = User(username=username, email=email, password_hash=password_hash)
        db.session.add(user)
        db.session.commit()
        
        return user

    @staticmethod
    def login_user(email, password):
        # 1. Find User
        user = User.query.filter_by(email=email).first()
        
        # 2. Check Password
        if not user or not bcrypt.check_password_hash(user.password_hash, password):
            raise AppError("Invalid credentials", 401)
            
        # 3. Generate Token
        access_token = create_access_token(identity=user.id)
        return {"access_token": access_token, "user_id": user.id}
```

## Protecting Routes

Now we use the **Auth Service** in our routes and protect endpoints.

### `routes/auth_routes.py`

```python
from flask import Blueprint, request, jsonify
from flask_jwt_extended import jwt_required, get_jwt_identity
from services.auth_service import AuthService
from services.user_service import UserService

auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    # Call Service
    user = AuthService.register_user(
        data['username'], data['email'], data['password']
    )
    return jsonify({"message": "User registered"}), 201

@auth_bp.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    # Call Service
    result = AuthService.login_user(data['email'], data['password'])
    return jsonify(result), 200
```

### Protecting an Endpoint (`jwt_required`)

```python
@auth_bp.route('/me', methods=['GET'])
@jwt_required()
def get_me():
    # Get ID from token
    current_user_id = get_jwt_identity()
    
    # Use UserService to get details
    user = UserService.get_user_by_id(current_user_id)
    
    return jsonify({"username": user.username, "email": user.email})
```

## Practice Problems

### Problem 1: Integration

Install `flask-bcrypt` and `flask-jwt-extended`. Update `app.py` to initialize them.

### Problem 2: Login Flow

Implement the `login` route using the `AuthService`. Test it with Postman:

1. Register a user (writes to DB with hashed PW).
2. Login with correct password -> Get Token.
3. Login with wrong password -> Get 401 Error.

### Problem 3: Protected Route

Create a route `/secret` that returns `{"message": "Top Secret"}`. Add `@jwt_required()`. Try accessing it without a token (401) and with a token (200).

## Trivia

### Question 1

**Why do we use JWTs instead of Sessions for APIs?**

- [x] JWTs are stateless and work well with mobile/frontend apps on potential separate domains.
- [ ] Sessions are less secure.
- [ ] JWTs are encrypted (They are signed, not encrypted by default!).
- [ ] Flask doesn't support sessions.

### Question 2

**What function generates the hash for a password?**

- [ ] `bcrypt.check_password_hash`
- [x] `bcrypt.generate_password_hash`
- [ ] `jwt.create_token`
- [ ] `hashlib.sha256`

### Question 3

**How do you access the user ID inside a protected route?**

- [ ] `request.user_id`
- [x] `get_jwt_identity()`
- [ ] `jwt.get_id()`
- [ ] `current_user.id` (This is for Flask-Login)

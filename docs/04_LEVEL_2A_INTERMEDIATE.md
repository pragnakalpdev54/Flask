# Level 2A: Intermediate - Databases & The Service Layer

> **Prerequisites**: [Test Task 1](03_TEST_TASK_1.md)
> **Next Level**: [Level 2B: Serialization](05_LEVEL_2B_INTERMEDIATE.md)

> [!NOTE]
> **OOP Level**: Low. We define classes for database models and services, but they are mostly **configuration** and **namespaces for functions**. You do **not** need to understand constructors, inheritance, or polymorphism to follow this level.

## Goal

In this level, we connect our Flask application to a Relational Database using **Flask-SQLAlchemy**. Crucially, we will learn the **Service Layer Pattern** to keep our code clean, scalable, and professional.

**Strict Rules:**

1. **Thin Routes**: Routes never access the database directly.
2. **Service Layer**: All business logic and DB calls reside in services.
3. **ORM Only**: Use SQLAlchemy models, not raw SQL.

## Table of Contents

1. [Setting Up Flask-SQLAlchemy](#setting-up-flask-sqlalchemy)
2. [Defining Models](#defining-models)
3. [Database Migrations](#database-migrations)
4. [The Service Layer Pattern](#the-service-layer-pattern)
5. [Refactoring: The Right Way](#refactoring-the-right-way)
6. [Practice Problems](#practice-problems)

## Setting Up Flask-SQLAlchemy

Install dependencies:

```bash
pip install flask-sqlalchemy flask-migrate psycopg2-binary
```

*(We use psycopg2 for PostgreSQL, but strict ORM usage means we can switch easily)*

### Configuration

In `app.py` or `extensions.py` (better structure):

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

db = SQLAlchemy()
migrate = Migrate()

def create_app():
    app = Flask(__name__)
    # Configure your DB URI
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'  # or postgresql://...

    db.init_app(app)
    migrate.init_app(app, db)

    return app
```

> [!TIP]
> **Magic Box – `db` and `migrate`**: `db = SQLAlchemy()` and `migrate = Migrate()` create Magic Boxes that handle the low-level database work and migrations for you. You mainly need to import them from `extensions.py` and run `flask db migrate` / `flask db upgrade` when models change.

## Defining Models

Models are Python classes that map to database tables.

```python
# models.py
from extensions import db

class User(db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(255), nullable=False)
    is_active = db.Column(db.Boolean, default=True)

    def __repr__(self):
        return f'<User {self.username}>'
```

## Database Migrations

### Why Migrations?

In a production app, you cannot just delete `app.db` and recreate it when you change a model. You need to **alter** the existing database (e.g., add a column) without losing data. **Alembic** (handled by Flask-Migrate) does this for us.

### The Workflow

1. **Initialize** (One time only):

    ```bash
    flask db init
    ```

    *Creates a `migrations/` folder. Commit this to Git!*

2. **Migrate** (Every time you change models):

    ```bash
    flask db migrate -m "Added phone column to user"
    ```

    *Generates a version file in `migrations/versions/`. Review this file! It contains exactly what changes will be applied.*

3. **Upgrade** (Apply to Database):

    ```bash
    flask db upgrade
    ```

    *Runs the version file against your actual database.*

## The Service Layer Pattern

This is the most important architectural rule in our project.

### 🧠 Concept: Classes as Namespaces

We use `class UserService`. **Don't panic.** We are not creating objects (`user_service = UserService()`).

We are just using the class as a **Namespace** (like a folder) to group related functions together.

* `UserService.create_user()`
* `UserService.get_all_users()`

It keeps our code organized. Pure functions would work too, but this is the Flask/Python industry standard.

You can think of it as two equivalent styles:

```python
# Pure functions (no classes)
def create_user(username, email):
    ...

def get_all_users():
    ...
```

```python
# Same idea, grouped in a class-as-namespace
class UserService:
    @staticmethod
    def create_user(username, email):
        ...

    @staticmethod
    def get_all_users():
        ...
```

In this course we mostly use the **second style** because it scales better in real projects, but you do **not** need to learn constructors, inheritance, or polymorphism. If you ever feel stuck, revisit the OOP FAQ in the main **Flask Learning Guide** and remember that classes here are just organized folders for related functions.

> [!TIP]
> If you see confusing errors around `self` or class methods, double-check that your service methods are marked with `@staticmethod` and that you are calling them like `UserService.create_user(...)` (without creating an instance).

### ❌ The Bad Way (Fat Routes)

**Do NOT do this:**

```python
# BAD CODE - DO NOT USE
@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    # Business logic in route!
    if User.query.filter_by(email=data['email']).first():
        return jsonify({"error": "Email exists"}), 400
    
    # DB access in route!
    new_user = User(username=data['username'], email=data['email'])
    db.session.add(new_user)
    db.session.commit()
    
    return jsonify({"id": new_user.id}), 201
```

### ✅ The Good Way (Thin Routes + Service Layer)

**1. Create the Service (`services/user_service.py`)**

```python
from models import User, db
from errors import AppError

class UserService:
    @staticmethod
    def create_user(username, email, password_hash):
        """
        Handles business logic for creating a user.
        """
        # Validation Logic
        if User.query.filter_by(email=email).first():
            raise AppError("Email already exists", 400)
            
        # DB Logic
        new_user = User(username=username, email=email, password_hash=password_hash)
        db.session.add(new_user)
        db.session.commit()
        
        return new_user

    @staticmethod
    def get_all_users():
        return User.query.all()
```

**2. Call it from the Route (`routes/user_routes.py`)**

```python
from flask import Blueprint, request, jsonify
from werkzeug.security import generate_password_hash
from services.user_service import UserService

user_bp = Blueprint('users', __name__)

@user_bp.route('/', methods=['POST'])
def create_user():
    data = request.get_json()
    
    password_hash = generate_password_hash(data['password'])

    # Route just delegates to Service
    user = UserService.create_user(
        username=data['username'],
        email=data['email'],
        password_hash=password_hash,
    )
    return jsonify({"id": user.id, "message": "Created"}), 201
```

### Why?

1. **Reusability**: You can call `UserService.create_user` from a CLI command or a test or another service.
2. **Testing**: You can test `UserService` without faking HTTP requests.
3. **Maintainability**: Routes stay simple and clean.

## Practice Problems

### Problem 1: Book Model

Create a `Book` model with `title`, `author`, and `isbn` (unique).
Run migrations.

### Problem 2: Book Service

Create a `BookService` with methods:

* `add_book(title, author, isbn)`
* `get_books_by_author(author)`

Ensure `add_book` raises an error if ISBN already exists.

### Problem 3: Refactor

Refactor your Calculator practice from Level 1 to use a `CalculationService`.

## Trivia

### Question 1

**Where should database queries reside?**

* [ ] In the Route function
* [ ] In the Template
* [x] In the Service Layer
* [ ] In the `app.py`

### Question 2

**What command applies changes to the database?**

* [ ] `flask db init`
* [ ] `flask db migrate`
* [x] `flask db upgrade`
* [ ] `flask run`

### Question 3

**Why do we avoid "Fat Routes"?**

* [ ] They take up too much disk space
* [x] They mix business logic with HTTP logic, making code hard to test and reuse
* [ ] Flask forbids them

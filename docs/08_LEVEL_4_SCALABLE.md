# Level 4: Scalable Structure & The Application Factory

> **Prerequisites**: [Test Task 2](07_TEST_TASK_2.md)
> **Next Level**: [Level 5: Extensions](09_LEVEL_5_EXTENSIONS.md)

> [!NOTE]
> **OOP Level**: Medium. The "Application Factory" (`create_app`) is a function that creates an object (`app`). You still do not write new classes here—this level is about **project structure and imports**, not advanced OOP.

## Goal

In this level, we graduate from "Scripts" to "Applications". We will learn the **Application Factory Pattern** to allow running multiple instances (dev, test, prod). Most importantly, we will solve the #1 beginner nightmare: **Circular Imports**.

## Table of Contents

1. [Introduction to Blueprints](#introduction-to-blueprints)
2. [The Circular Import Problem](#the-circular-import-problem)
3. [The Application Factory Pattern](#the-application-factory-pattern)
4. [Project Directory Structure](#project-directory-structure)
5. [Managing Extensions](#managing-extensions)
6. [Configuration Management](#configuration-management)
7. [Practice Problems](#practice-problems)

## Introduction to Blueprints

Before we can build scalable apps, we need to stop writing everything in one file. **Blueprints** allow you to organize your routes into separate files (modules).

### Directory Structure

```text
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

> [!TIP]
> **Magic Box – Blueprints**: `Blueprint(...)` and `app.register_blueprint(...)` are wiring Magic Boxes. Treat them as a standard pattern for grouping routes and attaching them to the app.

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

## The Circular Import Problem

### The Scenario

1. `app.py` imports `db` from `models.py`
2. `models.py` imports `app` from `app.py` (to use config)
3. **CRASH**: `ImportError: cannot import name...`

### The Solution

We must decouple the **Extension Creation** from the **App Creation**.

## Managing Extensions

Create a separate file `extensions.py`. This file instantiates plugins *without* attaching them to an app.

```python
# extensions.py
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_jwt_extended import JWTManager
from flask_marshmallow import Marshmallow

# Unbound extensions
db = SQLAlchemy()
migrate = Migrate()
jwt = JWTManager()
ma = Marshmallow()
```

> [!TIP]
> **Magic Box – Extensions Module**: `extensions.py` centralizes Magic Boxes like `db`, `migrate`, `jwt`, and `ma`. You import them where needed and call `.init_app(app)`, without worrying about how they create connections under the hood.

Now `models.py` can import `db` from `extensions.py`. `extensions.py` imports nothing. **No cycles!**

## The Application Factory Pattern

Instead of creating a global `app` object, we put creation logic in a function `create_app()`.

### `app.py` (or `application/__init__.py`)

```python
from flask import Flask
from extensions import db, migrate, jwt, ma
from routes.user_routes import user_bp
from routes.auth_routes import auth_bp
from app_config import Config

def create_app(config_class=Config):
    app = Flask(__name__)
    
    # 1. Load Configuration
    app.config.from_object(config_class)
    
    # 2. Init Extensions
    db.init_app(app)
    migrate.init_app(app, db)
    jwt.init_app(app)
    ma.init_app(app)
    
    # 3. Register Blueprints
    app.register_blueprint(user_bp)
    app.register_blueprint(auth_bp)
    
    return app
```

## Project Directory Structure

A professional flask project looks like this:

```
project_root/
├── app/                  # Application Package
│   ├── __init__.py       # Contains create_app()
│   ├── extensions.py     # All extensions (db, jwt, etc)
│   ├── models/           # DB Models
│   │   └── user.py
│   ├── routes/           # Blueprints
│   │   ├── __init__.py
│   │   └── user_routes.py
│   ├── schemas/          # Marshmallow Schemas
│   │   └── user_schema.py
│   └── services/         # Business Logic
│       └── user_service.py
├── config.py             # Settings (Dev/Prod)
├── wsgi.py               # Entry point for Server
└── requirements.txt
```

### `wsgi.py` (The Entry Point)

This is what your server (Gunicorn/uWSGI) runs.

```python
from app import create_app

application = create_app()

if __name__ == "__main__":
    application.run(debug=True)
```

## Configuration Management

Avoid hardcoding. Use classes for config.

### `config.py`

```python
import os

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY', 'default-dev-key')
    SQLALCHEMY_TRACK_MODIFICATIONS = False

class DevConfig(Config):
    SQLALCHEMY_DATABASE_URI = 'sqlite:///dev.db'
    DEBUG = True

class ProdConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')
    DEBUG = False
```

Then pass it to the factory: `create_app(DevConfig)`

## Practice Problems

### Problem 1: Refactor to Factory

Take your code from Level 3.

1. Create `extensions.py`.
2. Move global `app` var to `create_app` in `app/__init__.py`.
3. Create `wsgi.py`.
4. Run it!

### Problem 2: Circular Import Fix

Intentionally create a circular import (e.g., import `app` inside `extensions.py`). Observe the error. Then fix it by following the pattern above.

### Problem 3: Environment Config

Create a `TestConfig` that uses an in-memory database (`sqlite:///:memory:`). Update `create_app` to use it when running tests.

## Trivia

### Question 1

**Why do we use `extensions.py`?**

- [ ] To make the code look prettier
- [x] To avoid circular imports by creating extensions unbound to the app
- [ ] Because Flask requires it
- [ ] To store database passwords

### Question 2

**What does `init_app` do?**

- [ ] Deletes the database
- [ ] Creates a new app
- [x] Binds an extension instance (like `db`) to a specific Flask app instance
- [ ] Installs the library

### Question 3

**Which file is the entry point for production servers (like Gunicorn)?**

- [ ] `routes.py`
- [ ] `extensions.py`
- [x] `wsgi.py`
- [ ] `config.py`

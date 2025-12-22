# Flask Pattern Cheatsheet

Quick reference for common Flask patterns. Copy-paste these as starting points.

## Table of Contents

1. [Service Layer Pattern](#service-layer-pattern)
2. [Extensions Pattern](#extensions-pattern)
3. [Model Relationships](#model-relationships)
4. [Blueprint Registration](#blueprint-registration)
5. [Migration Workflow](#migration-workflow)
6. [Error Handling](#error-handling)
7. [Common Imports](#common-imports)

---

## Service Layer Pattern

You can use **either** class-based or function-based services. Both are correct!

### Option 1: Class-Based (Industry Standard)

```python
# services/user_service.py
from models import User
from extensions import db

class UserService:
    @staticmethod
    def create_user(username, email, password_hash):
        """Create a new user."""
        # Check if exists
        if User.query.filter_by(email=email).first():
            return None, "Email already exists"
        
        # Create and save
        user = User(username=username, email=email, password_hash=password_hash)
        db.session.add(user)
        db.session.commit()
        
        return user, None
    
    @staticmethod
    def get_user_by_id(user_id):
        """Get user by ID."""
        user = User.query.get(user_id)
        if not user:
            return None, "User not found"
        return user, None
    
    @staticmethod
    def get_all_users():
        """Get all users."""
        return User.query.all()
```

**Using class-based service in routes:**

```python
# app.py
from flask import request, jsonify
from services.user_service import UserService

@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    
    # Call service method
    user, error = UserService.create_user(
        username=data['username'],
        email=data['email'],
        password_hash=data['password']
    )
    
    if error:
        return jsonify({"error": error}), 400
    
    return jsonify({"id": user.id, "username": user.username}), 201

@app.route('/users', methods=['GET'])
def get_users():
    users = UserService.get_all_users()
    return jsonify({"users": [{"id": u.id, "username": u.username} for u in users]}), 200
```

### Option 2: Function-Based (Simple Alternative)

```python
# services/user_functions.py
from models import User
from extensions import db

def create_user(username, email, password_hash):
    """Create a new user."""
    # Check if exists
    if User.query.filter_by(email=email).first():
        return None, "Email already exists"
    
    # Create and save
    user = User(username=username, email=email, password_hash=password_hash)
    db.session.add(user)
    db.session.commit()
    
    return user, None

def get_user_by_id(user_id):
    """Get user by ID."""
    user = User.query.get(user_id)
    if not user:
        return None, "User not found"
    return user, None

def get_all_users():
    """Get all users."""
    return User.query.all()
```

**Using function-based service in routes:**

```python
# app.py
from flask import request, jsonify
from services.user_functions import create_user, get_user_by_id, get_all_users

@app.route('/users', methods=['POST'])
def create_user_endpoint():
    data = request.get_json()
    
    # Call service function
    user, error = create_user(
        username=data['username'],
        email=data['email'],
        password_hash=data['password']
    )
    
    if error:
        return jsonify({"error": error}), 400
    
    return jsonify({"id": user.id, "username": user.username}), 201

@app.route('/users', methods=['GET'])
def get_users_endpoint():
    users = get_all_users()
    return jsonify({"users": [{"id": u.id, "username": u.username} for u in users]}), 200
```

**Which to choose?**

- **Class-based**: Better for larger projects, groups related functions
- **Function-based**: Simpler, less to learn, works perfectly fine
- Use whichever you're comfortable with!

---

## Extensions Pattern

### extensions.py

```python
# extensions.py
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_jwt_extended import JWTManager
from flask_marshmallow import Marshmallow
from flask_bcrypt import Bcrypt

# Create unbound extensions
db = SQLAlchemy()
migrate = Migrate()
jwt = JWTManager()
ma = Marshmallow()
bcrypt = Bcrypt()
```

### Initialize in App Factory

```python
# app.py or app/__init__.py
from flask import Flask
from extensions import db, migrate, jwt, ma, bcrypt

def create_app():
    app = Flask(__name__)
    
    # Config
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    app.config['JWT_SECRET_KEY'] = 'your-secret-key'
    
    # Initialize extensions
    db.init_app(app)
    migrate.init_app(app, db)
    jwt.init_app(app)
    ma.init_app(app)
    bcrypt.init_app(app)
    
    # Import models (AFTER init, INSIDE function)
    from models import User
    
    return app

if __name__ == '__main__':
    app = create_app()
    app.run(debug=True)
```

---

## Model Relationships

### One-to-Many (Author → Books)

```python
# models.py
from extensions import db

class Author(db.Model):
    __tablename__ = 'authors'
    
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), unique=True, nullable=False)
    bio = db.Column(db.String(500))
    
    # One-to-Many relationship
    books = db.relationship('Book', backref='author', cascade='all, delete-orphan')

class Book(db.Model):
    __tablename__ = 'books'
    
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    price = db.Column(db.Float, nullable=False)
    
    # Foreign Key
    author_id = db.Column(db.Integer, db.ForeignKey('authors.id'), nullable=False)
```

### Manual Serialization (No Schemas Needed)

```python
# In route
@app.route('/authors/<int:id>', methods=['GET'])
def get_author(id):
    author = Author.query.get_or_404(id)
    
    return jsonify({
        "id": author.id,
        "name": author.name,
        "bio": author.bio,
        "books": [
            {"id": b.id, "title": b.title, "price": b.price} 
            for b in author.books
        ]
    }), 200
```

---

## Blueprint Registration

### Creating a Blueprint

```python
# routes/user_routes.py
from flask import Blueprint, jsonify, request

user_bp = Blueprint('users', __name__, url_prefix='/users')

@user_bp.route('/', methods=['GET'])
def get_users():
    return jsonify({"users": []})

@user_bp.route('/<int:id>', methods=['GET'])
def get_user(id):
    return jsonify({"id": id})
```

### Registering in App

```python
# app.py or app/__init__.py
from flask import Flask
from routes.user_routes import user_bp
from routes.auth_routes import auth_bp

def create_app():
    app = Flask(__name__)
    
    # Register blueprints
    app.register_blueprint(user_bp)
    app.register_blueprint(auth_bp)
    
    return app
```

---

## Migration Workflow

### Initial Setup (ONCE per project)

```bash
# Initialize migrations folder
flask db init
```

### When You Change Models

```bash
# 1. Modify your model in models.py
# 2. Generate migration
flask db migrate -m "Add phone number to users"

# 3. Apply migration
flask db upgrade
```

### Common Commands

```bash
# Check current migration version
flask db current

# Downgrade one version (rollback)
flask db downgrade

# View migration history
flask db history
```

### Troubleshooting

```bash
# "No changes detected" - Check:
# 1. Did you import model in app.py?
# 2. Did you save models.py?

# Nuclear option (development only!)
rm -rf migrations/
rm app.db
flask db init
flask db migrate -m "Initial migration"
flask db upgrade
```

---

## Error Handling

### Custom Exception

```python
# errors.py
class AppError(Exception):
    def __init__(self, message, status_code=400):
        super().__init__(message)
        self.message = message
        self.status_code = status_code
```

### Global Error Handlers

```python
# app.py
from errors import AppError

def create_app():
    app = Flask(__name__)
    
    @app.errorhandler(AppError)
    def handle_app_error(e):
        return jsonify({"error": e.message}), e.status_code
    
    @app.errorhandler(404)
    def handle_404(e):
        return jsonify({"error": "Resource not found"}), 404
    
    @app.errorhandler(500)
    def handle_500(e):
        # Log error internally
        app.logger.error(f"Server error: {e}")
        # Return generic message to user
        return jsonify({"error": "Internal server error"}), 500
    
    return app
```

### Using in Services

```python
from errors import AppError

class UserService:
    @staticmethod
    def get_user_by_id(user_id):
        user = User.query.get(user_id)
        if not user:
            raise AppError("User not found", 404)
        return user
```

---

## Common Imports

### Basic Route File

```python
from flask import request, jsonify, abort
from services.user_service import UserService
```

### Model File

```python
from extensions import db
from datetime import datetime
```

### Service File

```python
from models import User
from extensions import db
from errors import AppError  # If using custom errors
```

### App Factory (app.py)

```python
from flask import Flask
from extensions import db, migrate, jwt, ma, bcrypt
from routes.user_routes import user_bp
from routes.auth_routes import auth_bp
```

### Authentication Service

```python
from models import User
from extensions import db, bcrypt
from flask_jwt_extended import create_access_token
from errors import AppError
```

---

## Quick SQLAlchemy CRUD

### Create

```python
new_user = User(username="alice", email="alice@example.com")
db.session.add(new_user)
db.session.commit()
```

### Read

```python
# Get by ID
user = User.query.get(1)

# Get first match
user = User.query.filter_by(email="alice@example.com").first()

# Get all
users = User.query.all()

# Get with error if not found
user = User.query.get_or_404(1)
```

### Update

```python
user = User.query.get(1)
user.email = "newemail@example.com"
db.session.commit()
```

### Delete

```python
user = User.query.get(1)
db.session.delete(user)
db.session.commit()
```

---

## Status Code Quick Reference

| Code | Use When | Example |
|------|----------|---------|
| 200 | Success (GET, PUT, PATCH, DELETE) | Retrieved user successfully |
| 201 | Created (POST) | User created successfully |
| 400 | Bad request (validation fail) | Missing required field |
| 401 | Unauthorized (auth needed) | Invalid token |
| 404 | Not found | User with ID doesn't exist |
| 500 | Server error | Database connection failed |

---

## File Structure Reference

### Small Project (Levels 0-3)

```
project/
├── app.py
├── extensions.py
├── models.py
├── services/
│   ├── user_service.py
│   └── auth_service.py
├── errors.py
├── requirements.txt
└── migrations/
```

### Large Project (Levels 4-5)

```
project/
├── app/
│   ├── __init__.py       # create_app()
│   ├── extensions.py
│   ├── models/
│   │   ├── user.py
│   │   └── book.py
│   ├── routes/
│   │   ├── user_routes.py
│   │   └── auth_routes.py
│   ├── services/
│   │   ├── user_service.py
│   │   └── auth_service.py
│   └── schemas/
│       └── user_schema.py
├── config.py
├── wsgi.py
├── requirements.txt
└── migrations/
```

---

## .gitignore Template

```
# Virtual environment
venv/
env/

# Python cache
__pycache__/
*.pyc
*.pyo

# Database
*.db

# Migrations (company policy)
migrations/

# Environment variables
.env

# IDE
.vscode/
.idea/
```

---

**Need more details?** Refer back to the specific level documentation where each pattern was introduced.

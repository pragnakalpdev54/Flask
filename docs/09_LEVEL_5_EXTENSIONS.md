# Level 5: Expert - Powerful Extensions

> **Prerequisites**: [Level 4: Scalable Structure](08_LEVEL_4_SCALABLE.md)
> **Next Step**: [Test Task 3](10_TEST_TASK_3.md)

> [!NOTE]
> **OOP Level**: Higher. Flask-Restx uses “Resource” classes with methods like `get()` and `post()`, but you can treat them as **predefined shapes for your endpoints**. Think of this level as an **OOP preview**—you can still follow along by copying the pattern without deep OOP knowledge.

## Goal

In the final level, we stop reinventing the wheel. We will use **Flask-Restx** to get auto-generated Swagger documentation (like DRF's browserable API) and **Flask-Admin** for an instant internal dashboard.

We will also put together a short **production checklist** so you know what changes are required to safely run your Flask API in the real world.

## Table of Contents

1. [Flask-Restx: Better APIs](#flask-restx-better-apis)
2. [Documenting with Swagger](#documenting-with-swagger)
3. [Flask-Admin: Instant Dashboard](#flask-admin-instant-dashboard)
4. [Production Checklist](#production-checklist)
5. [Deployment Note](#deployment-note)
6. [Practice Problems](#practice-problems)

## Flask-Restx: Better APIs

`Flask-Restx` adds structure and documentation. It's similar to DRF.

### Setup

```bash
pip install flask-restx
```

### Usage with Blueprints

Instead of `Blueprint`, we use `Namespace`.

**`routes/api_routes.py`**

```python
from flask_restx import Namespace, Resource, fields
from services.user_service import UserService

# Define Namespace (like Blueprint)
api = Namespace('users', description='User operations')

# Define Model (for Swagger Docs)
user_model = api.model('User', {
    'id': fields.Integer(readonly=True),
    'username': fields.String(required=True),
    'email': fields.String(required=True)
})

@api.route('/')
class UserList(Resource):
    @api.doc('list_users')
    @api.marshal_list_with(user_model)
    def get(self):
        """List all users"""
        return UserService.get_all_users()

    @api.doc('create_user')
    @api.expect(user_model)
    @api.marshal_with(user_model, code=201)
    def post(self):
        """Create a new user"""
        data = api.payload
        return UserService.create_user(data['username'], data['email']), 201
```

> [!TIP]
> **Magic Box – `Namespace` & `Resource`**: In Flask-Restx, `Namespace` and `Resource` are Magic Boxes that define *how* routes are grouped and exposed in Swagger. Focus on the methods (`get`, `post`, etc.) and the models you document, not on how these classes are implemented internally.

### Registering in `app/__init__.py` (Factory)

```python
from flask_restx import Api
from routes.api_routes import api as user_ns

def create_app():
    # ... init app ...
    
    # Init Restx API
    api = Api(app, title='My API', version='1.0', description='Main API')
    
    # Add Namespaces
    api.add_namespace(user_ns, path='/api/users')
    
    return app
```

> [!TIP]
> **Magic Box – `Api` wrapper**: `Api(app, ...)` wires your Namespaces into a single Swagger UI. You mostly configure its title, version, and paths.

## Documenting with Swagger

**Good news!** If you used Flask-Restx as above, you **already have it.**

1. Run your app.
2. Go to `http://127.0.0.1:5000/`.
3. You will see a beautiful Swagger UI where you can try out your API endpoints.

**Why Use It?**

- Clients/Frontend devs can see exactly what JSON to send.
- They can test it directly in the browser.
- It stays up-to-date automatically as you change code.

## Flask-Admin: Instant Dashboard

Need a backend to manage users/data? Don't build it.

### Setup

```bash
pip install flask-admin
```

### Usage

**`extensions.py`**

```python
from flask_admin import Admin
admin = Admin(name='My Dashboard', template_mode='bootstrap3')
```

**`app/__init__.py`**

```python
from extensions import admin
from models.user import User
from flask_admin.contrib.sqla import ModelView

def create_app():
    # ...
    admin.init_app(app)
    
    # Add Views
    with app.app_context():
        # You might need to import db here to register views
        from extensions import db
        admin.add_view(ModelView(User, db.session))
        
    return app
```

> [!TIP]
> **Magic Box – Admin UI**: `Admin` and `ModelView` give you a working backoffice with minimal code. You just register models; the library handles forms, CRUD screens, and layout.

Visit `http://localhost:5000/admin` to see your data.

## Production Checklist

This is a **minimum** set of things you must do before exposing your Flask API to real users.

### 1. Configuration and secrets

- Set `FLASK_ENV=production` (or do not rely on debug defaults).
- Do **not** hardcode secrets in code:
  - `SECRET_KEY`
  - `JWT_SECRET_KEY`
- Use environment variables and your `Config` / `ProdConfig` classes:

```python
# config.py (example)

class ProdConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get("DATABASE_URL")
    DEBUG = False
    JWT_SECRET_KEY = os.environ.get("JWT_SECRET_KEY")
```

### 2. Logging and error visibility

- Keep `DEBUG = False` in production so Flask does not show stack traces to users.
- Use Flask’s built-in logger (or `logging`) in your global error handlers:

```python
# inside create_app

@app.errorhandler(500)
def handle_500(e):
    app.logger.exception("Unhandled error: %s", e)
    return jsonify({"error": "Internal Server Error"}), 500
```

- Send logs somewhere central (file, Docker logs, CloudWatch, etc.) so you can debug issues after they happen.

### 3. CORS and browser clients

If your API will be called from a browser frontend on a different origin, configure **CORS** explicitly.

```bash
pip install flask-cors
```

```python
from flask_cors import CORS

def create_app(config_class=Config):
    app = Flask(__name__)
    app.config.from_object(config_class)

    CORS(app, resources={r"/api/*": {"origins": "*"}})
    # tighten allowed origins in real projects instead of "*"

    return app
```

### 4. Database and migrations

- Use a real database (PostgreSQL/MySQL), not SQLite, for multi-user production workloads.
- Run migrations before deploying new code:

```bash
flask db upgrade
```

### 5. Running behind a WSGI server

- Do **not** use `app.run()` in production.
- Use a WSGI server such as **Gunicorn** or **uWSGI**:

```bash
gunicorn -w 4 wsgi:application
```

- Ensure your `wsgi.py` exposes `application = create_app(ProdConfig)` (or similar).

### 6. Security basics

- Use HTTPS in front of your Flask app (NGINX / a cloud load balancer).
- Set secure cookie flags if you ever use cookies (not needed for pure JWT APIs).
- Rotate JWT secret keys if compromised.

## Deployment Note

When deploying (e.g., to AWS, Heroku, DigitalOcean):

1. **Never use `app.run()`**: That's only for development.
2. **Use Gunicorn**: `gunicorn -w 4 wsgi:application`
3. **Set Environment Variables**:
   - `FLASK_ENV=production`
   - `SECRET_KEY=...` (Strong random string)
   - `DATABASE_URL=...` (Your prod DB)

## Practice Problems

### Problem 1: First Swagger

Convert your "Hello World" or "Books" API to use Flask-Restx. Verify you can see the Swagger UI.

### Problem 2: Documenting Input

Use `@api.expect(model)` to document what fields are required for creating a book (title, author).

### Problem 3: Admin Panel

Add `Flask-Admin` to your project. Register your `User` and `Book` models. Try creating a user via the Admin UI.

## Trivia

### Question 1

**What does Flask-Restx give you for free?**

- [ ] A database
- [x] Swagger Documentation (UI)
- [ ] Faster performance
- [ ] Users

### Question 2

**What server should you use for production?**

- [ ] `app.run()`
- [ ] The built-in development server
- [x] A WSGI server like Gunicorn or uWSGI
- [ ] None, Flask runs itself

### Question 3

**In Flask-Restx, what replaces the Concept of a Blueprint?**

- [ ] `Module`
- [ ] `Component`
- [x] `Namespace`
- [ ] `Section`

## 🔮 What's Next? (OOP Preview)

You have mastered functional Flask! But if you go to a sophisticated Django or Java shop, you will see a different style called **Object-Oriented Programming (OOP)**.

In Flask-Restx (Level 5), you saw a glimpse of it:

```python
class UserList(Resource):
    def get(self):
        # ...
    def post(self):
        # ...
```

**Why do some people prefer this?**

1. **Grouping**: It keeps `GET`, `POST`, `PUT` for the same URL in one visual block.
2. **Inheritance**: You can create a `BaseResource` that checks permissions, and all other resources inherit from it.

**Do you need to rewrite everything?**
No! Functional views (what you learned in Levels 0-4) are perfectly fine and preferred by many Flask developers. Use classes only when they make life *easier*, not harder.

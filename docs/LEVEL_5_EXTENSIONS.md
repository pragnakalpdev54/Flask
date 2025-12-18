# Level 5: Expert - Powerful Extensions

## Goal

In the final level, we stop reinventing the wheel. We will use **Flask-Restx** to get auto-generated Swagger documentation (like DRF's browserable API) and **Flask-Admin** for an instant internal dashboard.

## Table of Contents

1. [Flask-Restx: Better APIs](#flask-restx-better-apis)
2. [Documenting with Swagger](#documenting-with-swagger)
3. [Flask-Admin: Instant Dashboard](#flask-admin-instant-dashboard)
4. [Deployment Note](#deployment-note)
5. [Practice Problems](#practice-problems)

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

Visit `http://localhost:5000/admin` to see your data.

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

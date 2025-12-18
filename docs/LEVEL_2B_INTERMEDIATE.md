# Level 2B: Intermediate - Serialization & Error Handling

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
from models import User

ma = Marshmallow()

class UserSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User
        load_instance = True  # Optional: deserialize to model instances
        # exclude = ('password_hash',) # Hiding fields

# Initialize schemas
user_schema = UserSchema()
users_schema = UserSchema(many=True)
```

## Validation (Input)

Use the schema to validate incoming JSON before it reaches your Service Layer.

```python
@user_bp.route('/', methods=['POST'])
def create_user():
    json_data = request.get_json()
    
    try:
        # Validate data
        data = user_schema.load(json_data)
    except ValidationError as err:
        return jsonify(err.messages), 400
        
    # Proceed to service...
    user = UserService.create_user(data['username'], data['email'])
```

## Serialization (Output)

Never manually build dictionaries like `{'id': user.id, ...}`. Use the schema.

```python
@user_bp.route('/', methods=['GET'])
def get_users():
    users = UserService.get_all_users()
    
    # Dump list of users
    result = users_schema.dump(users)
    return jsonify(result), 200

@user_bp.route('/<int:id>', methods=['GET'])
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

### Problem 2: Validate Implementation

Update your `create_book` route to use `book_schema.load()`. Try sending invalid data (missing title) and confirm you get a 400 JSON response with validation errors.

### Problem 3: 404 Handler

Implement a global 404 handler that returns `{"error": "This endpoint does not exist"}`. Use Postman to trigger it.

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

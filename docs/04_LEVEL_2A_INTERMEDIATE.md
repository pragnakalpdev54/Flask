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

> [!NOTE]
> We use psycopg2 for PostgreSQL, but strict ORM usage means we can switch easily.

### Configuration

#### Why `extensions.py`?

In larger Flask applications, we create a separate `extensions.py` file to avoid **circular import** problems.

**The Problem:**

- `app.py` needs to import models to register routes
- `models.py` needs `db` to define tables
- If both files import from each other → **CircularImportError**

**The Solution:**

- Create `extensions.py` to hold `db` and `migrate`
- Both `app.py` and `models.py` import from `extensions.py`
- No circular dependency!

> [!NOTE]
> For tiny projects, you *could* put everything in `app.py`. But learning the `extensions.py` pattern now will save you pain later.

#### File Structure

Here's how to organize your Flask project:

```
my_flask_project/
├── app.py              # Application factory and routes
├── extensions.py       # Database and migration objects
├── models.py           # Database models
├── requirements.txt
└── migrations/         # Created after 'flask db init'
```

#### Step 1: Create `extensions.py`

```python
# extensions.py
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

db = SQLAlchemy()
migrate = Migrate()
```

This file just creates the "magic boxes" - they're not connected to any app yet.

#### Step 2: Create Your App in `app.py`

```python
# app.py
from flask import Flask
from extensions import db, migrate

def create_app():
    app = Flask(__name__)

    # Configure your DB URI
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'  # or postgresql://...
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False  # Recommended

    # Initialize extensions with app
    db.init_app(app)
    migrate.init_app(app, db)

    return app

if __name__ == "__main__":
    app = create_app()
    app.run(debug=True)
```

#### Step 3: Define Models in `models.py`

```python
# models.py
from extensions import db  # Import db from extensions, not app!

class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
```

#### Step 4: **CRITICAL** - Import Models in `app.py`

> [!WARNING]
> Flask-Migrate will **NOT** detect your models unless you import them!

Update your `app.py`:

```python
# app.py
from flask import Flask
from extensions import db, migrate
from models import User  # Import models so migrations detect them

def create_app():
    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'

    db.init_app(app)
    migrate.init_app(app, db)

    return app
```

**Why import inside the function?** To avoid circular imports. By importing inside `create_app()`, both files can safely import from `extensions.py` first.

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

### Model Relationships

When your data is related (like Authors and their Books), you need to define relationships.

#### One-to-Many Relationship

**Example**: One Author can have multiple Books. One Book belongs to one Author.

```python
# models.py
from extensions import db

class Author(db.Model):
    __tablename__ = 'authors'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), unique=True, nullable=False)
    bio = db.Column(db.String(500))

    # One-to-Many: One author has many books
    books = db.relationship('Book', backref='author', cascade='all, delete-orphan')

class Book(db.Model):
    __tablename__ = 'books'

    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    price = db.Column(db.Float, nullable=False)

    # Foreign Key: Links to Author table
    author_id = db.Column(db.Integer, db.ForeignKey('authors.id'), nullable=False)
```

**Key parts explained:**

1. **`db.ForeignKey('authors.id')`**: Creates a link to the `authors` table's `id` column
2. **`db.relationship('Book', ...)`**: Tells SQLAlchemy how models are connected
3. **`backref='author'`**: Automatically creates `book.author` to access the author from a book
4. **`cascade='all, delete-orphan'`**: When you delete an author, all their books are deleted too

**Using relationships in code:**

```python
# Create an author
author = Author(name="J.K. Rowling", bio="British author")
db.session.add(author)
db.session.commit()

# Create books for that author
book1 = Book(title="Harry Potter", price=19.99, author_id=author.id)
book2 = Book(title="Fantastic Beasts", price=15.99, author_id=author.id)
db.session.add_all([book1, book2])
db.session.commit()

# Access author's books
print(author.books)  # [<Book Harry Potter>, <Book Fantastic Beasts>]

# Access book's author (thanks to backref)
print(book1.author)  # <Author J.K. Rowling>

# Cascade delete: Delete author deletes all their books
db.session.delete(author)
db.session.commit()
# Both books are automatically deleted!
```

**Common cascade options:**

- `'all, delete-orphan'`: Delete children when parent is deleted (most common)
- `'all'`: All operations cascade, but orphans remain
- `None`: No cascading (default)

> [!NOTE]
> **For Test Task 2**: You'll need to use exactly this pattern for Authors and Books with cascade delete!

## Database Migrations

### Why Migrations?

Imagine you have a running website with 10,000 users in your database. You realize you need to add a `phone_number` column to the `users` table.

**Bad approach (loses all data):**

```python
# Delete the database and recreate it
os.remove('app.db')
db.create_all()  # All 10,000 users are gone!
```

**Good approach (with migrations):**

```bash
# Add column to model, then run:
flask db migrate -m "Add phone number to users"
flask db upgrade
# All 10,000 users are still there, just with a new NULL phone_number column
```

**Migrations** allow you to evolve your database schema over time without losing data. **Alembic** (handled by Flask-Migrate) generates the SQL commands needed to transform your database from one state to another.

### The Migration Workflow: When to Run Each Command

> [!IMPORTANT]
> **You do NOT need to run all 3 commands every time!** Here's when to use each:

#### 1. `flask db init` - Initialize Migrations (ONCE per project)

```bash
flask db init
```

**When to run:** Only **once** when you're setting up migrations for the first time in a project.

**What it does:** Creates a `migrations/` folder with Alembic configuration.

**You've already run this if:** You see a `migrations/` directory in your project.

> [!WARNING]
> **Do NOT run `flask db init` multiple times!** If you do, delete the `migrations/` folder first.

#### 2. `flask db migrate` - Generate Migration Script (When models change)

```bash
flask db migrate -m "Descriptive message about the change"
```

**When to run:** **Every time you modify a model** (add/remove columns, add new models, etc.).

**What it does:**

- Compares your Python models to the current database schema
- Generates a Python migration file in `migrations/versions/`
- **Does NOT** change your database yet!

**Example scenarios:**

- Added a new column: `flask db migrate -m "Add phone to user"`
- Created a new model: `flask db migrate -m "Add Product model"`
- Removed a column: `flask db migrate -m "Remove deprecated status field"`

**Important:** Always review the generated file before running `upgrade`!

#### 3. `flask db upgrade` - Apply Migration (After migrate, or when deploying)

```bash
flask db upgrade
```

**When to run:**

- **After** running `flask db migrate` to apply your changes locally
- **When deploying** to production/staging (to update their databases)
- **After pulling** code from teammates who added migrations

**What it does:** Executes the migration scripts and actually modifies your database.

### Complete Example: Adding a Column

Let's say you want to add a `phone_number` field to your User model.

#### Step 1: Modify your model

```python
# models.py
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), nullable=False)
    email = db.Column(db.String(120), nullable=False)
    phone_number = db.Column(db.String(20))  # New field!
```

#### Step 2: Generate migration

```bash
flask db migrate -m "Add phone number to users"
```

Output:

```
INFO  [alembic.autogenerate.compare] Detected added column 'users.phone_number'
Generating /path/to/migrations/versions/abc123_add_phone_number_to_users.py ... done
```

#### Step 3: Apply migration

```bash
flask db upgrade
```

Output:

```
INFO  [alembic.runtime.migration] Running upgrade -> abc123, Add phone number to users
```

Done! Your database now has the `phone_number` column.

### Common Mistakes Juniors Make

1. **❌ Running all 3 commands every time**

   ```bash
   # WRONG - Don't do this!
   flask db init    # Only run once ever
   flask db migrate
   flask db upgrade
   ```

2. **❌ Forgetting to import models in app.py**

   Result: `flask db migrate` says "No changes detected" even though you modified models.

   Fix: Make sure you imported your models in `app.py` as shown in the Configuration section.

3. **❌ Running `upgrade` without `migrate` first**

   Result: No new migration file, so `upgrade` does nothing.

### Troubleshooting

**Problem:** "No changes detected" even though I modified my model.

**Solutions:**

- Did you import the model in `app.py`?
- Did you save the `models.py` file?
- Try: `flask db migrate -m "Force migration" --autogenerate`

**Problem:** "Can't locate revision identified by 'xyz'"

**Solution:** Your `migrations/` folder is out of sync. In development, you can:

1. Delete `migrations/` folder
2. Delete `app.db`
3. Run `flask db init`, then `flask db migrate`, then `flask db upgrade`

## SQLAlchemy Fundamentals: CRUD Operations

Before we jump into Services, you need to know how to actually **use** SQLAlchemy to work with data. This section teaches you the basics.

### The Database Session

SQLAlchemy uses a **session** (`db.session`) to manage database operations. Think of it like a shopping cart:

- You add items (records) to the cart (`db.session.add()`)
- When ready, you checkout (`db.session.commit()`)
- If you change your mind, you can empty the cart (`db.session.rollback()`)

### Create: Adding New Records

```python
from models import User
from extensions import db

# Create a new user object
new_user = User(
    username="alice",
    email="alice@example.com",
    password_hash="hashed_password_here"
)

# Add to session (not saved yet!)
db.session.add(new_user)

# Commit to database (now it's saved)
db.session.commit()

# After commit, the user object gets its ID from the database
print(new_user.id)  # e.g., 1
```

**Adding multiple records:**

```python
users = [
    User(username="bob", email="bob@example.com", password_hash="hash1"),
    User(username="charlie", email="charlie@example.com", password_hash="hash2")
]

db.session.add_all(users)
db.session.commit()
```

### Read: Querying Data

SQLAlchemy provides a powerful query API. Here are the essentials:

#### Get All Records

```python
# Get all users
users = User.query.all()
for user in users:
    print(user.username)
```

#### Get by Primary Key

```python
# Get user with ID = 1
user = User.query.get(1)
if user:
    print(user.username)
else:
    print("User not found")
```

Or safer with `get_or_404` (if using in routes):

```python
from flask import abort

user = User.query.get_or_404(1)  # Auto-returns 404 if not found
```

#### Filter Records

```python
# Get all users with a specific username
users = User.query.filter_by(username="alice").all()

# Get first user matching condition
user = User.query.filter_by(email="alice@example.com").first()
if not user:
    print("No user found")
```

**Multiple conditions:**

```python
# Find active users named Alice
users = User.query.filter_by(username="alice", is_active=True).all()
```

**Complex filters:**

```python
# Using filter() for more complex conditions
users = User.query.filter(User.username.like('%alice%')).all()

# Multiple conditions with filter()
users = User.query.filter(
    User.is_active == True,
    User.email.contains('@example.com')
).all()
```

#### Count Records

```python
total_users = User.query.count()
active_users = User.query.filter_by(is_active=True).count()
```

#### Order Results

```python
# Order by username ascending
users = User.query.order_by(User.username).all()

# Order by ID descending
users = User.query.order_by(User.id.desc()).all()
```

#### Limit Results

```python
# Get first 10 users
users = User.query.limit(10).all()

# Skip first 20, then get next 10 (pagination)
users = User.query.offset(20).limit(10).all()
```

### Update: Modifying Existing Records

```python
# Get the user
user = User.query.get(1)

if user:
    # Modify attributes
    user.email = "newemail@example.com"
    user.is_active = False

    # Commit changes
    db.session.commit()
```

**Bulk update:**

```python
# Update all inactive users to active
User.query.filter_by(is_active=False).update({"is_active": True})
db.session.commit()
```

### Delete: Removing Records

```python
# Get the user
user = User.query.get(1)

if user:
    db.session.delete(user)
    db.session.commit()
```

**Bulk delete:**

```python
# Delete all inactive users
User.query.filter_by(is_active=False).delete()
db.session.commit()
```

### Error Handling with Rollback

If something goes wrong, you can rollback the session:

```python
try:
    new_user = User(username="alice", email="alice@example.com")
    db.session.add(new_user)
    db.session.commit()
except Exception as e:
    db.session.rollback()  # Undo all changes
    print(f"Error: {e}")
```

### Quick Reference: SQLAlchemy CRUD

| Operation | Code Example |
|-----------|-------------|
| **Create** | `db.session.add(obj)` + `db.session.commit()` |
| **Read All** | `Model.query.all()` |
| **Read One** | `Model.query.get(id)` or `.first()` |
| **Filter** | `Model.query.filter_by(field=value).all()` |
| **Update** | `obj.field = new_value` + `db.session.commit()` |
| **Delete** | `db.session.delete(obj)` + `db.session.commit()` |
| **Rollback** | `db.session.rollback()` |

## The Service Layer Pattern

This is the most important architectural rule in our project.

### 🧠 Concept: Classes as Namespaces

We use `class UserService`. **Don't panic.** We are not creating objects (`user_service = UserService()`).

We are just using the class as a **Namespace** (like a folder) to group related functions together.

- `UserService.create_user()`
- `UserService.get_all_users()`

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

You can implement the Service Layer using either **classes** or **functions**. Both are correct!

#### Option 1: Class-Based Service (Recommended)

**1. Create the Service (`services/user_service.py`)**

```python
from models import User
from extensions import db

class UserService:
    @staticmethod
    def create_user(username, email, password_hash):
        """
        Handles business logic for creating a user.
        """
        # Validation Logic
        if User.query.filter_by(email=email).first():
            return None, "Email already exists"

        # DB Logic
        new_user = User(username=username, email=email, password_hash=password_hash)
        db.session.add(new_user)
        db.session.commit()

        return new_user, None

    @staticmethod
    def get_all_users():
        return User.query.all()
```

**2. Call it from the Route (`app.py`)**

```python
from flask import Flask, request, jsonify
from extensions import db
from models import User
from services.user_service import UserService

app = Flask(__name__)
# ... app configuration ...

@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()

    # Route just delegates to Service
    user, error = UserService.create_user(
        username=data['username'],
        email=data['email'],
        password_hash=data['password'],  # In real app, hash this first!
    )

    if error:
        return jsonify({"error": error}), 400

    return jsonify({"id": user.id, "message": "Created"}), 201


@app.route('/users', methods=['GET'])
def get_users():
    users = UserService.get_all_users()
    return jsonify({"users": [{"id": u.id, "username": u.username} for u in users]}), 200
```

#### Option 2: Function-Based Service (Also Acceptable)

If you're not comfortable with classes, you can use plain functions:

**1. Create Service Functions (`services/user_functions.py`)**

```python
from models import User
from extensions import db

def create_user(username, email, password_hash):
    """
    Handles business logic for creating a user.
    """
    # Validation Logic
    if User.query.filter_by(email=email).first():
        return None, "Email already exists"

    # DB Logic
    new_user = User(username=username, email=email, password_hash=password_hash)
    db.session.add(new_user)
    db.session.commit()

    return new_user, None


def get_all_users():
    return User.query.all()
```

**2. Call it from the Route (`app.py`)**

```python
from flask import Flask, request, jsonify
from extensions import db
from models import User
from services.user_functions import create_user, get_all_users

app = Flask(__name__)
# ... app configuration ...

@app.route('/users', methods=['POST'])
def create_user_endpoint():
    data = request.get_json()

    # Route just delegates to service function
    user, error = create_user(
        username=data['username'],
        email=data['email'],
        password_hash=data['password'],
    )

    if error:
        return jsonify({"error": error}), 400

    return jsonify({"id": user.id, "message": "Created"}), 201


@app.route('/users', methods=['GET'])
def get_users_endpoint():
    users = get_all_users()
    return jsonify({"users": [{"id": u.id, "username": u.username} for u in users]}), 200
```

> [!NOTE]
> Both approaches achieve the same goal! The class-based approach is industry standard for larger projects, but function-based works perfectly. Choose what you're comfortable with.

### Why?

1. **Reusability**: You can call `UserService.create_user` from a CLI command or a test or another service.
2. **Testing**: You can test `UserService` without faking HTTP requests.
3. **Maintainability**: Routes stay simple and clean.

## Refactoring: The Right Way

When you have existing code with fat routes (business logic and database calls in routes), here's how to refactor it properly:

### Step-by-Step Refactoring Process

1. **Identify the Logic**: Find all business logic and database operations in your routes
2. **Extract to Service**: Move that logic to a service method
3. **Update Route**: Make the route call the service method
4. **Test**: Verify the behavior remains the same

### Example: Refactoring a Create Endpoint

**Before (Fat Route):**

```python
@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    # Validation in route
    if User.query.filter_by(email=data['email']).first():
        return jsonify({"error": "Email exists"}), 400
    # DB logic in route
    new_user = User(username=data['username'], email=data['email'])
    db.session.add(new_user)
    db.session.commit()
    return jsonify({"id": new_user.id}), 201
```

**After (Thin Route + Service):**

```python
# In services/user_service.py
class UserService:
    @staticmethod
    def create_user(username, email):
        if User.query.filter_by(email=email).first():
            return None, "Email already exists"
        new_user = User(username=username, email=email)
        db.session.add(new_user)
        db.session.commit()
        return new_user, None

# In app.py
@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    user, error = UserService.create_user(
        username=data['username'],
        email=data['email']
    )
    if error:
        return jsonify({"error": error}), 400
    return jsonify({"id": user.id}), 201
```

### Key Refactoring Principles

* **One Step at a Time**: Refactor one route at a time, test, then move to the next
* **Preserve Behavior**: The API should work exactly the same after refactoring
* **Keep It Simple**: Don't over-engineer - extract logic, don't add complexity

## Practice Problems

### Problem 1: Book Model

Create a `Book` model with the following fields:

- `title` (String, required)
- `author` (String, required)
- `isbn` (String, unique)

#### Step 1: Create the Model

Create `models.py` (or add to existing):

```python
# models.py
from extensions import db

class Book(db.Model):
    __tablename__ = 'books'

    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    author = db.Column(db.String(100), nullable=False)
    isbn = db.Column(db.String(13), unique=True, nullable=False)
```

#### Step 2: Run Migrations

```bash
flask db migrate -m "Add Book model"
flask db upgrade
```

Verify the migration was created and the table exists.

### Problem 2: Book Service

Create a `BookService` with methods:

- `add_book(title, author, isbn)`
- `get_books_by_author(author)`

Ensure `add_book` raises an error if ISBN already exists.

**Starter code**:

```python
# services/book_service.py
from models import Book
from extensions import db

class BookService:
    @staticmethod
    def add_book(title, author, isbn):
        # Check if ISBN exists
        existing = Book.query.filter_by(isbn=isbn).first()
        if existing:
            return None, "Book with this ISBN already exists"

        # Create book
        book = Book(title=title, author=author, isbn=isbn)
        db.session.add(book)
        # Hint: Something is missing here...

        return book, None

    @staticmethod
    def get_books_by_author(author):
        return Book.query.filter_by(author=author).all()
```

### Problem 3: Refactor Calculator to Use Service Pattern

In Level 1, you built a `/calculate` endpoint that performed operations directly in the route. Now, refactor it to use the **Service Layer Pattern**.

**What is "CalculationService"?**

A `CalculationService` is simply a place to put your calculator logic (add, subtract, multiply, divide) **outside** of the route function. The route should only:

1. Get the request data
2. Call the service
3. Return the response

The service does the actual calculation.

**Why do this?**

- **Testable**: You can test calculations without making HTTP requests
- **Reusable**: Other parts of your app can use the same calculation functions
- **Clean**: Routes stay simple and focused on HTTP handling

#### Solution 1: Class-Based (Recommended)

```python
# services/calculation_service.py
class CalculationService:
    @staticmethod
    def calculate(operation, x, y):
        """
        Performs a calculation based on the operation.
        Raises ValueError if operation is invalid.
        """
        if operation == "add":
            return x + y
        elif operation == "subtract":
            return x - y
        elif operation == "multiply":
            return x * y
        elif operation == "divide":
            if y == 0:
                raise ValueError("Cannot divide by zero")
            return x / y
        else:
            raise ValueError(f"Invalid operation: {operation}")
```

```python
# app.py or routes/calculator_routes.py
from flask import Flask, request, jsonify
from services.calculation_service import CalculationService

app = Flask(__name__)

@app.route("/calculate", methods=["POST"])
def calculate():
    data = request.get_json()

    # Validation
    if not data:
        return jsonify({"error": "Request body required"}), 400

    operation = data.get("operation")
    x = data.get("x")
    y = data.get("y")

    if not all([operation, x is not None, y is not None]):
        return jsonify({"error": "Missing operation, x, or y"}), 400

    # Call the service
    try:
        result = CalculationService.calculate(operation, x, y)
        return jsonify({"result": result}), 200
    except ValueError as e:
        return jsonify({"error": str(e)}), 400
```

#### Solution 2: Function-Based (Also Acceptable)

If you're not comfortable with classes yet, you can use plain functions:

```python
# services/calculation_functions.py
def calculate(operation, x, y):
    """
    Performs a calculation based on the operation.
    Raises ValueError if operation is invalid.
    """
    if operation == "add":
        return x + y
    elif operation == "subtract":
        return x - y
    elif operation == "multiply":
        return x * y
    elif operation == "divide":
        if y == 0:
            raise ValueError("Cannot divide by zero")
        return x / y
    else:
        raise ValueError(f"Invalid operation: {operation}")
```

```python
# app.py or routes/calculator_routes.py
from flask import Flask, request, jsonify
from services.calculation_functions import calculate

app = Flask(__name__)

@app.route("/calculate", methods=["POST"])
def calculate_endpoint():
    data = request.get_json()

    if not data:
        return jsonify({"error": "Request body required"}), 400

    operation = data.get("operation")
    x = data.get("x")
    y = data.get("y")

    if not all([operation, x is not None, y is not None]):
        return jsonify({"error": "Missing operation, x, or y"}), 400

    try:
        result = calculate(operation, x, y)
        return jsonify({"result": result}), 200
    except ValueError as e:
        return jsonify({"error": str(e)}), 400
```

> [!NOTE]
> Both solutions are correct! The class-based approach is the industry standard for larger projects, but the function-based approach works perfectly fine. Choose whichever you're more comfortable with.

## Common Pitfalls Quiz

Test your understanding of database and architecture mistakes:

### Pitfall 1: Forgetting db.session.commit()

**Question**: Why doesn't the user get created?

```python
from models import User
from extensions import db

@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    user = User(username=data['username'], email=data['email'])
    db.session.add(user)
    # Missing commit!
    return jsonify({"message": "Created"}), 201
```

<details>
<summary>Click to see answer</summary>

**Answer**: Changes are only written to the database when you call `db.session.commit()`. Without it, changes stay in memory and are lost.

**Fix**:

```python
@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    user = User(username=data['username'], email=data['email'])
    db.session.add(user)
    db.session.commit()  # Actually save to database!
    return jsonify({"message": "Created"}), 201
```

**Remember**: SQLAlchemy uses a "Unit of Work" pattern:

1. `db.session.add()` - Stage changes
2. `db.session.commit()` - Write to database

</details>

### Pitfall 2: Running flask db init Multiple Times

**Question**: What happens?

```bash
$ flask db init
# Creates migrations/ folder
$ flask db init  # Run again by mistake
Error: Directory migrations already exists
```

<details>
<summary>Click to see answer</summary>

**Answer**: `flask db init` should only be run **ONCE** per project. It creates the `migrations/` folder structure.

**If you need to start fresh**:

```bash
# Delete everything and start over (DEVELOPMENT ONLY!)
rm -rf migrations/
rm app.db
flask db init
flask db migrate -m "Initial migration"
flask db upgrade
```

**Remember**:

- `flask db init` - Once per project
- `flask db migrate` - Every time models change
- `flask db upgrade` - After migrate, and when deploying

</details>

### Pitfall 3: Forgetting to Import Models

**Question**: Why does `flask db migrate` say "No changes detected"?

```python
# app.py
from flask import Flask
from extensions import db, migrate

def create_app():
    app = Flask(__name__)
    db.init_app(app)
    migrate.init_app(app, db)
    # Forgot to import models!
    return app

# models.py
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80))
```

<details>
<summary>Click to see answer</summary>

**Answer**: Flask-Migrate can't detect models you haven't imported! If Python doesn't load the model class, Alembic doesn't know it exists.

**Fix**:

```python
# app.py
from flask import Flask
from extensions import db, migrate
from models import User  # Import all models!

def create_app():
    app = Flask(__name__)
    db.init_app(app)
    migrate.init_app(app, db)
    return app
```

</details>

### Pitfall 4: Putting Database Logic in Routes

**Question**: What's wrong with this architecture?

```python
@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()

    # Business logic in route!
    if User.query.filter_by(email=data['email']).first():
        return jsonify({"error": "Email exists"}), 400

    # Database calls in route!
    user = User(username=data['username'], email=data['email'])
    db.session.add(user)
    db.session.commit()

    return jsonify({"id": user.id}), 201
```

<details>
<summary>Click to see answer</summary>

**Answer**: **FAT ROUTES ARE BAD!** This violates the Service Layer pattern. Routes should only handle HTTP, not business logic or database calls.

**Fix with Service Layer**:

```python
# services/user_service.py
class UserService:
    @staticmethod
    def create_user(username, email):
        if User.query.filter_by(email=email).first():
            return None, "Email exists"

        user = User(username=username, email=email)
        db.session.add(user)
        db.session.commit()
        return user, None

# app.py
@app.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()
    user, error = UserService.create_user(data['username'], data['email'])

    if error:
        return jsonify({"error": error}), 400
    return jsonify({"id": user.id}), 201
```

</details>

### Pitfall 5: Circular Import Hell

**Question**: Why does this crash with ImportError?

```python
# app.py
from extensions import db
from models import User

app = Flask(__name__)
db.init_app(app)

# models.py
from app import db  # Circular import!

class User(db.Model):
    pass
```

<details>
<summary>Click to see answer</summary>

**Answer**: Circular import! `app.py` imports `models.py`, and `models.py` tries to import `app.py`.

**Fix with extensions.py**:

```python
# extensions.py
from flask_sqlalchemy import SQLAlchemy
db = SQLAlchemy()

# models.py
from extensions import db  # Import from extensions!

class User(db.Model):
    pass

# app.py
from extensions import db
from models import User

def create_app():
    app = Flask(__name__)
    db.init_app(app)
    return app
```

</details>

## Trivia

### Question 1

**Where should database queries reside?**

- [ ] In the Route function
- [ ] In the Template
- [x] In the Service Layer
- [ ] In the `app.py`

### Question 2

**What command applies changes to the database?**

- [ ] `flask db init`
- [ ] `flask db migrate`
- [x] `flask db upgrade`
- [ ] `flask run`

### Question 3

**Why do we avoid "Fat Routes"?**

- [ ] They take up too much disk space
- [x] They mix business logic with HTTP logic, making code hard to test and reuse
- [ ] Flask forbids them

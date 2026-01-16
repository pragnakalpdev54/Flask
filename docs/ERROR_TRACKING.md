# ERROR TRACKING DOCUMENT

## ⚠️ FOR TEACHER USE ONLY ⚠️

This document tracks all intentional errors placed in the Flask documentation to test students' debugging skills and ensure they're reading code carefully.

## Error Summary

| # | Level | Location | Error Type | Description | Expected Fix | Tests |
|---|-------|----------|------------|-------------|--------------|-------|
| 1 | 2A | Practice Problem 1 | Silent/Import | Model created but not imported in app.py | Add `from models import Book` | Import awareness, silent failures |
| 2 | 1 | Practice Problem 2 | Import | Uses `jsonify()` without importing it | Add `jsonify` to imports | Import awareness |
| 3 | 2A | Practice Problem 2 | Logical/DB | Missing `db.session.commit()` | Add commit after add | Database transactions |
| 4 | 2B | Practice Problem 2 | Logical | Uses schema.load() for output | Change to schema.dump() | Schema understanding |
| 5 | 3 | Practice Problem 3 | Security | Missing @jwt_required() | Add decorator | Authentication |
| 6 | 4 | Practice Problem 1 | Logical | Blueprint not registered | Add app.register_blueprint() | Blueprint pattern |

---

## Detailed Error Documentation

### Error #1: Missing Model Import (Level 2A) ⭐

**File**: `04_LEVEL_2A_INTERMEDIATE.md`
**Location**: Practice Problem 1: "Book Model"
**Lines**: ~823-855

**Intentional Error**:

- Provides complete `Book` model code
- Shows how to run migrations
- **NEVER mentions** importing the model in `app.py`

**What Happens**:

```bash
$ flask db migrate -m "Add Book model"
INFO  [alembic.autogenerate.compare] Detected no changes  # ← Silent failure!
```

**Expected Fix**: Add `from models import Book` to `app.py` or `app/__init__.py`

**Learning**: Debugging silent failures, understanding Flask-Migrate requirements

---

### Error #2: Missing jsonify Import (Level 1)

**File**: `02_LEVEL_1_FOUNDATIONS.md`
**Location**: Practice Problem 2 (Extended): "Calculator API"
**Lines**: ~803-849

**Intentional Error**:

- Code uses `jsonify()` throughout
- Import statement only has: `from flask import Flask, request`
- Missing `jsonify` from imports

**What Happens**:

```python
NameError: name 'jsonify' is not defined
```

**Expected Fix**: Add `jsonify` to the import: `from flask import Flask, request, jsonify`

**Learning**: Import awareness, reading error messages

---

### Error #3: Missing db.session.commit() (Level 2A)

**File**: `04_LEVEL_2A_INTERMEDIATE.md`
**Location**: Practice Problem 2: "Book Service"
**Lines**: ~857-891

**Intentional Error**:

- Code creates book and adds to session
- **Missing** `db.session.commit()`
- Hint comment: "What's missing here?"

**What Happens**:

- Book appears to be created
- But when querying database, book doesn't exist
- Changes weren't saved

**Expected Fix**: Add `db.session.commit()` after `db.session.add(book)`

**Learning**: Database transaction understanding, Unit of Work pattern

---

### Error #4: schema.load() vs dump() Confusion (Level 2B)

**File**: `05_LEVEL_2B_INTERMEDIATE.md`
**Location**: Practice Problem 2: "Serialize User List"
**Lines**: ~179-200

**Intentional Error**:

- Endpoint fetches users from database
- Uses `users_schema.load(users)` instead of `dump()`
- Question comment: "Is this correct?"

**What Happens**:

```python
TypeError or ValidationError - load() expects dict/JSON, not model objects
```

**Expected Fix**: Change `load()` to `dump()`

**Learning**: Understanding load (input) vs dump (output)

---

### Error #5: Missing @jwt_required() (Level 3)

**File**: `06_LEVEL_3_AUTHENTICATION.md`
**Location**: Practice Problem 3: "Protected Route"
**Lines**: ~239-260

**Intentional Error**:

- Route calls `get_jwt_identity()` but has NO `@jwt_required()` decorator
- Anyone can access without authentication

**What Happens**:

- Route accessible to everyone (no 401)
- `get_jwt_identity()` returns `None`

**Expected Fix**: Add `@jwt_required()` decorator above the route

**Learning**: Understanding JWT protection requirements

---

### Error #6: Blueprint Not Registered (Level 4)

**File**: `08_LEVEL_4_SCALABLE.md`
**Location**: Practice Problem 1: "Refactor to Factory"
**Lines**: ~237-266

**Intentional Error**:

- Imports blueprint: `from routes.user_routes import user_bp`
- But NEVER calls `app.register_blueprint(user_bp)`
- Comment: "Is something missing here with the blueprint?"

**What Happens**:

```
404 Not Found on all blueprint routes
```

**Expected Fix**: Add `app.register_blueprint(user_bp)` in `create_app()`

**Learning**: Blueprint registration pattern

---

## Testing Guide (For Teachers)

To verify students caught the errors:

1. **Code Review**: Check for missing imports, decorators, commits, registrations
2. **Ask**: "Did you encounter any issues?"
3. **Observe**: Did they debug or ask for help immediately?

**Good Sign**: Student debugs, reads docs, fixes it themselves
**Red Flag**: Student asks "why isn't it working" without debugging

---

## Statistics

- **Total Errors**: 6
- **Silent Failures**: 1 (model import)
- **Import Errors**: 2 (jsonify, model)
- **Logical Errors**: 3 (commit, schema, blueprint)
- **Security Errors**: 1 (JWT)

**Last Updated**: 2025-12-31

# Flask Test Task 3 – Scalable Architecture & Blueprints

> **Prerequisite**: [Level 4: Scalable Structure](08_LEVEL_4_SCALABLE.md).

## Objective

Refactor the Library Management API (from Task 2) into a production-ready Structure using the **Application Factory Pattern** and **Blueprints**.
This task evaluates:

* Project Layout & Organization
* Circular Import Avoidance
* Blueprint Usage
* Configuration Management

## Problem Statement

Take the code from Task 2 and refactor it. The functionality remains the same (Authors & Books), but the structure must change drastically.

## Required Structure

```text
library_project/
├── app/
│   ├── __init__.py         # create_app() function
│   ├── extensions.py       # db, migrate initialization
│   ├── models/             # Separate file for models (or package)
│   │   ├── author.py
│   │   └── book.py
│   ├── routes/             # Blueprints
│   │   ├── __init__.py
│   │   ├── author_routes.py
│   │   └── book_routes.py
│   ├── services/           # Service Layer
│   │   ├── author_service.py
│   │   └── book_service.py
├── config.py               # Config classes (Dev, Test)
├── wsgi.py                 # Entry point
└── requirements.txt
```

## Technical Constraints

1. **Application Factory**: `app/__init__.py` must contain `def create_app(config_class):`.
2. **Extensions**: `db` and `migrate` must be created in `extensions.py` (not in `__init__.py`).
3. **Blueprints**:
    * `author_bp` prefix: `/api/v1/authors`
    * `book_bp` prefix: `/api/v1/books`
4. **No Circular Imports**: Models must import `db` from `extensions.py`.
5. **Config**: `config.py` should have a `DevConfig` and `TestConfig`.

## Bonus Challenge (Optional)

Add a custom CLI command to seed the database with initial data.

* Command: `flask seed-db`
* Action: Adds 2 authors and 5 books.

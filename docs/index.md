# Flask Learning Guide

> [!CAUTION]
> **Read All Code Carefully Before Copy-Pasting**
>
> This documentation is designed to teach you proper debugging skills. Some code examples may contain intentional mistakes that you must identify and fix using the debugging techniques taught in Level 0.
>
> **Always:**
>
> - Read the entire code example before copying
> - Understand what each line does
> - Test your code and debug any errors
> - Use the Flask debugger when things break
>
> Blindly copy-pasting code will lead to errors. This is intentional. Debugging is a core skill.

## Welcome! 👋

This comprehensive guide will take you from a complete beginner to an expert in **Flask**. Whether you're new to Python web development or have some experience, this structured learning path will help you master building scalable, RESTful APIs with Flask.

## 📚 Learning Path Overview

This guide is divided into **7 levels** (including sub-levels), each building upon the previous one. We focus strictly on building **APIs** (no HTML templates).

- **Level 0: Complete Beginner** - Python basics, web concepts, and environment setup
- **Level 1: Foundations** - Routing, JSON handling, and Request methods
- **Level 2A: Databases** - SQLAlchemy, Migrations, and the **Service Layer**
- **Level 2B: Serialization** - Marshmallow Schemas and Global Error Handling
- **Level 3: Authentication** - JWTs, Security, and Password Hashing
- **Level 4: Scalable** - Application Factory, Blueprints, and Circular Imports

## Phases Overview (OOP Expectations)

To keep things beginner-friendly, the Flask track is split into **phases** with clear expectations about Object-Oriented Programming (OOP):

- **Phase 1 – Levels 0–1 (No OOP Required)**
  You only write **functions**. You do not need to know or use `class` at all.
- **Phase 2 – Levels 2–4 (Light `class` Usage)**
  We use `class` mainly for **models**, **schemas**, and **services** as **namespaces / configuration**. You can treat them as patterns and *Magic Boxes* and still succeed without deep OOP theory.

## ⏱️ Time Estimates

| Level | Estimated Time | Difficulty |
|-------|---------------|------------|
| Level 0 | 2-3 hours | ⭐ Beginner |
| Level 1 | 5-8 hours | ⭐ Beginner |
| Level 2A| 8-12 hours | ⭐⭐ Intermediate |
| Level 2B| 5-8 hours | ⭐⭐ Intermediate |
| Level 3 | 10-15 hours | ⭐⭐⭐ Advanced |
| Level 4 | 8-12 hours | ⭐⭐⭐ Advanced |

**Total Estimated Time: 40-60 hours** (approximately 1-2 months of part-time study)

## 📋 Prerequisites

### Required Knowledge

Before starting **Level 0**, you should have:

- ✅ **Basic Python knowledge**:
  - Variables, dictionaries, lists
  - Functions and decorators (helpful but we explain them)
  - Modules and imports

- ✅ **Command line basics**:
  - `cd`, `ls`, `mkdir`
  - Running scripts (`python app.py`)

### Optional (Helpful)

- Understanding of HTTP (GET vs POST)
- JSON syntax

## ❓ FAQ: Do I need to know Object-Oriented Programming (OOP)?

**Short Answer: No.**

Many beginners get stuck because they think they need to master Classes and Objects before learning Flask. You do not.

- **Phase 1 (Levels 0-1)**: Purely functional. You write functions. No OOP knowledge is required.
- **Phase 2 (Levels 2-4)**: We use `class` keywords, but only as "containers", **namespaces**, or **configuration**. You rarely write complex OOP logic.

> [!TIP]
> We have added specific notes in each level to explain exactly what you need to know about classes, so you never feel lost. When in doubt, glance at the **Phases Overview** above and the **Classes as Namespaces** section in Level 2A.

### FAQ: Classes, Objects, and When to Care

- **Do I need to understand classes before starting Level 0?**
  No. You can complete **Phase 1 (Levels 0–1)** using only functions and modules.

- **When will I actually need OOP knowledge?**
  Mainly when you want to go beyond this track and design more advanced architectures (custom domain models, inheritance, interfaces). You can safely finish Levels 0–4 first.

- **What does `class UserService` really mean in this course?**
  It is just a **namespace** (a folder for related functions). You typically call `UserService.create_user(...)` without worrying about constructing objects or writing `__init__` methods.

## 🎯 How to Use This Guide

### Step-by-Step Approach

1. **Start sequentially**: Do not skip Level 0 or 1. They lay the foundation.
2. **STOP at Checkpoints**: There are **3 Test Tasks**. You *must* complete them before moving on. They are the gatekeepers.
    - [Test Task 1](03_TEST_TASK_1.md) (After Level 1)
    - [Test Task 2](07_TEST_TASK_2.md) (After Level 3)
    - [Test Task 3](10_TEST_TASK_3.md) (After Level 4)
3. **Code Along**: Reading is not enough. Type the code.

## 🛠️ Tools You'll Need

1. **Python 3.8+**
2. **Code Editor** (VS Code recommended)
3. **Postman** (Essential for testing APIs. Browser is not enough!)
4. **DB Browser for SQLite** (Optional, for visualizing the DB)

## 🧰 Magic Boxes (What You Can Ignore for Now)

Some libraries and patterns in this course are **Magic Boxes**: you can use them confidently without fully understanding their internals yet.

Examples of Magic Boxes in this track:

- `Flask` itself (`Flask(__name__)`, `@app.route`)
- Database layer: `SQLAlchemy`, `Flask-Migrate`
- Serialization and validation: `Flask-Marshmallow`, `Marshmallow`
- Auth tooling: `flask-bcrypt`, `flask-jwt-extended`

> [!TIP]
> When you see a **Magic Box** callout in the levels, it means: “Focus on how to use it; you do not need to understand how it works internally yet.”

You can also refer to the **Pattern Cheatsheet** (`PATTERN_CHEATSHEET.md`) at any time for small, copy-pastable examples of how these pieces fit together.

## 📖 Guide Structure

### Phase 1: The Basics (JSON & HTTP)

- **[Level 0: Beginner](01_LEVEL_0_BEGINNER.md)**: Setup, Environment, Hello World. *(No OOP required.)*
- **[Level 1: Foundations](02_LEVEL_1_FOUNDATIONS.md)**: Routing, JSON Handling, Request Methods. *(No OOP required.)*

> [!IMPORTANT]
> **Checkpoint**: Complete **[Test Task 1](03_TEST_TASK_1.md)** before moving forward.

### Phase 2: Intermediate (Data & Logic)

- **[Level 2A: Databases](04_LEVEL_2A_INTERMEDIATE.md)**: SQLAlchemy, Migrations, **Service Layer**. *(Light `class` usage as namespaces and configuration.)*
- **[Level 2B: Serialization](05_LEVEL_2B_INTERMEDIATE.md)**: Marshmallow Schemas, Error Handling. *(Schemas are template-like classes; OOP knowledge is optional.)*
- **[Level 3: Authentication](06_LEVEL_3_AUTHENTICATION.md)**: JWTs, Securing Endpoints. *(Security-focused; classes used as thin helpers and services.)*

> [!IMPORTANT]
> **Checkpoint**: Complete **[Test Task 2](07_TEST_TASK_2.md)** (Library API) before moving forward.

### Phase 3: Advanced Architecture

- **[Level 4: Scalable Structure](08_LEVEL_4_SCALABLE.md)**: **Blueprints**, Application Factory, Solving Circular Imports. *(Focus on project structure and imports, not deep OOP.)*

> [!IMPORTANT]
> **Checkpoint**: Complete **[Test Task 3](10_TEST_TASK_3.md)** (Refactor to Scalable App).

## 🐛 Troubleshooting Common Issues

### 1. "Method Not Allowed" (405 Error)

- **Cause**: You tried to access a POST route via the browser address bar (which uses GET).
- **Fix**: Use **Postman** or `curl` to send the correct method.

### 2. "Circular Import Error"

- **Cause**: `app.py` imports `models`, and `models` imports `app` (to get `db`).
- **Fix**: See **Level 4** (The Application Factory & Extensions pattern). Move `db` to `extensions.py`.

### 3. "RuntimeError: Working outside of application context"

- **Cause**: You tried to access `current_app` or `db` without an active request or app context.
- **Fix**: Wrap your code in `with app.app_context():`.

### 4. "AttributeError: 'function' object has no attribute..."

- **Cause**: You might be trying to access a property on a function instead of a class instance, or vice versa.
- **Fix**: Check if you are using `self.variable` inside a `class` method, or just `variable` inside a standard function.

### Common Class-Related Errors (Beginner Edition)

- **`AttributeError: 'function' object has no attribute '...'`**
  Usually means you treated a **function** like an object. Example: calling `user_service.create_user(...)` when `user_service` is a function, or forgetting to import `UserService` correctly.

- **`TypeError: create_user() missing 1 required positional argument: 'self'`**
  This often appears if you define a method on a class without `@staticmethod` but call it like `UserService.create_user(...)`. In this course, we almost always mark service methods as `@staticmethod` so you never need `self`.

- **Import-related `NameError` for classes like `UserService` or `AuthService`**
  Usually a sign the import path is wrong or there is a circular import. Double-check you are importing from `services.user_service` or `services.auth_service`, and see **Level 4** for how `extensions.py` helps avoid circular imports.

## ✅ Progress Checklist

- [ ] **Level 0: Setup (no OOP)**
  - [ ] Hello World works
  - [ ] `jsonify` understood

- [ ] **Level 1: Requests (no OOP)**
  - [ ] Query Params vs JSON Body
  - [ ] Postman installed and working
  - [ ] **Test Task 1 Completed**

- [ ] **Level 2: Data (light classes as namespaces/config)**
  - [ ] Connected to SQLite
  - [ ] Migrations run successfully
  - [ ] **Service Layer** created (No DB calls in routes!)
  - [ ] Marshmallow schemas defined

- [ ] **Level 3: Security (Magic Box heavy, OOP optional)**
  - [ ] Passwords hashed (Bcrypt)
  - [ ] JWTs working
  - [ ] **Test Task 2 Completed**

- [ ] **Level 4: Architecture (classes as patterns and configuration)**
  - [ ] App Factory implemented
  - [ ] Blueprints set up
  - [ ] **Test Task 3 Completed**

## 🚀 Ready to Start?

Begin your journey with **[Level 0: Beginner](01_LEVEL_0_BEGINNER.md)**!

Good luck! 🐍

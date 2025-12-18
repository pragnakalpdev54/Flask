# Flask Learning Roadmap

Welcome to the internal Flask training series. This guide outlines the path from beginner to expert.

## How to use this guide

1. **Start sequentially**: Do not skip Level 0 or 1. They lay the foundation.
2. **Do the Practice Problems**: Reading is not enough. You must write code.
3. **Internal Projects**: After Level 3, try building a small internal tool.

## The Modules

### [Level 0: Beginner](LEVEL_0_BEGINNER.md)

**Prerequisites**: Basic Python
**Goal**: Build a "Hello World" API.
**Key Concepts**: Virtual Envs, JSON, HTTP Methods.

### [Level 1: Foundations](LEVEL_1_FOUNDATIONS.md)

**Goal**: Understand Routing and Requests.
**Key Concepts**: `request` object, URL parameters, Blueprints (`user_bp`), Serving Static Files (No Jinja!).

### [Level 2A: Intermediate - Databases](LEVEL_2A_INTERMEDIATE.md)

**Goal**: Connect to a Database properly.
**Key Concepts**: Flask-SQLAlchemy, Migrations, **Service Layer Pattern** (Crucial).

### [Level 2B: Intermediate - Serialization](LEVEL_2B_INTERMEDIATE.md)

**Goal**: robust Error Handling and Validation.
**Key Concepts**: Marshmallow Schemas, Global Error Handlers, hiding 500 errors.

### [Level 3: Authentication](LEVEL_3_AUTHENTICATION.md)

**Goal**: Secure your API.
**Key Concepts**: JWT vs Sessions, Bcrypt, Auth Service, Protecting Routes.

### [Level 4: Scalable Structure](LEVEL_4_SCALABLE.md)

**Goal**: Write Professional, Scalable Code.
**Key Concepts**: **Application Factory Pattern**, Solving Circular Imports, Project Layout.

### [Level 5: Expert Extensions](LEVEL_5_EXTENSIONS.md)

**Goal**: Use power tools.
**Key Concepts**: **Flask-Restx** (Swagger UI), Flask-Admin (Dashboards).

## Final Project Idea: Employee Directory API

Combine everything you learned to build an Employee API:

1. **Auth**: Login/Register (Level 3)
2. **Users**: Create/List Employees (Level 2A/2B)
3. **Search**: Filter by department (Level 1)
4. **Docs**: Auto-generated Swagger (Level 5)
5. **Structure**: Use App Factory & Service Layer (Level 4)

Good luck!

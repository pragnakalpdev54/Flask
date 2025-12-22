# Flask Test Task 1 – JSON File-Based CRUD API

> **STOP**: Do not attempt this task until you have completed [Level 1: Foundations](02_LEVEL_1_FOUNDATIONS.md).
> **Next Level**: [Level 2A: Databases](04_LEVEL_2A_INTERMEDIATE.md)
Objective
Build a Flask-based REST API that performs Create, Read, Update, and Delete (CRUD) operations on a JSON file acting as a simple database.
This task evaluates:
Flask fundamentals
REST API design
File handling
Error handling
Code structure and clarity
Problem Statement
You are required to create a Flask application that manages a collection of users stored in a JSON file.
The application should expose REST APIs that allow clients to:
Create a new user
Retrieve users
Update an existing user
Delete a user
Data Format
Store data in a file called users.json.
Initial Structure
{
  "users": []
}

User Object Structure
{
  "id": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
  "name": "John Doe",
  "email": "<john@example.com>",
  "age": 30
}

API Requirements

1. Create User
Endpoint
POST /users

Request Body
{
  "name": "Alice",
  "email": "<alice@example.com>",
  "age": 25
}

Behavior
Automatically generate a unique UUID as the id
Save the user to users.json
Response (201 Created)
{
  "message": "User created successfully",
  "user": {
    "id": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
    "name": "Alice",
    "email": "<alice@example.com>",
    "age": 25
  }
}

2. Get All Users
Endpoint
GET /users

Response
{
  "users": [
    {
      "id": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
      "name": "Alice",
      "email": "alice@example.com",
      "age": 25
    }
  ]
}

3. Get Single User
Endpoint
GET /users/<uuid>

Response (200 OK)
{
  "id": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
  "name": "Alice",
  "email": "<alice@example.com>",
  "age": 25
}

Error (404)
{
  "error": "User not found"
}

Error (400) - Invalid UUID format
{
  "error": "Invalid UUID format"
}

4. Update User
Endpoint
PATCH /users/<uuid>

Request Body
{
  "name": "Alice Smith",
  "age": 26
}

Behavior
Update only the provided fields
Keep other fields unchanged
Response
{
  "message": "User updated successfully",
  "user": {
    "id": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d",
    "name": "Alice Smith",
    "email": "<alice@example.com>",
    "age": 26
  }
}

5. Delete User
Endpoint
DELETE /users/<uuid>

Response
{
  "message": "User deleted successfully"
}

Validation & Error Handling
The application should handle:
Missing request body fields
Invalid JSON
Non-existent user UUIDs
Invalid UUID format in URL
Empty JSON file
Concurrent file access safety (basic handling)
Use proper HTTP status codes:
200 OK
201 Created
400 Bad Request
404 Not Found
500 Internal Server Error
Implementation Notes
Use Python's built-in uuid module to generate UUIDs (e.g., uuid.uuid4())
Convert UUIDs to strings when storing in JSON
Validate UUID format before performing operations (consider using uuid.UUID() for validation)

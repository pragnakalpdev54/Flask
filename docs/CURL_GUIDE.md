# cURL Command Guide for API Testing

## Overview

cURL (Client URL) is a command-line tool for making HTTP requests. It's the most reliable way to test REST APIs and is available on all operating systems. This guide will teach you how to use cURL to test your Flask APIs.

## Table of Contents

1. [What is cURL?](#what-is-curl)
2. [Installation](#installation)
3. [Basic Requests](#basic-requests)
4. [Headers and Content Types](#headers-and-content-types)
5. [Authentication](#authentication)
6. [File Uploads](#file-uploads)
7. [Advanced Usage](#advanced-usage)
8. [Examples by Flask Level](#examples-by-flask-level)
9. [Troubleshooting](#troubleshooting)

## What is cURL?

cURL is a command-line tool that allows you to transfer data to or from a server using various protocols (HTTP, HTTPS, FTP, etc.). For API testing, we primarily use it for HTTP/HTTPS requests.

**Why use cURL?**

- Available on all platforms
- No GUI needed - works in terminal
- Scriptable and automatable
- Shows raw HTTP requests/responses
- Industry standard for API testing

## Installation

### Linux

Most Linux distributions come with cURL pre-installed. If not:

```bash
# Ubuntu/Debian
sudo apt install curl

# CentOS/RHEL
sudo yum install curl

# Verify installation
curl --version
```

### macOS

cURL comes pre-installed. Verify:

```bash
curl --version
```

### Windows

#### Option 1: Windows 10/11 (Built-in)

- cURL is included in Windows 10 version 1803 and later
- Open PowerShell or Command Prompt and type: `curl --version`

#### Option 2: Download

- Download from [curl.se/windows/](https://curl.se/windows/)
- Or use Git Bash (includes cURL)

#### Option 3: Using Chocolatey

```bash
choco install curl
```

## Basic Requests

### GET Request

The simplest request - fetching data:

```bash
# Basic GET request
curl http://127.0.0.1:5000/api/tasks/

# With verbose output (see headers)
curl -v http://127.0.0.1:5000/api/tasks/

# Pretty print JSON response (requires jq)
curl http://127.0.0.1:5000/api/tasks/ | jq

# Save response to file
curl http://127.0.0.1:5000/api/tasks/ -o response.json

# Follow redirects
curl -L http://127.0.0.1:5000/api/tasks/
```

### POST Request

Creating new resources:

```bash
# Basic POST with JSON data
curl -X POST http://127.0.0.1:5000/api/tasks/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn Flask", "desc": "Complete the learning guide", "completed": false}'

# POST with data from file
curl -X POST http://127.0.0.1:5000/api/tasks/ \
  -H "Content-Type: application/json" \
  -d @data.json

# POST with form data
curl -X POST http://127.0.0.1:5000/api/tasks/ \
  -d "title=Learn Flask" \
  -d "desc=Complete the learning guide" \
  -d "completed=false"
```

> [!IMPORTANT]
> **Windows PowerShell Users**: The syntax above uses Unix/Linux/Mac conventions. See the [Windows PowerShell section](#windows-powershell-syntax) below for correct syntax on Windows.

### PUT Request

Full update of a resource:

```bash
curl -X PUT http://127.0.0.1:5000/api/tasks/1/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Task", "desc": "Updated description", "completed": true}'
```

### PATCH Request

Partial update of a resource:

```bash
curl -X PATCH http://127.0.0.1:5000/api/tasks/1/ \
  -H "Content-Type: application/json" \
  -d '{"completed": true}'
```

### DELETE Request

Deleting a resource:

```bash
curl -X DELETE http://127.0.0.1:5000/api/tasks/1/
```

## Windows PowerShell Syntax

> [!WARNING]
> **PowerShell uses different syntax than Unix shells!** Copy-pasting Unix examples will fail.

### Key Differences

| Feature | Unix/Linux/Mac | Windows PowerShell |
|---------|----------------|---------------------|
| Line continuation | `\` backslash | `` ` `` backtick |
| JSON quotes | Single quotes `'` | Escaped double quotes |
| Variables | `$VAR` | `$env:VAR` |

### GET Request (Same on Windows)

```powershell
# GET works the same
curl http://127.0.0.1:5000/api/tasks/
```

### POST Request with JSON

#### Option 1: Single Line (Recommended for Beginners)

```powershell
curl -X POST http://127.0.0.1:5000/api/tasks/ -H "Content-Type: application/json" -d "{`"title`":`"Learn Flask`",`"completed`":false}"
```

#### Option 2: Multi-line with Backtick

```powershell
curl -X POST http://127.0.0.1:5000/api/tasks/ `
  -H "Content-Type: application/json" `
  -d "{`"title`":`"Learn Flask`",`"completed`":false}"
```

#### Option 3: Using Here-String (Cleanest)

```powershell
$json = @"
{
  "title": "Learn Flask",
  "desc": "Complete the guide",
  "completed": false
}
"@

curl -X POST http://127.0.0.1:5000/api/tasks/ `
  -H "Content-Type: application/json" `
  -d $json
```

### PUT Request

```powershell
curl -X PUT http://127.0.0.1:5000/api/tasks/1/ `
  -H "Content-Type: application/json" `
  -d "{`"title`":`"Updated Task`",`"completed`":true}"
```

### PATCH Request

```powershell
curl -X PATCH http://127.0.0.1:5000/api/tasks/1/ `
  -H "Content-Type: application/json" `
  -d "{`"completed`":true}"
```

### DELETE Request (Same on Windows)

```powershell
curl -X DELETE http://127.0.0.1:5000/api/tasks/1/
```

### JWT Authentication

```powershell
# Get token
$response = curl -X POST http://127.0.0.1:5000/auth/login `
  -H "Content-Type: application/json" `
  -d "{`"email`":`"user@example.com`",`"password`":`"pass123`"}" | ConvertFrom-Json

$token = $response.access_token

# Use token
curl -H "Authorization: Bearer $token" http://127.0.0.1:5000/api/tasks/
```

> [!TIP]
> **Quick Tip**: Use the **here-string** method (`@" ... "@`) for complex JSON in PowerShell. It's much cleaner than escaping quotes!

## Headers and Content Types

### Setting Headers

```bash
# Single header
curl -H "Content-Type: application/json" http://127.0.0.1:5000/api/tasks/

# Multiple headers
curl -H "Content-Type: application/json" \
     -H "Accept: application/json" \
     http://127.0.0.1:5000/api/tasks/

# Custom header
curl -H "X-Custom-Header: my-value" http://127.0.0.1:5000/api/tasks/
```

### Common Headers

```bash
# JSON content
curl -H "Content-Type: application/json" ...

# XML content
curl -H "Content-Type: application/xml" ...

# Accept only JSON response
curl -H "Accept: application/json" ...

# User agent
curl -H "User-Agent: MyApp/1.0" ...
```

## Authentication

### Basic Authentication

```bash
# Basic auth (username:password)
curl -u username:password http://127.0.0.1:5000/api/tasks/

# Or with header
curl -H "Authorization: Basic $(echo -n 'username:password' | base64)" \
  http://127.0.0.1:5000/api/tasks/
```

### Bearer Token (JWT)

```bash
# With Bearer token
curl -H "Authorization: Bearer your_jwt_token_here" \
  http://127.0.0.1:5000/api/tasks/

# Store token in variable (bash)
TOKEN="your_jwt_token_here"
curl -H "Authorization: Bearer $TOKEN" \
  http://127.0.0.1:5000/api/tasks/
```

### Getting JWT Token

```bash
# Login to get token (example: your Flask /auth/login endpoint)
curl -X POST http://127.0.0.1:5000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com", "password": "password123"}' \
  | jq -r '.access_token'

# Save token to variable
TOKEN=$(curl -s -X POST http://127.0.0.1:5000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com", "password": "password123"}' \
  | jq -r '.access_token')

# Use token in subsequent requests
curl -H "Authorization: Bearer $TOKEN" \
  http://127.0.0.1:5000/api/tasks/
```

## File Uploads

### Upload Single File

```bash
# Upload file
curl -X POST http://127.0.0.1:5000/api/tasks/1/upload/ \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@/path/to/file.pdf"

# Upload with additional data
curl -X POST http://127.0.0.1:5000/api/tasks/1/upload/ \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@/path/to/file.pdf" \
  -F "description=Task attachment"
```

### Upload Multiple Files

```bash
curl -X POST http://127.0.0.1:5000/api/tasks/1/upload/ \
  -H "Authorization: Bearer $TOKEN" \
  -F "files=@/path/to/file1.pdf" \
  -F "files=@/path/to/file2.jpg"
```

## Advanced Usage

### Verbose Mode

See full request/response details:

```bash
curl -v http://127.0.0.1:5000/api/tasks/

# Shows:
# - Request headers
# - Response headers
# - SSL handshake (if HTTPS)
# - Connection details
```

### Silent Mode

Suppress progress meter:

```bash
curl -s http://127.0.0.1:5000/api/tasks/
```

### Show Only Response Headers

```bash
curl -I http://127.0.0.1:5000/api/tasks/

# Or
curl -D - http://127.0.0.1:5000/api/tasks/
```

### Follow Redirects

```bash
curl -L http://127.0.0.1:5000/api/tasks/
```

### Timeout

```bash
# Set timeout to 10 seconds
curl --max-time 10 http://127.0.0.1:5000/api/tasks/

# Connection timeout
curl --connect-timeout 5 http://127.0.0.1:5000/api/tasks/
```

### Save Response Headers

```bash
# Save headers to file
curl -D headers.txt http://127.0.0.1:5000/api/tasks/

# Save both headers and body
curl -D headers.txt -o response.json http://127.0.0.1:5000/api/tasks/
```

### Pretty Print JSON

```bash
# Using jq (install: sudo apt install jq)
curl http://127.0.0.1:5000/api/tasks/ | jq

# Using python
curl http://127.0.0.1:5000/api/tasks/ | python -m json.tool
```

## Examples by Flask Level

### Level 1: Basic CRUD Operations

```bash
# GET - List all tasks
curl http://127.0.0.1:5000/api/tasks/

# GET - Retrieve single task
curl http://127.0.0.1:5000/api/tasks/1/

# POST - Create new task
curl -X POST http://127.0.0.1:5000/api/tasks/ \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Learn Flask",
    "desc": "Complete Level 1",
    "completed": false
  }'

# PUT - Update task
curl -X PUT http://127.0.0.1:5000/api/tasks/1/ \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Learn Flask - Updated",
    "desc": "Completed Level 1",
    "completed": true
  }'

# PATCH - Partial update
curl -X PATCH http://127.0.0.1:5000/api/tasks/1/ \
  -H "Content-Type: application/json" \
  -d '{"completed": true}'

# DELETE - Delete task
curl -X DELETE http://127.0.0.1:5000/api/tasks/1/
```

### Level 2: Authenticated Requests

```bash
# Step 1: Get JWT token from your Flask auth endpoint
TOKEN=$(curl -s -X POST http://127.0.0.1:5000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com", "password": "password123"}' \
  | python -c "import sys, json; print(json.load(sys.stdin)['access_token'])")

# Step 2: Use token in requests
curl -H "Authorization: Bearer $TOKEN" \
  http://127.0.0.1:5000/api/tasks/

# Create task with authentication
curl -X POST http://127.0.0.1:5000/api/tasks/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "Private Task",
    "desc": "Only visible to authenticated users",
    "completed": false
  }'
```

### Level 3: Complex Nested Data

```bash
# POST with nested relationships (example blog API)
curl -X POST http://127.0.0.1:5000/api/posts/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "My First Post",
    "content": "This is the content",
    "author_id": 1,
    "tags": ["flask", "api"],
    "comments": [
      {
        "content": "Great post!",
        "author_name": "Bob"
      }
    ]
  }'

# GET with nested data
curl -H "Authorization: Bearer $TOKEN" \
  http://127.0.0.1:5000/api/posts/1/
```

### Level 4: File Uploads and Versioned APIs

```bash
# File upload
curl -X POST http://127.0.0.1:5000/api/v1/tasks/1/attachments/ \
  -H "Authorization: Bearer $TOKEN" \
  -F "file=@/path/to/document.pdf" \
  -F "description=Task attachment"

# Versioned API (v1)
curl http://127.0.0.1:5000/api/v1/tasks/

# Versioned API (v2)
curl http://127.0.0.1:5000/api/v2/tasks/

# With version header
curl -H "Accept: application/json; version=2" \
  http://127.0.0.1:5000/api/tasks/
```

## Common cURL Flags Reference

| Flag | Description |
|------|-------------|
| `-X METHOD` | HTTP method (GET, POST, PUT, DELETE, etc.) |
| `-H "Header: value"` | Add header |
| `-d "data"` | Send data in request body |
| `-F "name=value"` | Form data (for file uploads) |
| `-u user:pass` | Basic authentication |
| `-v` | Verbose output |
| `-s` | Silent mode (no progress) |
| `-i` | Include response headers |
| `-I` | Head request (headers only) |
| `-L` | Follow redirects |
| `-o file` | Save output to file |
| `-O` | Save with remote filename |
| `-D file` | Save headers to file |
| `--max-time SEC` | Maximum time for request |
| `--connect-timeout SEC` | Connection timeout |
| `-k` | Allow insecure SSL connections |
| `-c file` | Save cookies to file |
| `-b file` | Load cookies from file |

## Converting Postman to cURL

### In Postman

1. Open your request in Postman
2. Click "Code" button (bottom left)
3. Select "cURL" from dropdown
4. Copy the generated cURL command

### Manual Conversion Tips

- **Headers**: Each header becomes `-H "Header: value"`
- **Body**: JSON becomes `-d '{"key": "value"}'`
- **Auth**: Bearer token becomes `-H "Authorization: Bearer token"`
- **Files**: Use `-F "file=@path/to/file"`

## Troubleshooting

### Common Issues

#### 1. Connection Refused

```bash
# Check if server is running
curl http://127.0.0.1:5000/api/tasks/

# Error: Connection refused
# Solution: Start your Flask server
# Example:
# python app.py
# or:
# flask run
```

#### 2. 404 Not Found

```bash
# Check URL is correct
curl -v http://127.0.0.1:5000/api/tasks/

# Verify route in Flask:
# - Check your @app.route or @blueprint.route path
# - Ensure you are using the correct HTTP method (GET/POST/...)
```

#### 3. 401 Unauthorized

```bash
# Missing or invalid token
# Solution: Get new token from your Flask auth endpoint
TOKEN=$(curl -s -X POST http://127.0.0.1:5000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "pass"}' \
  | jq -r '.access_token')
```

#### 4. 400 Bad Request

```bash
# Check JSON syntax
curl -X POST http://127.0.0.1:5000/api/tasks/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Test"}'  # Valid JSON

# Use verbose mode to see error details
curl -v -X POST http://127.0.0.1:5000/api/tasks/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Test"}'
```

#### 5. SSL Certificate Errors

```bash
# For development (not recommended for production)
curl -k https://api.example.com/

# Or specify certificate
curl --cacert /path/to/cert.pem https://api.example.com/
```

### Debugging Tips

1. **Use verbose mode**: `curl -v` shows full request/response
2. **Check response headers**: `curl -I` shows only headers
3. **Save responses**: `curl -o response.json` to inspect later
4. **Test with simple request first**: Start with GET before POST
5. **Validate JSON**: Use `jq` or `python -m json.tool`

## Practice Exercises

### Exercise 1: Basic CRUD

```bash
# 1. List all tasks
curl http://127.0.0.1:5000/api/tasks/

# 2. Create a task
curl -X POST http://127.0.0.1:5000/api/tasks/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Exercise Task", "completed": false}'

# 3. Get the created task (use ID from step 2)
curl http://127.0.0.1:5000/api/tasks/1/

# 4. Update the task
curl -X PATCH http://127.0.0.1:5000/api/tasks/1/ \
  -H "Content-Type: application/json" \
  -d '{"completed": true}'

# 5. Delete the task
curl -X DELETE http://127.0.0.1:5000/api/tasks/1/
```

### Exercise 2: Authentication Flow

```bash
# 1. Register a user
curl -X POST http://127.0.0.1:5000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "testuser", "password": "testpass123", "email": "test@example.com"}'

# 2. Get JWT token
TOKEN=$(curl -s -X POST http://127.0.0.1:5000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "testuser@example.com", "password": "testpass123"}' \
  | jq -r '.access_token')

# 3. Use token to access protected endpoint
curl -H "Authorization: Bearer $TOKEN" \
  http://127.0.0.1:5000/api/tasks/
```

## Next Steps

Now that you know cURL basics:

1. Practice with your Level 1 API
2. Use cURL to test all endpoints
3. Create a script with common requests
4. Refer back to this guide as you progress through levels

For more information:

- [cURL Official Documentation](https://curl.se/docs/)
- [HTTPie](https://httpie.io/) - Alternative to cURL with better UX
- [Postman](https://www.postman.com/) - GUI alternative

---

**Ready to test your APIs?** Use these cURL commands as you work through each level of the [Flask Learning Guide](00_LEARNING_GUIDE.md)!

`` ``

# REST API Design

REST (Representational State Transfer) is a convention for designing web APIs. It uses HTTP methods and URLs to represent operations on resources.

## Resources and URLs

A resource is a thing your API manages: users, orders, products. URLs identify resources.

| URL | Resource |
|-----|----------|
| `/users` | The collection of all users |
| `/users/42` | The user with ID 42 |
| `/users/42/orders` | Orders belonging to user 42 |

URLs are nouns, not verbs. The HTTP method provides the verb.

## CRUD Mapping

| Operation | HTTP Method | URL | Description |
|-----------|------------|-----|-------------|
| Create | POST | `/users` | Create a new user |
| Read (list) | GET | `/users` | Get all users |
| Read (one) | GET | `/users/42` | Get user 42 |
| Update | PUT | `/users/42` | Replace user 42 entirely |
| Partial update | PATCH | `/users/42` | Update specific fields of user 42 |
| Delete | DELETE | `/users/42` | Delete user 42 |

## Request and Response Examples

### Create a user

Request:
```
POST /users
Content-Type: application/json

{"name": "Alice", "email": "alice@example.com"}
```

Response:
```
201 Created
Content-Type: application/json
Location: /users/3

{"id": 3, "name": "Alice", "email": "alice@example.com"}
```

Note: `201 Created` (not `200 OK`) for resource creation. `Location` header tells the client where the new resource is.

### List users

Request:
```
GET /users
```

Response:
```
200 OK
Content-Type: application/json

[
  {"id": 1, "name": "Alice", "email": "alice@example.com"},
  {"id": 2, "name": "Bob", "email": "bob@example.com"}
]
```

### Get one user

Request:
```
GET /users/1
```

Response:
```
200 OK
Content-Type: application/json

{"id": 1, "name": "Alice", "email": "alice@example.com"}
```

### User not found

Request:
```
GET /users/999
```

Response:
```
404 Not Found
Content-Type: application/json

{"error": "User not found"}
```

### Delete a user

Request:
```
DELETE /users/1
```

Response:
```
204 No Content
```

No body needed for a successful deletion. `204` means "success, no content to return."

## Status Code Selection

| Scenario | Status code |
|----------|------------|
| Successful read | 200 OK |
| Resource created | 201 Created |
| Successful delete | 204 No Content |
| Bad request (invalid JSON) | 400 Bad Request |
| Not authenticated | 401 Unauthorized |
| Not authorized (wrong role) | 403 Forbidden |
| Resource does not exist | 404 Not Found |
| Validation error | 422 Unprocessable Entity |
| Server error | 500 Internal Server Error |

## Pagination

Return large collections in pages. Never return unbounded lists.

Request:
```
GET /users?page=2&per_page=20
```

Response:
```json
{
  "data": [
    {"id": 21, "name": "..."},
    {"id": 22, "name": "..."}
  ],
  "page": 2,
  "per_page": 20,
  "total": 150,
  "total_pages": 8
}
```

Common pagination strategies:

| Strategy | How it works | When to use |
|----------|-------------|-------------|
| Offset | `?page=2&per_page=20` (skip 20, take 20) | Small to medium datasets |
| Cursor | `?cursor=abc123` (return items after this key) | Large datasets, real-time data |
| Keyset | `?after_id=100` (return items with ID > 100) | Ordered data |

## Error Response Format

Use a consistent error structure:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "details": [
      {"field": "email", "message": "This field is required"}
    ]
  }
}
```

## Naming Conventions

| Rule | Example |
|------|---------|
| Use plural nouns | `/users` not `/user` |
| Use lowercase | `/users` not `/Users` |
| Use hyphens for multi-word | `/user-profiles` not `/userProfiles` |
| Nest for relationships | `/users/42/orders` |
| Keep nesting shallow | Max 2 levels. Beyond that, use query params. |

## Next

The next file builds your first API with Axum. You will implement these REST patterns in Rust code.

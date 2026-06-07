`[Mid]`

# Capstone Project: Memory-Safe Task API

Build a complete backend API that demonstrates everything you have learned. This project is your proof of skill.

## Project Spec

Build a **task management API** where users can create, read, update, and delete tasks. Users authenticate with JWT. The API connects to PostgreSQL. It is tested, logged, and deployed in Docker.

## Requirements

### Data Model

```
User:
  id: UUID (primary key)
  email: text (unique)
  password_hash: text
  created_at: timestamp

Task:
  id: UUID (primary key)
  user_id: UUID (foreign key -> users.id)
  title: text
  description: text (optional)
  status: enum (todo, in_progress, done)
  created_at: timestamp
  updated_at: timestamp
```

### API Endpoints

| Method | Path | Auth? | Description |
|--------|------|-------|-------------|
| POST | /auth/register | No | Create account. Body: `{email, password}`. Returns JWT. |
| POST | /auth/login | No | Login. Body: `{email, password}`. Returns JWT. |
| GET | /tasks | Yes | List user's tasks. Support `?status=todo` filter. Paginate with `?page=1&per_page=20`. |
| POST | /tasks | Yes | Create task. Body: `{title, description?, status?}`. |
| GET | /tasks/{id} | Yes | Get one task. Only if owned by the user. |
| PUT | /tasks/{id} | Yes | Update task. Body: `{title?, description?, status?}`. |
| DELETE | /tasks/{id} | Yes | Delete task. Only if owned by the user. |
| GET | /health | No | Health check. Returns 200 OK. |

### Technical Requirements

1. **Axum** for routing and handlers
2. **SQLx** with compile-time checked queries against PostgreSQL
3. **JWT authentication** with Bearer tokens. Middleware protects authenticated routes.
4. **bcrypt** for password hashing
5. **tracing** for structured JSON logging. Every handler is instrumented.
6. **Error handling** with `thiserror` for custom error types. No `unwrap()` in handlers.
7. **Tests**: unit tests for business logic, handler tests for API routes, integration test for the full auth + CRUD flow
8. **Docker**: multi-stage build. Final image under 20 MB. Health check configured.

### Error Responses

All errors return JSON:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required"
  }
}
```

Use appropriate status codes: 400, 401, 403, 404, 409 (duplicate email), 422, 500.

### Validation Rules

- Email: required, valid format
- Password: minimum 8 characters
- Task title: required, max 200 characters
- Task status: must be one of `todo`, `in_progress`, `done`
- User can only access their own tasks

## Suggested Implementation Order

1. Set up project with Cargo, add dependencies
2. Create database migrations (users table, tasks table)
3. Implement data models with `serde` and `FromRow`
4. Implement error types with `thiserror`
5. Build auth routes: register and login
6. Build auth middleware
7. Build task CRUD routes
8. Add pagination and filtering
9. Add tracing and instrumentation
10. Write unit tests for validation
11. Write handler tests for each endpoint
12. Write one integration test covering register -> create task -> list tasks -> delete task
13. Create Dockerfile with multi-stage build
14. Test the Docker container locally

## Dependencies

```toml
[dependencies]
axum = "0.8"
tokio = { version = "1", features = ["rt-multi-thread", "macros", "signal"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "uuid", "chrono"] }
uuid = { version = "1", features = ["v7", "serde"] }
chrono = { version = "0.4", features = ["serde"] }
jsonwebtoken = "9"
bcrypt = "0.16"
thiserror = "2"
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }
tower-http = { version = "0.6", features = ["trace"] }
```

## Success Criteria

Your project is complete when:

1. All endpoints work as specified. Test with `curl`.
2. Unauthenticated requests to protected routes return 401.
3. Users cannot access other users' tasks.
4. All SQL queries are compile-time checked (use `query_as!`).
5. Structured JSON logs appear for every request.
6. `cargo test` passes with at least 10 tests.
7. `docker build` produces an image under 20 MB.
8. `docker run` starts the API, and all endpoints work inside the container.

## Stretch Goals

If you finish early, add:

- Rate limiting middleware (max 100 requests per minute per IP)
- OpenTelemetry tracing export to Jaeger or Zipkin
- Database connection retry logic on startup
- Configuration via a `config.toml` file with environment variable overrides
- A `/metrics` endpoint exposing Prometheus metrics
- Background task that deletes tasks marked as done for more than 30 days

## What You Have Proven

By completing this project, you have demonstrated:

- Memory-safe code enforced by the Rust compiler
- Async backend with Tokio and Axum
- Database integration with compile-time checked queries
- Authentication and authorization
- Structured logging
- Comprehensive testing
- Containerized deployment

You can build production Rust backends.

`[Mid]`

# Testing

Rust has built-in test support. No test framework to install. Write tests in the same file as your code or in a separate tests directory.

## Unit Tests

Unit tests live in the same file, in a `#[cfg(test)]` module:

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn divide(a: i32, b: i32) -> Option<i32> {
    if b == 0 {
        None
    } else {
        Some(a / b)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }

    #[test]
    fn test_add_negative() {
        assert_eq!(add(-1, 1), 0);
    }

    #[test]
    fn test_divide() {
        assert_eq!(divide(10, 2), Some(5));
    }

    #[test]
    fn test_divide_by_zero() {
        assert_eq!(divide(10, 0), None);
    }
}
```

Step by step:
- `#[cfg(test)]` — this module is only compiled when running tests, not in production builds
- `mod tests` — convention: name the module `tests`
- `use super::*` — bring the parent module's items into scope
- `#[test]` — marks a function as a test
- `assert_eq!` — assert two values are equal. If not, the test fails with a message showing both values.

### Useful Assertions

| Macro | Purpose |
|-------|---------|
| `assert!(expr)` | Assert expression is true |
| `assert_eq!(a, b)` | Assert equality |
| `assert_ne!(a, b)` | Assert inequality |
| `assert!(expr, "message: {}", val)` | Assert with custom message |

### Testing Error Cases

```rust
fn parse_age(input: &str) -> Result<u32, String> {
    let age: u32 = input.parse().map_err(|_| "Not a number".to_string())?;
    if age > 150 {
        Err("Age too high".to_string())
    } else {
        Ok(age)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_valid_age() {
        assert_eq!(parse_age("25"), Ok(25));
    }

    #[test]
    fn test_invalid_input() {
        assert_eq!(parse_age("abc"), Err("Not a number".to_string()));
    }

    #[test]
    fn test_age_too_high() {
        assert_eq!(parse_age("200"), Err("Age too high".to_string()));
    }
}
```

## Async Tests

Use `#[tokio::test]` for async functions:

```rust
async fn fetch_user(pool: &sqlx::PgPool, id: u32) -> Option<String> {
    sqlx::query_scalar("SELECT name FROM users WHERE id = $1")
        .bind(id)
        .fetch_optional(pool)
        .await
        .unwrap()
        .flatten()
}

#[tokio::test]
async fn test_fetch_user() {
    let pool = setup_test_db().await;
    let name = fetch_user(&pool, 1).await;
    assert_eq!(name, Some("Alice".to_string()));
}
```

Step by step:
- `#[tokio::test]` — creates a Tokio runtime for this test, similar to `#[tokio::main]`
- The test function is async and can use `.await`

## Testing Axum Handlers

Test your API handlers without starting a real HTTP server:

```rust
use axum::{
    body::Body,
    http::{Request, StatusCode},
    Router,
};
use tower::ServiceExt;

async fn setup_app() -> Router {
    // create your app with a test database
    let pool = create_test_pool().await;
    Router::new()
        .route("/users", get(list_users).post(create_user))
        .with_state(pool)
}

#[tokio::test]
async fn test_list_users_empty() {
    let app = setup_app().await;

    let response = app
        .oneshot(
            Request::builder()
                .uri("/users")
                .body(Body::empty())
                .unwrap(),
        )
        .await
        .unwrap();

    assert_eq!(response.status(), StatusCode::OK);
}

#[tokio::test]
async fn test_create_user() {
    let app = setup_app().await;

    let response = app
        .oneshot(
            Request::builder()
                .method("POST")
                .uri("/users")
                .header("Content-Type", "application/json")
                .body(Body::from(r#"{"name":"Alice","email":"alice@test.com"}"#))
                .unwrap(),
        )
        .await
        .unwrap();

    assert_eq!(response.status(), StatusCode::OK);

    let body = axum::body::to_bytes(response.into_body(), usize::MAX)
        .await
        .unwrap();
    let user: serde_json::Value = serde_json::from_slice(&body).unwrap();
    assert_eq!(user["name"], "Alice");
}
```

Step by step:
- `app.oneshot(...)` — send a single request through the router without starting a real server
- `Request::builder()` — build an HTTP request programmatically
- `Body::from(...)` — set the request body
- `assert_eq!(response.status(), StatusCode::OK)` — verify the status code
- Parse the response body as JSON and verify the content

## Integration Tests

Integration tests live in the `tests/` directory at the project root:

```
my-api/
  src/
    main.rs
    lib.rs
  tests/
    users_test.rs        # each file is a separate test crate
    health_test.rs
```

`tests/users_test.rs`:

```rust
use my_api::create_app;

#[tokio::test]
async fn test_users_crud() {
    let app = create_app().await;

    // create
    let response = app
        .clone()
        .oneshot(
            Request::builder()
                .method("POST")
                .uri("/users")
                .header("Content-Type", "application/json")
                .body(Body::from(r#"{"name":"Bob","email":"bob@test.com"}"#))
                .unwrap(),
        )
        .await
        .unwrap();

    assert_eq!(response.status(), StatusCode::OK);
}
```

Integration tests can only access items marked `pub` in your library crate.

## Running Tests

```bash
cargo test                  # run all tests
cargo test test_add         # run tests matching "test_add"
cargo test -- --nocapture   # show println! output
cargo test -- --show-output # show test output even on success
```

## Test Strategy

| Level | What it tests | Speed |
|-------|--------------|-------|
| Unit | Individual functions | Very fast (microseconds) |
| Handler | API routes via oneshot | Fast (milliseconds) |
| Integration | Full request flow with test DB | Slower (setup/teardown) |

Write many unit tests, some handler tests, and a few integration tests for critical paths.

## Next

The next file covers logging and monitoring with the `tracing` crate.

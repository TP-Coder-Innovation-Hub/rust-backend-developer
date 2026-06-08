``

# Logging and Monitoring

Logs tell you what your application is doing. Structured logs let you search and filter them efficiently. The `tracing` crate is the standard for Rust.

## Why tracing, Not log

The `log` crate provides basic logging (info!, warn!, error!). The `tracing` crate extends this with **spans** — contextual scopes that attach metadata to all log events within them.

| Feature | `log` | `tracing` |
|---------|-------|-----------|
| Log levels | Yes | Yes |
| Structured fields | No | Yes |
| Spans (context) | No | Yes |
| Async-aware | No | Yes |
| OpenTelemetry integration | No | Yes |

For a backend service, you need spans and structured fields. Use `tracing`.

## Setup

Add to `Cargo.toml`:

```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }
```

Initialize in `main.rs`:

```rust
use tracing_subscriber::{fmt, EnvFilter};

#[tokio::main]
async fn main() {
    fmt()
        .with_env_filter(
            EnvFilter::try_from_default_env()
                .unwrap_or_else(|_| EnvFilter::new("info")),
        )
        .json()            // output JSON format
        .init();

    tracing::info!("Server starting");
}
```

Step by step:
- `fmt()` — create a formatting subscriber
- `with_env_filter(...)` — control log levels via `RUST_LOG` environment variable
- `EnvFilter::try_from_default_env()` — read `RUST_LOG` from environment
- Fallback to `"info"` level if `RUST_LOG` is not set
- `.json()` — output structured JSON logs (for production). Remove this line for human-readable output during development.
- `.init()` — set as the global subscriber

Set log level at runtime:

```bash
RUST_LOG=debug cargo run       # show debug and above
RUST_LOG=my_api=trace cargo run  # show trace for your crate only
RUST_LOG=sqlx=warn cargo run   # suppress sqlx noise
```

## Log Levels

| Level | When to use |
|-------|------------|
| `error!` | Something failed that affects the user |
| `warn!` | Something unexpected but recoverable |
| `info!` | Normal operations (server started, request handled) |
| `debug!` | Detailed info for debugging (query parameters, intermediate values) |
| `trace!` | Very detailed (function entry/exit, full request/response bodies) |

## Basic Logging

```rust
use tracing::{info, warn, error, debug};

async fn handle_request(user_id: u32) -> Result<String, String> {
    info!(user_id = user_id, "Processing request");

    let user = fetch_user(user_id).await;
    match user {
        Some(u) => {
            debug!(name = %u.name, "User found");
            Ok(u.name)
        }
        None => {
            warn!(user_id = user_id, "User not found");
            Err("Not found".into())
        }
    }
}
```

Step by step:
- `info!(user_id = user_id, "Processing request")` — log with a structured field `user_id`
- `debug!(name = %u.name, ...)` — the `%` forces Display formatting. Without it, `name` uses Debug formatting.
- Structured fields appear in JSON output as key-value pairs, making them searchable

JSON output:
```json
{"timestamp":"2026-06-07T10:30:00Z","level":"INFO","message":"Processing request","user_id":42}
```

## Spans

Spans add context to all events within a scope:

```rust
use tracing::{info, instrument, span, Level};

async fn process_order(order_id: u32) -> Result<(), String> {
    let span = span!(Level::INFO, "process_order", order_id = order_id);
    let _enter = span.enter();

    info!("Validating order");
    validate_order(order_id)?;

    info!("Charging payment");
    charge_payment(order_id)?;

    info!("Order complete");
    Ok(())
}
```

All `info!` calls inside this function include the `order_id` field automatically.

The `#[instrument]` attribute does this automatically:

```rust
#[tracing::instrument(skip(pool))]
async fn create_user(pool: &PgPool, name: String, email: String) -> Result<User, AppError> {
    info!("Creating user");
    let user = sqlx::query_as::<_, User>(
        "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *"
    )
    .bind(&name)
    .bind(&email)
    .fetch_one(pool)
    .await?;

    info!(user_id = %user.id, "User created");
    Ok(user)
}
```

Step by step:
- `#[instrument]` — creates a span with the function name and all parameters
- `skip(pool)` — do not log the pool parameter (it is not useful in logs)
- All `info!` calls inside include `name` and `email` fields automatically
- The span records when the function starts and when it returns

## Logging in Axum

Add tracing middleware to your router:

```rust
use axum::Router;
use tower_http::trace::TraceLayer;

let app = Router::new()
    .route("/users", get(list_users))
    .layer(TraceLayer::new_for_http());
```

This logs every incoming HTTP request with method, path, status code, and latency.

## Structured Error Logging

```rust
use tracing::error;

async fn handler() -> Result<Json<User>, (StatusCode, String)> {
    let user = fetch_user(1).await.map_err(|e| {
        error!(error = %e, "Failed to fetch user");
        (StatusCode::INTERNAL_SERVER_ERROR, "Internal error".into())
    })?;

    Ok(Json(user))
}
```

- Log the full error internally with `error!`
- Return a generic message to the client (never expose internal details)

## Production Logging Tips

| Practice | Why |
|----------|-----|
| Use JSON format | Parsable by log aggregators (Datadog, ELK, CloudWatch) |
| Set `RUST_LOG=info` in production | Avoid log noise from dependencies |
| Use spans for request context | Trace a request across multiple functions |
| Log request IDs | Correlate logs from the same request |
| Never log secrets | Mask tokens, passwords, API keys |
| Use `error!` sparingly | Alerts trigger on errors. False alerts erode trust. |

## Next

The next file covers deployment: building a static binary, Docker packaging, and creating small production images.

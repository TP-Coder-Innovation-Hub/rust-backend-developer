`[Mid]`

# Your First API

Build a minimal REST API with Axum. You will create routes, handlers, and return JSON. Every line is explained.

## Setup

Create a new project:

```bash
cargo new my-api
cd my-api
```

Replace `Cargo.toml` dependencies:

```toml
[dependencies]
axum = "0.8"
tokio = { version = "1", features = ["rt-multi-thread", "macros", "signal"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

Step by step:
- `axum` — web framework. Handles routing, request parsing, response building.
- `tokio` — async runtime. Axum is async; it needs Tokio to run.
- `serde` — serialization. Converts Rust structs to/from JSON.
- `serde_json` — JSON-specific serde support.

## The Complete Server

Replace `src/main.rs`:

```rust
use axum::{
    Json,
    Router,
    routing::{get, post},
    extract::Path,
};
use serde::{Deserialize, Serialize};
use std::sync::Arc;
use tokio::sync::Mutex;
use std::collections::HashMap;

// ---------- Data types ----------

#[derive(Debug, Serialize, Deserialize, Clone)]
struct User {
    id: u32,
    name: String,
    email: String,
}

#[derive(Debug, Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

// ---------- Shared state ----------

type Db = Arc<Mutex<HashMap<u32, User>>>;

// ---------- Handlers ----------

async fn list_users(state: Db) -> Json<Vec<User>> {
    let db = state.lock().await;
    let users: Vec<User> = db.values().cloned().collect();
    Json(users)
}

async fn get_user(state: Db, Path(id): Path<u32>) -> Json<Option<User>> {
    let db = state.lock().await;
    let user = db.get(&id).cloned();
    Json(user)
}

async fn create_user(state: Db, Json(input): Json<CreateUser>) -> Json<User> {
    let mut db = state.lock().await;
    let id = (db.len() + 1) as u32;
    let user = User {
        id,
        name: input.name,
        email: input.email,
    };
    db.insert(id, user.clone());
    Json(user)
}

// ---------- Main ----------

#[tokio::main]
async fn main() {
    let db: Db = Arc::new(Mutex::new(HashMap::new()));

    let app = Router::new()
        .route("/users", get(list_users).post(create_user))
        .route("/users/{id}", get(get_user))
        .with_state(db);

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Server running on http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}
```

## Line by Line Explanation

### Imports

```rust
use axum::{Json, Router, routing::{get, post}, extract::Path};
```

- `Json` — wrapper that serializes responses to JSON and parses JSON request bodies
- `Router` — defines URL routes and maps them to handler functions
- `get`, `post` — route method handlers
- `Path` — extracts values from the URL path (like `/users/42`)

```rust
use serde::{Deserialize, Serialize};
```

- `Serialize` — converts a Rust struct to JSON (for responses)
- `Deserialize` — converts JSON to a Rust struct (for request bodies)

### Data Types

```rust
#[derive(Debug, Serialize, Deserialize, Clone)]
struct User {
    id: u32,
    name: String,
    email: String,
}
```

- `#[derive(...)]` — auto-implement these traits
- `Serialize` — `User` can be converted to JSON
- `Deserialize` — JSON can be parsed into `User`
- `Clone` — `User` can be cloned (needed because we return it from the Mutex guard)

```rust
#[derive(Debug, Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}
```

- Only `Deserialize` — this type is for incoming data. We never serialize it.
- No `id` field — the server assigns the ID, not the client.

### Shared State

```rust
type Db = Arc<Mutex<HashMap<u32, User>>>;
```

Step by step, from inside out:
- `HashMap<u32, User>` — maps user IDs to users
- `Mutex<...>` — wraps it so only one task can access at a time (thread safety)
- `Arc<...>` — atomic reference counting so multiple handlers share the same data

### Handler: list_users

```rust
async fn list_users(state: Db) -> Json<Vec<User>> {
    let db = state.lock().await;
    let users: Vec<User> = db.values().cloned().collect();
    Json(users)
}
```

- `state: Db` — Axum injects the shared state automatically
- `state.lock().await` — acquire the Mutex lock (async, does not block the runtime)
- `db.values().cloned().collect()` — get all values, clone them (because the Mutex guard borrows them), collect into a Vec
- `Json(users)` — wrap the Vec in `Json` to set the response Content-Type to application/json

### Handler: get_user

```rust
async fn get_user(state: Db, Path(id): Path<u32>) -> Json<Option<User>> {
    let db = state.lock().await;
    let user = db.get(&id).cloned();
    Json(user)
}
```

- `Path(id): Path<u32>` — extract the `{id}` from the URL path as a `u32`
- `db.get(&id)` — look up the user by ID. Returns `Option<&User>`.
- `.cloned()` — convert `Option<&User>` to `Option<User>`
- Returns `Json(Some(user))` or `Json(None)` (which serializes to JSON null)

### Handler: create_user

```rust
async fn create_user(state: Db, Json(input): Json<CreateUser>) -> Json<User> {
    let mut db = state.lock().await;
    let id = (db.len() + 1) as u32;
    let user = User { id, name: input.name, email: input.email };
    db.insert(id, user.clone());
    Json(user)
}
```

- `Json(input): Json<CreateUser>` — parse the request body as JSON into a `CreateUser` struct
- `let mut db` — mutable borrow (we are inserting)
- `db.insert(id, user.clone())` — add the user to the HashMap

### Router Setup

```rust
let app = Router::new()
    .route("/users", get(list_users).post(create_user))
    .route("/users/{id}", get(get_user))
    .with_state(db);
```

- `.route("/users", ...)` — handle requests to `/users`
- `get(list_users).post(create_user)` — GET goes to `list_users`, POST goes to `create_user`
- `.route("/users/{id}", ...)` — handle requests with a path parameter
- `.with_state(db)` — share the database with all handlers

### Server Start

```rust
let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
axum::serve(listener, app).await.unwrap();
```

- Bind to port 3000 on all network interfaces
- Start serving requests with the router

## Test It

```bash
# Run the server
cargo run

# In another terminal:
# Create a user
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@example.com"}'

# List users
curl http://localhost:3000/users

# Get one user
curl http://localhost:3000/users/1
```

## Next

The next file covers database access with SQLx, replacing the in-memory HashMap with a real PostgreSQL database.

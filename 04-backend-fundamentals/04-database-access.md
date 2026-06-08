``

# Database Access

SQLx provides compile-time checked SQL queries for Rust. If your SQL has a syntax error or references a column that does not exist, your code does not compile. This catches bugs before deployment.

## Why SQLx

| Problem | Without SQLx | With SQLx |
|---------|-------------|-----------|
| SQL typo | Runtime error at 3 AM | Compile error during `cargo build` |
| Wrong column name | Runtime error | Compile error |
| Type mismatch | Runtime crash or silent truncation | Compile error |
| Missing migration | Runtime error in production | Caught during development |

SQLx talks to a real database at compile time (or caches the query metadata). It verifies your SQL against the actual schema.

## Setup

Add to `Cargo.toml`:

```toml
[dependencies]
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "uuid"] }
```

You need a running PostgreSQL database. Use Docker:

```bash
docker run -d \
  --name rust-db \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  postgres:17
```

Set the database URL for SQLx compile-time checking:

```bash
export DATABASE_URL="postgres://postgres:secret@localhost/myapp"
```

## Migrations

Create the users table:

```bash
cargo add sqlx-cli
sqlx migrate add create_users_table
```

This creates `migrations/<timestamp>_create_users_table.sql`. Edit it:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Run the migration:

```bash
sqlx migrate run
```

## Querying Data

### Query As — Map Rows to Structs

```rust
use sqlx::FromRow;
use uuid::Uuid;

#[derive(Debug, FromRow, serde::Serialize)]
struct User {
    id: Uuid,
    name: String,
    email: String,
    created_at: chrono::DateTime<chrono::Utc>,
}

async fn list_users(pool: &sqlx::PgPool) -> Result<Vec<User>, sqlx::Error> {
    sqlx::query_as::<_, User>("SELECT id, name, email, created_at FROM users")
        .fetch_all(pool)
        .await
}
```

Step by step:
- `#[derive(FromRow)]` — auto-generate code that maps database rows to this struct
- Column names must match field names (or use `#[sqlx(rename = "column_name")]`)
- `query_as::<_, User>(...)` — run the query and map each row to a `User` struct
- `fetch_all(pool)` — execute the query and return all rows as a `Vec<User>`

### Query As with Compile-Time Checking

```rust
async fn get_user(pool: &sqlx::PgPool, id: Uuid) -> Result<Option<User>, sqlx::Error> {
    sqlx::query_as::<_, User>("SELECT id, name, email, created_at FROM users WHERE id = $1")
        .bind(id)
        .fetch_optional(pool)
        .await
}
```

Step by step:
- `$1` — parameterized query. First parameter.
- `.bind(id)` — bind the `id` value to `$1`. Prevents SQL injection.
- `fetch_optional` — returns `Some(User)` if found, `None` if not. No panic on missing row.

The `query_as!` macro provides stronger compile-time checking:

```rust
async fn get_user_checked(pool: &sqlx::PgPool, id: Uuid) -> Result<Option<User>, sqlx::Error> {
    sqlx::query_as!(
        User,
        "SELECT id, name, email, created_at FROM users WHERE id = $1",
        id
    )
    .fetch_optional(pool)
    .await
}
```

The `query_as!` macro connects to the database at compile time and verifies:
- The SQL syntax is valid
- The table and columns exist
- The column types match the struct field types
- The parameter count matches the bind count

If any check fails, your code does not compile.

### Insert

```rust
#[derive(serde::Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

async fn create_user(pool: &sqlx::PgPool, input: CreateUser) -> Result<User, sqlx::Error> {
    let row = sqlx::query_as::<_, User>(
        "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id, name, email, created_at"
    )
    .bind(input.name)
    .bind(input.email)
    .fetch_one(pool)
    .await?;

    Ok(row)
}
```

Step by step:
- `RETURNING id, name, email, created_at` — PostgreSQL returns the inserted row
- `.bind(input.name)` — bind name to `$1`
- `.bind(input.email)` — bind email to `$2`
- `fetch_one` — expect exactly one row back (the inserted row)
- `?` — propagate the error if the insert fails (e.g., duplicate email)

### Update

```rust
async fn update_user_email(
    pool: &sqlx::PgPool,
    id: Uuid,
    new_email: String,
) -> Result<bool, sqlx::Error> {
    let result = sqlx::query("UPDATE users SET email = $1 WHERE id = $2")
        .bind(new_email)
        .bind(id)
        .execute(pool)
        .await?;

    Ok(result.rows_affected() > 0)
}
```

- `execute` — run a query that does not return rows
- `result.rows_affected()` — number of rows modified (0 means user not found)

### Delete

```rust
async fn delete_user(pool: &sqlx::PgPool, id: Uuid) -> Result<bool, sqlx::Error> {
    let result = sqlx::query("DELETE FROM users WHERE id = $1")
        .bind(id)
        .execute(pool)
        .await?;

    Ok(result.rows_affected() > 0)
}
```

## Connection Pool

```rust
let pool = sqlx::postgres::PgPoolOptions::new()
    .max_connections(20)
    .connect("postgres://postgres:secret@localhost/myapp")
    .await?;
```

- `PgPool` maintains a pool of reusable connections
- `max_connections(20)` — up to 20 concurrent database connections
- Connections are automatically created, reused, and recycled

Pass the pool to your handlers via Axum state, the same way you passed `Db` in the previous file.

## Offline Mode

For CI environments without a database, use offline mode:

```bash
cargo sqlx prepare
```

This saves query metadata to `.sqlx/`. Commit this directory. SQLx uses the cached metadata instead of connecting to a database.

## Next

The next file covers authentication: JWT tokens, session management, and protecting routes in Rust.

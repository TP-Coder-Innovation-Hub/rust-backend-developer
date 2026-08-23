# Workshop: Book Catalog RESTful API

> **Scenario:** A small bookshop wants to sell online. Their web team needs a catalog API. It must run the same on every machine — so it ships in Docker.

Build a production-style RESTful API in Rust. One service, one table, the full pipeline: server, config, database, validation, docs, containers.

You write all the code. This file tells you what to build and how each part is judged.

**Level:** Mid — finish modules 00–05 first.
**Time:** 1–2 weeks.

---

## What You Will Build

| Method | Route | Action |
|--------|-------|--------|
| GET | `/v1/books` | List books (paged) |
| POST | `/v1/books` | Add a book |
| GET | `/v1/books/{id}` | Get one book |
| PUT | `/v1/books/{id}` | Update a book |
| DELETE | `/v1/books/{id}` | Remove a book |
| GET | `/livez` | Health check |

## The Stack

| Tool | Job |
|------|-----|
| axum + tokio | HTTP server and async runtime |
| tower + tower-http | CORS, request ID, trace middleware |
| serde + serde_json | JSON in and out |
| sqlx + PostgreSQL | Database with compile-time checked SQL |
| garde | Input validation |
| utoipa | OpenAPI docs generated from code |
| tracing | Structured JSON logs |
| envconfig | Config from environment variables |
| thiserror + anyhow | Domain errors and app errors |
| Docker Compose | App + database in containers |

## Skills You Will Prove

- Set up a Cargo workspace with strict lint rules
- Load all config from env vars — nothing hard-coded
- Add middleware: CORS, request ID, request tracing
- Write SQL migrations and compile-time checked queries
- Map every error to a safe JSON response
- Validate input before it reaches your handlers
- Generate OpenAPI docs from the code itself
- Run the whole stack with one Docker command

## Database Design

One table: `books`.

| Column | Type | Null | Notes |
|--------|------|------|-------|
| `created_at` | TIMESTAMPTZ | no | set on insert |
| `updated_at` | TIMESTAMPTZ | no | set on insert and update |
| `id` | UUID | no | primary key, UUIDv7 made in app code |
| `published_date` | DATE | no | |
| `status` | SMALLINT | no | 0 = draft, 1 = published |
| `title` | TEXT | no | |
| `description` | TEXT | yes | |
| `image_url` | TEXT | yes | |

> **Tip — column order matters.** Put fixed-width columns first, widest to smallest. Put text columns last. This trims row padding in PostgreSQL. Known as "Column Tetris". Nice to know, not graded.

## API Contract

**Request body** — `POST` and `PUT`:

```json
{
  "title": "Harry Potter and the Deathly Hallows",
  "description": "The seventh and final novel in the series",
  "image_url": "https://example.com/cover.jpg",
  "published_date": "2007-07-21",
  "status": "published"
}
```

**Response body** — `GET`, `POST`, `PUT`:

```json
{
  "id": "018f6b2e-7c1a-7b3e-9f2a-3d4c5b6a7f81",
  "title": "Harry Potter and the Deathly Hallows",
  "description": "The seventh and final novel in the series",
  "image_url": "https://example.com/cover.jpg",
  "published_date": "2007-07-21",
  "status": "published",
  "created_at": "2026-01-01T00:00:00Z",
  "updated_at": "2026-01-01T00:00:00Z"
}
```

The list endpoint returns an array of the response above.

**Validation error** — status `422`:

```json
{
  "errors": {
    "title": "Must be at least 1 character long",
    "image_url": "Must be a valid URL"
  }
}
```

**Other errors** — status `500`, plain and safe:

```json
{ "error": "DB_FETCH_FAILED" }
```

---

## Requirements

Seven parts. Do them in order. Each part ends with a checkpoint — do not move on until it passes.

### Part 1 — Workspace

- One Cargo workspace, one crate: `book_service`
- Two binaries: `app` (the server) and `migration` (runs migrations), plus a `lib` target for shared code
- `rustfmt.toml` and clippy rules set from day one
- A `justfile` (or `Makefile`) with recipes: `run`, `test`, `lint`, `migrate`

**Checkpoint**
- [ ] `cargo clippy --workspace --all-targets -- -D warnings` passes
- [ ] `just run` starts the server and prints a startup log

### Part 2 — Server and Config

- `GET /livez` returns `200 OK`
- All settings come from env vars: port, allowed CORS origins, body size limit, database URL
- Use `envconfig` (or similar) — the app panics fast with a clear message if config is missing
- Log output as JSON with `tracing`; control log level with one env var (e.g. `RUST_LOG`)

**Checkpoint**
- [ ] `curl -i localhost:3000/livez` returns `200`
- [ ] Changing the port env var changes the port the server binds

### Part 3 — Middleware

Add tower-http layers to the router:

- **Request ID** — every request gets an `x-request-id` header in the response
- **Trace** — one log line per request with method, path, status, and latency
- **CORS** — origins, methods, and headers come from config, not code
- **Body limit** — requests over the configured size are rejected

**Checkpoint**
- [ ] Two `curl` calls return different `x-request-id` values
- [ ] A request from a disallowed origin is blocked by CORS
- [ ] Every request logs method, path, status, latency, and request ID

### Part 4 — Database and Migrations

- `compose.yml` runs PostgreSQL with a health check
- `books` table created by a sqlx migration — never by hand
- Status stored as SMALLINT, mapped to a Rust enum with `#[derive(sqlx::Type)]`:
  - `Draft = 0`, `Published = 1`
- IDs are UUIDv7, made in app code with the `uuid` crate
- All queries use `sqlx::query_as!` or `query!` — compile-time checked
- Pool lives in `AppState`

**Checkpoint**
- [ ] `docker compose up -d db` then `just migrate` creates the table
- [ ] Break a query on purpose (rename a column) — the build fails, not the runtime

### Part 5 — Routes and Errors

- All six routes work end to end against the database
- List supports `?page=1&per_page=10` — `per_page` clamped to 100 max
- One error enum for the whole API. It implements `IntoResponse`:

| Failure | Status | Body |
|---------|--------|------|
| Malformed JSON | 400 | axum rejection |
| Book not found | 404 | empty |
| Invalid input | 422 | field errors |
| DB insert fails | 500 | `{"error": "DB_INSERT_FAILED"}` |
| DB fetch fails | 500 | `{"error": "DB_FETCH_FAILED"}` |
| DB update fails | 500 | `{"error": "DB_UPDATE_FAILED"}` |
| DB delete fails | 500 | `{"error": "DB_DELETE_FAILED"}` |

- **Golden rule:** log the real error with `tracing`, return a short safe message. Never leak internal details to clients.

**Checkpoint**
- [ ] Full CRUD works with `curl`: create, list, read, update, delete
- [ ] `GET /v1/books/{random-uuid}` returns `404`
- [ ] Error logs contain the real cause; responses do not

### Part 6 — Validation

- Use `garde` with rules on the request struct:
  - `title`: 1–255 characters
  - `image_url`: must be a valid URL, when present
- Build a `ValidatedJson<T>` extractor (implement `FromRequest`) so handlers never see bad input
- Validation failures return `422` with one message per field

**Checkpoint**
- [ ] Empty `title` returns `422` with `"Must be at least 1 character long"`
- [ ] `image_url: "not-a-url"` returns `422`
- [ ] Valid payloads still work

### Part 7 — Docs and Docker

- Annotate handlers with `#[utoipa::path]`, derive `ToSchema` on types
- Generate `openapi.yaml` from the code — a small `apidoc` binary or a test that writes the file
- One command runs everything:

```
docker compose up --build
```

- The app container waits for the database health check, runs migrations, then starts the server

**Checkpoint**
- [ ] `openapi.yaml` lists all five book endpoints with request and response schemas
- [ ] On a clean machine: `docker compose up --build`, then CRUD works on `localhost:3000`

---

## Rules

| Rule | Why |
|------|-----|
| Rust 2024 edition | Repo standard |
| No `unwrap()` or `expect()` outside tests | Panics take the whole server down |
| No SQL strings built by hand | Compile-time checked queries only |
| No config values in code | Twelve-factor: config comes from the environment |
| No secrets in git | Ship a `.env.example`, ignore `.env` |
| Errors log full detail, return safe messages | Clients see less, you see more |

## Submission Checklist

- [ ] `cargo clippy --workspace --all-targets -- -D warnings` passes
- [ ] `cargo test` passes with tests for the error mapping and pagination clamps
- [ ] `sqlx` query metadata committed (offline builds work — `cargo sqlx prepare`)
- [ ] `docker compose up --build` starts app + database from clean state
- [ ] All endpoints return the status codes in the contract
- [ ] README covers setup, run, test, and example requests

## Stretch Goals

- Serve Swagger UI at `/docs` with `utoipa-swagger-ui`
- Production image: multi-stage build to a distroless base — aim under 20 MB
- Release profile tuned: `lto`, `codegen-units = 1`, `strip`
- Swap the system allocator for `mimalloc`
- Add a unique `isbn` column with a migration — reject duplicates with `409`
- Return proper JSON for `400` malformed-body errors instead of plain text
- Add search: `?title=harry` filters the list endpoint

## Before You Go

You now have a small but honest production API. Natural next steps:

- Auth — who may create or delete books?
- Tests — handler tests with a real database per test run
- Observability — metrics and health details, not just `/livez`

The deeper workshop in this repo — [Content Moderation Pipeline](./01-workshop-spec.md) — picks up where this one ends: concurrency, channels, and benchmarking.

---

*Structure inspired by the [Learning Rust Laboratory](https://learning-rust.github.io/labs/overview/) RESTful API series. This spec is our own condensed version — sqlx over Toasty, spec-driven over walkthrough.*

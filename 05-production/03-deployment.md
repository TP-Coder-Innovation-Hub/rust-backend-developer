`[Mid]`

# Deployment

Rust compiles to a single static binary. No runtime. No dependencies. This makes deployment straightforward.

## Building a Release Binary

```bash
cargo build --release
```

The binary is at `target/release/your-app-name`. This is a fully optimized, self-contained executable.

Properties of a Rust release binary:
- Static linking by default (no external library dependencies on Linux)
- Stripped of debug symbols (with proper configuration)
- Typically 5–30 MB (small for a web server)
- Starts in milliseconds

### Optimizing the Binary

Add to `Cargo.toml` for smaller binaries:

```toml
[profile.release]
strip = true        # remove debug symbols
lto = true          # link-time optimization (smaller, slower compile)
codegen-units = 1   # better optimization (slower compile)
opt-level = "s"     # optimize for size (or "z" for even smaller)
```

| Setting | Effect |
|---------|--------|
| `strip = true` | Removes debug info. Smaller binary. |
| `lto = true` | Whole-program optimization. Smaller and faster, slower compile. |
| `codegen-units = 1` | Single codegen unit. Better optimization. |
| `opt-level = "s"` | Optimize for size. Use 3 for speed. |

## Cross-Compilation

Build for Linux from macOS:

```bash
rustup target add x86_64-unknown-linux-musl
cargo build --release --target x86_64-unknown-linux-musl
```

`musl` produces a fully static binary — no glibc dependency. Works on any Linux distribution, including Alpine (minimal Docker images).

Build for ARM (AWS Graviton, Raspberry Pi):

```bash
rustup target add aarch64-unknown-linux-musl
cargo build --release --target aarch64-unknown-linux-musl
```

## Docker

### Multi-Stage Build

`Dockerfile`:

```dockerfile
# Stage 1: Build
FROM rust:1.87-slim AS builder

WORKDIR /app

COPY Cargo.toml Cargo.lock ./
RUN mkdir src && echo "fn main() {}" > src/main.rs
RUN cargo build --release && rm -rf src

COPY src ./src
RUN touch src/main.rs && cargo build --release

# Stage 2: Runtime
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/target/release/my-api /usr/local/bin/my-api

EXPOSE 3000

CMD ["my-api"]
```

Step by step:
- **Stage 1** builds the binary. Uses the full Rust image with the compiler.
- `COPY Cargo.toml Cargo.lock ./` — copy dependency files first
- `RUN mkdir src && echo "fn main() {}" > src/main.rs` — create a dummy main.rs
- `RUN cargo build --release` — build dependencies. Docker caches this layer. Dependencies rarely change.
- `COPY src ./src` — copy your actual source code
- `RUN touch src/main.rs && cargo build --release` — rebuild only your code (cached dependencies)
- **Stage 2** copies just the binary. Uses a minimal Debian image. The Rust compiler is not included.
- `ca-certificates` — needed for HTTPS requests to external services

### Alpine-based (Smaller Image)

```dockerfile
FROM rust:1.87-alpine AS builder

RUN apk add --no-cache musl-dev

WORKDIR /app
COPY Cargo.toml Cargo.lock ./
RUN mkdir src && echo "fn main() {}" > src/main.rs
RUN cargo build --release && rm -rf src

COPY src ./src
RUN touch src/main.rs && cargo build --release

FROM alpine:3.21

RUN apk add --no-cache ca-certificates
COPY --from=builder /app/target/release/my-api /usr/local/bin/my-api

EXPOSE 3000
CMD ["my-api"]
```

Result: an image around 10–20 MB (vs 100+ MB for Node.js or Java).

### Image Size Comparison

| Runtime | Typical image size |
|---------|-------------------|
| Rust (Alpine) | 10–20 MB |
| Go (Alpine) | 10–15 MB |
| Node.js (Alpine) | 120+ MB |
| Java (JRE slim) | 200+ MB |
| Python (Alpine) | 50+ MB |

## Environment Configuration

Use environment variables for configuration, not config files:

```rust
use std::env;

fn get_config() -> Config {
    Config {
        database_url: env::var("DATABASE_URL")
            .expect("DATABASE_URL must be set"),
        port: env::var("PORT")
            .unwrap_or_else(|_| "3000".into())
            .parse()
            .expect("PORT must be a number"),
        jwt_secret: env::var("JWT_SECRET")
            .expect("JWT_SECRET must be set"),
        log_level: env::var("RUST_LOG")
            .unwrap_or_else(|_| "info".into()),
    }
}
```

Pass secrets via environment variables in Docker:

```bash
docker run -d \
  -p 3000:3000 \
  -e DATABASE_URL=postgres://... \
  -e JWT_SECRET=... \
  -e RUST_LOG=info \
  my-api
```

Never bake secrets into the Docker image.

## Health Checks

Add a health endpoint for orchestration systems (Kubernetes, ECS, Docker health check):

```rust
async fn health() -> &'static str {
    "OK"
}

let app = Router::new()
    .route("/health", get(health))
    .route("/users", get(list_users));
```

Docker health check:

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1
```

## Graceful Shutdown

Handle SIGTERM to finish in-flight requests:

```rust
use tokio::signal;

async fn shutdown_signal() {
    signal::ctrl_c()
        .await
        .expect("Failed to install Ctrl+C handler");
    println!("Shutting down gracefully...");
}

#[tokio::main]
async fn main() {
    let app = Router::new().route("/health", get(health));
    let listener = TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app)
        .with_graceful_shutdown(shutdown_signal())
        .await
        .unwrap();
}
```

## Deployment Checklist

| Item | Why |
|------|-----|
| Build with `--release` | Optimized binary, 10–100x faster than debug |
| Use multi-stage Docker build | Final image contains only the binary |
| Set environment variables | Configure without rebuilding |
| Add health endpoint | Orchestration systems need it |
| Handle graceful shutdown | Finish in-flight requests before exiting |
| Strip debug symbols | Smaller binary |
| Use musl target | Fully static binary, no glibc issues |

## Next

The capstone project in the next module brings everything together: build a complete memory-safe API.

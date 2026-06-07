`[Entry]` `[Mid]`

# Crates and Modules

Rust code is organized into crates and modules. Crates are packages you share and reuse. Modules organize code within a crate.

## Cargo.toml — Project Configuration

Every Rust project has a `Cargo.toml` file:

```toml
[package]
name = "my-api"
version = "0.1.0"
edition = "2024"

[dependencies]
axum = "0.8"
tokio = { version = "1", features = ["rt-multi-thread", "macros"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
sqlx = { version = "0.8", features = ["postgres", "runtime-tokio"] }
```

Step by step:
- `[package]` — project metadata: name, version, Rust edition
- `edition = "2024"` — the Rust edition. Editions are released every 3 years. 2024 is current.
- `[dependencies]` — external crates your project needs
- `features = [...]` — enable optional functionality. Tokio needs `rt-multi-thread` for the async runtime and `macros` for `#[tokio::main]`.
- Serde needs `derive` to auto-generate serialization code with `#[derive(Serialize, Deserialize)]`.

### Adding Dependencies

```bash
# Add a dependency
cargo add axum

# Add with features
cargo add tokio --features rt-multi-thread,macros

# Add a dev dependency (only for tests)
cargo add --dev assert_cmd
```

Or edit `Cargo.toml` manually. Both approaches work.

## Crates — Reusable Packages

A crate is a compilation unit. There are two types:

| Type | Purpose | Entry point |
|------|---------|-------------|
| Binary crate | Executable program | `src/main.rs` |
| Library crate | Reusable code | `src/lib.rs` |

A project can have both:

```
my-project/
  Cargo.toml
  src/
    main.rs       # binary crate
    lib.rs        # library crate
    bin/
      tool.rs     # additional binary crate
```

### Using External Crates

```rust
use serde_json::json;

fn main() {
    let data = json!({
        "name": "Alice",
        "age": 30
    });
    println!("{data}");
}
```

Step by step:
- `use serde_json::json` — bring the `json!` macro into scope from the `serde_json` crate
- `json!({...})` — create a JSON value using the macro

### Finding Crates

- [crates.io](https://crates.io) — the official registry. Search by category or keyword.
- [lib.rs](https://lib.rs) — curated index of quality Rust crates.
- Check download count, recent updates, and issue activity before choosing a crate.

## Modules — Organizing Code Within a Crate

Modules group related code. They control visibility (what is public vs private).

### File-Based Modules

```
src/
  main.rs
  models.rs        # a module named "models"
  handlers/
    mod.rs          # a module named "handlers"
    users.rs        # a submodule named "handlers::users"
    health.rs       # a submodule named "handlers::health"
```

In `src/main.rs`:

```rust
mod models;              // declare module from models.rs
mod handlers;            // declare module from handlers/mod.rs

use handlers::users;     // bring users into scope
use models::User;        // bring User type into scope
```

### Inline Modules

```rust
mod math {
    pub fn add(a: i32, b: i32) -> i32 {   // pub = public
        a + b
    }

    fn subtract(a: i32, b: i32) -> i32 {  // private (no pub)
        a - b
    }
}

fn main() {
    let sum = math::add(3, 4);       // OK: add is pub
    // math::subtract(3, 4);         // ERROR: subtract is private
}
```

Step by step:
- `mod math { ... }` — define a module inline
- `pub fn add` — the function is public. Code outside the module can use it.
- `fn subtract` — no `pub`, so it is private. Only code inside `math` can use it.

### Visibility Rules

| Keyword | Visibility |
|---------|-----------|
| (no keyword) | Private: only the current module and children |
| `pub` | Public: any code that can see the module |
| `pub(crate)` | Visible within the current crate |
| `pub(super)` | Visible in the parent module |

### The use Keyword

`use` brings items into scope so you do not need the full path:

```rust
// Without use
std::collections::HashMap::new();

// With use
use std::collections::HashMap;
HashMap::new();

// Bring multiple items
use std::collections::{HashMap, HashSet};

// Bring everything (avoid this — unclear what is used)
use std::collections::*;
```

Prefer specific imports. `use std::collections::HashMap` is clear. `use std::collections::*` is not.

## Module vs Crate Summary

| Concept | Scope | Example |
|---------|-------|---------|
| Crate | External package | `axum`, `serde`, `tokio` |
| Module | Internal organization | `mod handlers`, `mod models` |
| `pub` | Visibility control | `pub fn`, `pub struct` |
| `use` | Import into scope | `use crate::handlers::users` |

## Next

The next file covers async Rust: how to write concurrent code using async/await and the Tokio runtime.

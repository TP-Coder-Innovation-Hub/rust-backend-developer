`[Entry]` `[Mid]`

# Error Handling

Rust has no exceptions and no null. Instead, it uses two types to represent fallible operations: `Option<T>` for values that might be absent, and `Result<T, E>` for operations that might fail.

## Option — Absent Values

`Option<T>` replaces null. A value is either `Some(data)` or `None`.

```rust
fn find_user(id: i32) -> Option<String> {
    if id == 1 {
        Some(String::from("Alice"))
    } else {
        None
    }
}

fn main() {
    let user = find_user(1);

    match user {
        Some(name) => println!("Found: {name}"),
        None => println!("Not found"),
    }
}
```

Step by step:
- `find_user` returns `Option<String>` — either a name or nothing
- `Some(...)` wraps a value. `None` means no value.
- `match` handles both cases. The compiler forces you to handle `None`.

### Option Methods

```rust
fn main() {
    let maybe_number: Option<i32> = Some(42);

    // unwrap: get the value or panic
    let n = maybe_number.unwrap();           // 42 (panics if None)

    // unwrap_or: get the value or a default
    let none: Option<i32> = None;
    let n = none.unwrap_or(0);              // 0

    // map: transform the inner value
    let doubled = maybe_number.map(|x| x * 2);  // Some(84)

    // is_some / is_none: check
    if maybe_number.is_some() {
        println!("Has a value");
    }
}
```

| Method | Behavior |
|--------|----------|
| `.unwrap()` | Returns value, panics on `None`. Use only in tests or prototyping. |
| `.unwrap_or(default)` | Returns value or the provided default. |
| `.unwrap_or_else(\|\| compute())` | Returns value or computes a default lazily. |
| `.map(\|x\| transform(x))` | Transform the value inside `Some`. Returns `None` unchanged. |
| `.and_then(\|x\| ...)` | Chain operations that return `Option`. |
| `.is_some()` / `.is_none()` | Boolean check. |

## Result — Fallible Operations

`Result<T, E>` represents either success `Ok(T)` or failure `Err(E)`.

```rust
use std::fs;

fn read_config() -> Result<String, std::io::Error> {
    fs::read_to_string("config.toml")
}

fn main() {
    match read_config() {
        Ok(content) => println!("Config: {content}"),
        Err(error) => println!("Failed: {error}"),
    }
}
```

Step by step:
- `Result<String, std::io::Error>` — success returns a `String`, failure returns an `io::Error`
- `Ok(...)` wraps the success value. `Err(...)` wraps the error.
- `match` handles both cases.

### The ? Operator

The `?` operator propagates errors. If the result is `Err`, it returns early from the function. If `Ok`, it unwraps the value:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut file = File::open("username.txt")?;   // ? propagates Err, or unwraps Ok
    let mut username = String::new();
    file.read_to_string(&mut username)?;           // ? propagates Err, or unwraps Ok
    Ok(username)                                    // wrap success in Ok
}
```

Step by step:
- `File::open("username.txt")?` — try to open the file. If it fails, return the error immediately. If it succeeds, bind the file handle to `file`.
- `file.read_to_string(&mut username)?` — try to read. Same behavior: propagate error or continue.
- `Ok(username)` — return the successful result.

Without `?`, the same code is:

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut file = match File::open("username.txt") {
        Ok(f) => f,
        Err(e) => return Err(e),
    };
    let mut username = String::new();
    match file.read_to_string(&mut username) {
        Ok(_) => {}
        Err(e) => return Err(e),
    }
    Ok(username)
}
```

The `?` operator removes this boilerplate. Use it.

### ? with Option

`?` also works with `Option`:

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

Step by step:
- `text.lines().next()` — get the first line. Returns `Option<&str>`.
- `?` — if `None`, return `None` from the function. If `Some`, unwrap.
- `.chars().last()` — get the last character. Returns `Option<char>`.

## Error Types in Practice

For libraries, use `thiserror` to define error enums:

```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum AppError {
    #[error("user not found: {id}")]
    UserNotFound { id: u32 },
    #[error("database error: {0}")]
    Database(#[from] sqlx::Error),
}
```

For applications, use `anyhow` for flexible error handling:

```rust
use anyhow::{Context, Result};

fn load_config() -> Result<Config> {
    let content = fs::read_to_string("config.toml")
        .context("Failed to read config file")?;
    let config: Config = toml::from_str(&content)
        .context("Failed to parse config")?;
    Ok(config)
}
```

| Crate | Use when |
|-------|----------|
| `thiserror` | Library code — callers need to match specific error variants |
| `anyhow` | Application code — you just need to propagate errors with context |

## Why This Is Better Than Exceptions

| Property | Exceptions (Java, Python) | Result/Option (Rust) |
|----------|--------------------------|---------------------|
| Visible in type signature | No | Yes |
| Forgetting to handle | Possible | Compiler rejects |
| Control flow surprise | Yes (non-local return) | No (explicit propagation) |
| Performance | Unwinding overhead | Zero cost on success path |

The function signature tells you whether an operation can fail. The compiler ensures you handle it. No surprises.

## Next

You now understand ownership, borrowing, traits, and error handling. The next module covers the ecosystem: crates, async Rust, and thread safety.

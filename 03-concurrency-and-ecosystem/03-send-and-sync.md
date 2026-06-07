`[Mid]` `[Senior]`

# Send and Sync

When Rust code runs across multiple threads (which Tokio does by default), the compiler uses two marker traits to enforce thread safety: `Send` and `Sync`. You do not implement these yourself often, but you need to understand them when the compiler rejects your code.

## What They Mean

| Trait | Meaning |
|-------|---------|
| `Send` | A type is safe to **transfer ownership** to another thread |
| `Sync` | A type is safe to **share a reference** between threads |

Most types are both `Send` and `Sync`. The compiler automatically derives these traits when all fields of a struct are `Send`/`Sync`.

## Send — Moving Values Between Threads

If a type is `Send`, you can move it to another thread:

```rust
use std::thread;

fn main() {
    let data = vec![1, 2, 3];       // Vec<i32> is Send

    thread::spawn(move || {          // move data into the new thread
        println!("{:?}", data);      // OK: Vec<i32> is Send
    }).join().unwrap();
}
```

Step by step:
- `data` is a `Vec<i32>`. `Vec<i32>` is `Send` because `i32` is `Send`.
- `move ||` — closure takes ownership of `data`
- The new thread owns `data` and can use it safely

### When a Type Is NOT Send

```rust
use std::rc::Rc;
use std::thread;

fn main() {
    let data = Rc::new(vec![1, 2, 3]);    // Rc is NOT Send

    thread::spawn(move || {
        println!("{:?}", data);           // ERROR: Rc<i32> cannot be sent between threads
    });
}
```

`Rc` (Reference Counted) is a single-threaded reference-counted pointer. It is not thread-safe because its reference count is not atomic. Two threads incrementing the count simultaneously would cause a data race.

Fix: use `Arc` (Atomic Reference Counted) instead:

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let data = Arc::new(vec![1, 2, 3]);   // Arc IS Send + Sync

    thread::spawn(move || {
        println!("{:?}", data);           // OK
    }).join().unwrap();
}
```

| Type | Thread-safe? | Use case |
|------|-------------|----------|
| `Rc<T>` | No (`!Send`) | Single-threaded reference counting |
| `Arc<T>` | Yes (`Send + Sync`) | Multi-threaded reference counting |
| `RefCell<T>` | No (`!Sync`) | Single-threaded interior mutability |
| `Mutex<T>` | Yes (`Send + Sync`) | Multi-threaded interior mutability |

## Sync — Sharing References Between Threads

If a type is `Sync`, multiple threads can hold references to it simultaneously:

```rust
use std::thread;

fn main() {
    let value = 42;                          // i32 is Sync

    thread::spawn(move || {
        println!("{}", value);               // OK: i32 is Send + Sync
    }).join().unwrap();
}
```

`Sync` means `&T` is `Send`. If you can send a reference to another thread safely, `T` is `Sync`.

### When a Type Is NOT Sync

```rust
use std::cell::RefCell;
use std::thread;

fn main() {
    let data = RefCell::new(vec![1, 2, 3]);    // RefCell is NOT Sync

    thread::spawn(move || {
        data.borrow_mut().push(4);              // ERROR: RefCell cannot be shared between threads
    });
}
```

`RefCell` enforces borrow rules at runtime (instead of compile time) for single-threaded code. It is not safe across threads.

Fix: use `Mutex` for multi-threaded interior mutability:

```rust
use std::sync::Mutex;
use std::thread;

fn main() {
    let data = Mutex::new(vec![1, 2, 3]);

    let handle = thread::spawn(move || {
        data.lock().unwrap().push(4);           // OK: Mutex is Send + Sync
    });

    handle.join().unwrap();
}
```

## How This Affects Async Code

Tokio tasks may run on different threads. Types captured by `tokio::spawn` must be `Send`:

```rust
use std::rc::Rc;
use tokio::spawn;

async fn broken() {
    let data = Rc::new(vec![1, 2, 3]);     // Rc is !Send

    spawn(async move {
        println!("{:?}", data);             // ERROR: future is not Send
    });
}
```

The compiler error says "future cannot be sent between threads safely" and points to `Rc`. The fix is to use `Arc` instead.

### Holding Mutex Across Await Points

```rust
use std::sync::Mutex;

async fn dangerous(data: &Mutex<Vec<i32>>) {
    let mut guard = data.lock().unwrap();
    some_async_operation().await;           // PROBLEM: guard is held across await
    guard.push(4);
}
```

When you hold a `Mutex` lock across `.await`, the lock is held while the task is suspended. Other tasks waiting for the lock will block. In a single-threaded runtime, this deadlocks. In a multi-threaded runtime, it causes contention.

Fix: scope the lock:

```rust
async fn safe(data: &Mutex<Vec<i32>>) {
    {
        let mut guard = data.lock().unwrap();
        guard.push(4);
    }                                       // lock released here
    some_async_operation().await;           // no lock held
}
```

For async-aware locking, use `tokio::sync::Mutex`:

```rust
use tokio::sync::Mutex;

async fn async_safe(data: &Mutex<Vec<i32>>) {
    let mut guard = data.lock().await;      // async lock
    guard.push(4);
    some_async_operation().await;           // OK with tokio::sync::Mutex
}
```

## Summary

| Trait | Meaning | Auto-derived? |
|-------|---------|---------------|
| `Send` | Safe to move to another thread | Yes, if all fields are `Send` |
| `Sync` | Safe to share references across threads | Yes, if all fields are `Sync` |

Common fixes:
- `Rc` not `Send` — use `Arc`
- `RefCell` not `Sync` — use `Mutex`
- Holding locks across `.await` — scope the lock or use `tokio::sync::Mutex`

## Next

You understand ownership, async, and thread safety. The next module applies these concepts to building backend APIs with HTTP, REST, Axum, SQLx, and authentication.

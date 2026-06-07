`[Entry]`

# What Is Rust

Rust is a systems programming language that guarantees memory safety without a garbage collector. It compiles to native machine code and runs with the same performance as C and C++.

## History

- **2006** — Graydon Hoare starts Rust as a personal project at Mozilla. Motivated by a faulty elevator caused by a C++ memory bug.
- **2009** — Mozilla sponsors Rust officially. Goal: build a safe language for writing a web browser (Servo).
- **2015** — Rust 1.0 released. Stability guarantee: code that compiles on 1.0 will compile on all future versions.
- **2016–2024** — Stack Overflow's "most loved language" every year. Adopted by major companies.
- **2021** — Rust Foundation established. Mozilla, Google, Microsoft, AWS, Huawei, and others fund independent governance.
- **2024** — Rust 2024 edition released. Linux kernel accepts Rust code. Android OS uses Rust for new native code.

## The Core Promise

Rust's tagline is "memory safety without garbage collection." This means:

1. **No null pointer dereferences.** Rust does not have `null`. It uses `Option<T>` instead.
2. **No use-after-free.** The ownership system ensures memory is freed exactly once.
3. **No data races.** The borrow checker prevents concurrent access to the same data without proper synchronization.
4. **No buffer overflows.** Array access is bounds-checked by default.
5. **No uninitialized memory.** The compiler ensures variables are initialized before use.

These guarantees are enforced at compile time. If your code would cause one of these bugs, it does not compile. No runtime cost for the checks.

## Zero-Cost Abstractions

Rust's abstractions (generics, iterators, traits) compile down to the same machine code you would write by hand. Using a higher-level construct does not make your program slower.

Example: `vec.iter().map(|x| x * 2).sum()` compiles to a tight loop equivalent to hand-written C code. The compiler inlines and optimizes away the abstraction entirely.

## Who Uses Rust in Production

| Company | Use case |
|---------|----------|
| Shopify | High-performance HTTP router handling millions of requests |
| Discord | Read states service (replaced Go, 10x latency improvement) |
| Cloudflare | Edge network services, firewall rules |
| AWS | Firecracker (microVM for Lambda), Bottlerocket OS |
| Microsoft | Windows kernel components, rewriting vulnerable C++ code |
| Google | Android OS, Chrome components, Fuchsia OS |
| Dropbox | File synchronization engine (Nucleus) |
| Firefox | CSS engine (Stylo), rendering pipeline |

## The Rust Ecosystem for Backend Development

| Tool/Library | Purpose |
|-------------|---------|
| Cargo | Build system and package manager (like npm + make combined) |
| Tokio | Async runtime for concurrent I/O |
| Axum | Web framework (built on Tokio and Tower) |
| SQLx | Compile-time checked SQL queries |
| Serde | Serialization (JSON, YAML, TOML, binary formats) |
| Tracing | Structured logging and diagnostics |
| Clippy | Linter that catches common mistakes |

## What Makes Rust Different

Three things distinguish Rust from other backend languages:

1. **The compiler prevents bugs other languages find at runtime.** The compile-test cycle is slower, but you ship fewer bugs.

2. **No runtime overhead.** No garbage collector pauses. No interpreter. Predictable latency and consistent performance.

3. **Fearless refactoring.** Because the compiler checks so much, large-scale refactoring is safer. If it compiles, it probably works.

## Next

The next file compares Rust against C++, Go, and Java honestly — including when Rust is not the right choice.

# AGENTS.md

## Context

This repo is part of the TP Coder Innovation Hub learning platform. It serves as a structured learning path for Rust backend development, organized as a directory-per-topic guide from foundations through production deployment.

## Audience

Learners range from entry-level to senior engineers exploring Rust. Adjust explanations based on the learner's level:

- **Entry-level**: Explain concepts from scratch, use analogies. Walk through code line by line. Reference files in `00-foundations/` and `01-first-code/`.
- **Mid-level**: Focus on patterns, trade-offs, and how Rust differs from languages they already know. Reference files in `02-ownership-and-safety/` through `04-backend-fundamentals/`.
- **Senior-level**: Discuss memory model implications, zero-cost abstractions, and systems design. Reference files in `03-concurrency-and-ecosystem/` and `05-production/`.

## How to Help

- Guide learners to discover answers rather than giving them directly. Ask "what do you think the borrow checker is complaining about?" before explaining.
- When showing code, explain WHY the borrow checker requires something, not just what to change.
- Connect concepts to the learner's existing knowledge. If they know Java, compare traits to interfaces.
- Point out common mistakes before the learner makes them: using unwrap() in production, holding locks across await points, fighting the borrow checker with excessive cloning.
- Suggest incremental exercises. Small steps, each building on the last.
- Reference specific files when relevant. "02-ownership-and-safety/01-ownership.md covers this in detail."
- When a learner is stuck on a compiler error, read the error message together. Rust compiler errors are informative.

## How NOT to Help

- Do NOT give copy-paste solutions without explanation.
- Do NOT assume the learner knows systems programming.
- Do NOT skip ownership fundamentals.
- Do NOT recommend `unwrap()` in production code. Show the proper error handling pattern.
- Do NOT overwhelm entry-level learners with advanced topics.
- Do NOT present opinions as facts.

## Key Concepts to Emphasize

1. **Ownership is the foundation.** Every value has one owner. When the owner is dropped, the value is freed. Enforced by the compiler.
2. **Borrowing is how you share data.** Many readers OR one writer, never both at the same time.
3. **Result and Option replace exceptions and null.** The return type tells you whether an operation can fail.
4. **Traits replace inheritance.** Shared behavior defined in traits. No class hierarchies.
5. **Async is cooperative, not preemptive.** Tasks yield at await points. Use spawn_blocking for CPU-heavy work.
6. **The compiler is your teammate.** Rust compiler errors prevent real bugs.
7. **Idiomatic Rust differs from translated code.** Learn the patterns: builder, newtype, From/Into.

## Rust-Specific Guidelines (2026)

- Use Rust 2024 edition. Enable `edition = "2024"` in Cargo.toml.
- Use `axum` as the default web framework.
- Use `sqlx` for database access with compile-time checked queries.
- Use `tokio` as the async runtime with features: `rt-multi-thread`, `macros`, `signal`.
- Use `tracing` for structured logging, not `log`.
- Prefer `thiserror` for library error types and `anyhow` for application error types.
- Use `serde` with `serde_json` for JSON serialization.
- Use `uuid` with `v7` feature for primary keys.
- Prefer `&str` over `String` in function parameters when you only need to read.

## Repository Structure

```
rust-backend-developer/
  README.md                                  # Navigation table and learning objectives
  AGENTS.md                                  # Instructions for AI assistants (this file)
  00-foundations/
    01-what-is-programming.md                # What code does. Recipe analogy.
    02-paradigms.md                          # Imperative, OOP, functional, systems.
    03-sequential-decision-iteration.md      # The 3 building blocks.
    04-compiler-vs-interpreter.md            # Compiled vs interpreted.
    05-memory-management-basics.md           # Stack vs heap. GC.
    06-what-is-rust.md                       # History, philosophy, users.
    07-why-rust-why-not-x.md                # Rust vs C++ vs Go vs Java.
  01-first-code/
    01-setup.md                              # rustup, cargo, first program.
    02-variables-and-mutability.md           # let, let mut, shadowing.
    03-control-flow.md                       # if/else, loop/while/for, match.
    04-functions.md                          # fn, parameters, expressions.
    05-structs-and-enums.md                  # struct, enum, impl.
  02-ownership-and-safety/
    01-ownership.md                          # One owner, move semantics.
    02-references-and-borrowing.md           # &T, &mut T, borrow rules.
    03-lifetimes.md                          # Why lifetimes exist. Patterns.
    04-traits.md                             # Trait bounds, impl Trait, trait objects.
    05-error-handling.md                     # Result, Option, ? operator.
  03-concurrency-and-ecosystem/
    01-crates-and-modules.md                 # Cargo.toml, dependencies, modules.
    02-async-rust.md                         # async/await, Tokio, spawning tasks.
    03-send-and-sync.md                      # Thread safety traits.
  04-backend-fundamentals/
    01-http-and-web-servers.md               # HTTP basics.
    02-rest-api-design.md                    # Resources, verbs, status codes.
    03-your-first-api.md                     # Minimal Axum API, line by line.
    04-database-access.md                    # SQLx compile-time checked queries.
    05-authentication.md                     # JWT, sessions, middleware.
  05-production/
    01-testing.md                            # Unit, handler, integration tests.
    02-logging-and-monitoring.md             # tracing crate, structured logging.
    03-deployment.md                         # Static binary, Docker, small images.
  06-workshop/
    README.md                                # Project spec: memory-safe task API.
```

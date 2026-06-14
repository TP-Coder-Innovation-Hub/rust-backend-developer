# Rust Backend Developer — Learning Path

A structured guide from zero to production Rust backend. Each module builds on the last.

## Learning Objectives

By the end you will understand why Rust exists, write safe concurrent code, build a backend API, and deploy it.

## Prerequisites

You need basic computer literacy. No prior Rust, systems programming, or backend experience required. If you know another language, you will move faster through foundations.

## Path Overview

| # | Module | Topics | Level |
|---|--------|--------|-------|
| 00 | [Foundations](00-foundations/) | Programming concepts, memory, what Rust is | Entry |
| 01 | [First Code](01-first-code/) | Setup, variables, control flow, functions, structs | Entry |
| 02 | [Ownership and Safety](02-ownership-and-safety/) | Ownership, borrowing, lifetimes, traits, errors | Entry–Mid |
| 03 | [Concurrency and Ecosystem](03-concurrency-and-ecosystem/) | Crates, async, Send/Sync | Mid |
| 04 | [Backend Fundamentals](04-backend-fundamentals/) | HTTP, REST, Axum, SQLx, auth | Mid |
| 05 | [Production](05-production/) | Testing, logging, deployment | Mid–Senior |
| 06 | [Workshop](06-workshop/README.md) | Build a memory-safe API end to end | Mid |

## Full Topic List

### 00 — Foundations

| File | Topic |
|------|-------|
| [01-what-is-programming.md](00-foundations/01-what-is-programming.md) | What code does. How humans instruct machines. |
| [02-paradigms.md](00-foundations/02-paradigms.md) | Imperative, OOP, functional, systems programming. |
| [03-sequential-decision-iteration.md](00-foundations/03-sequential-decision-iteration.md) | The 3 building blocks of all programs. |
| [04-compiler-vs-interpreter.md](00-foundations/04-compiler-vs-interpreter.md) | Compiled vs interpreted languages. |
| [05-memory-management-basics.md](00-foundations/05-memory-management-basics.md) | Stack vs heap. What garbage collection does. |
| [06-what-is-rust.md](00-foundations/06-what-is-rust.md) | History, philosophy, who uses it. |
| [07-why-rust-why-not-x.md](00-foundations/07-why-rust-why-not-x.md) | Rust vs C++ vs Go vs Java. Trade-offs. |

### 01 — First Code

| File | Topic |
|------|-------|
| [01-setup.md](01-first-code/01-setup.md) | rustup, cargo, editor setup. Run first program. |
| [02-variables-and-mutability.md](01-first-code/02-variables-and-mutability.md) | let, let mut, shadowing. |
| [03-control-flow.md](01-first-code/03-control-flow.md) | if/else, loop/while/for, match. |
| [04-functions.md](01-first-code/04-functions.md) | fn, parameters, expressions vs statements. |
| [05-structs-and-enums.md](01-first-code/05-structs-and-enums.md) | struct, enum, impl blocks. |

### 02 — Ownership and Safety

| File | Topic |
|------|-------|
| [01-ownership.md](02-ownership-and-safety/01-ownership.md) | One owner, move semantics, no GC needed. |
| [02-references-and-borrowing.md](02-ownership-and-safety/02-references-and-borrowing.md) | &T, &mut T, borrow rules. |
| [03-lifetimes.md](02-ownership-and-safety/03-lifetimes.md) | Why lifetimes exist. Common patterns. |
| [04-traits.md](02-ownership-and-safety/04-traits.md) | Trait bounds, impl Trait, trait objects. |
| [05-error-handling.md](02-ownership-and-safety/05-error-handling.md) | Result, Option, the ? operator. |

### 03 — Concurrency and Ecosystem

| File | Topic |
|------|-------|
| [01-crates-and-modules.md](03-concurrency-and-ecosystem/01-crates-and-modules.md) | Cargo.toml, dependencies, module system. |
| [02-async-rust.md](03-concurrency-and-ecosystem/02-async-rust.md) | async/await, Tokio, spawning tasks. |
| [03-send-and-sync.md](03-concurrency-and-ecosystem/03-send-and-sync.md) | Thread safety traits. |

### 04 — Backend Fundamentals

| File | Topic |
|------|-------|
| [01-http-and-web-servers.md](04-backend-fundamentals/01-http-and-web-servers.md) | HTTP basics. What happens when you type a URL. |
| [02-rest-api-design.md](04-backend-fundamentals/02-rest-api-design.md) | Resources, verbs, status codes, pagination. |
| [03-your-first-api.md](04-backend-fundamentals/03-your-first-api.md) | Build a minimal API with Axum. |
| [04-database-access.md](04-backend-fundamentals/04-database-access.md) | SQLx: compile-time checked queries. |
| [05-authentication.md](04-backend-fundamentals/05-authentication.md) | JWT, sessions, auth in Rust. |

### 05 — Production

| File | Topic |
|------|-------|
| [01-testing.md](05-production/01-testing.md) | Unit tests, integration tests, testing handlers. |
| [02-logging-and-monitoring.md](05-production/02-logging-and-monitoring.md) | tracing crate, structured logging. |
| [03-deployment.md](05-production/03-deployment.md) | Static binary, Docker, small images. |

### 06 — Capstone

| File | Topic |
|------|-------|
| [README.md](06-capstone/README.md) | Project spec: build a memory-safe API. |

## How to Use This Guide

Start at 00-foundations. Read each file in order. Every module assumes you understand the previous one. Code examples explain what each line does. If you get stuck, re-read the previous file.

`` means fundamental concepts. `` means patterns and trade-offs. `` means architecture and internals.

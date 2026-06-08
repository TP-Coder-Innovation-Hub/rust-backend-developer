`` ``

# Why Rust, Why Not X

No language is best for everything. This file compares Rust against its closest alternatives honestly.

## Comparison Table

> 🖼️ **[IMAGE_PLACEHOLDER]** — Rust vs C++ Go Java comparison trade-off chart

| Property | Rust | C++ | Go | Java |
|----------|------|-----|-----|------|
| Memory safety | Compile-time guaranteed | Manual, error-prone | Garbage collected | Garbage collected |
| Performance | Very fast (near C++) | Very fast | Fast (GC pauses) | Medium (JIT warmup) |
| Compile speed | Slow | Slow | Fast | Medium |
| Learning curve | Steep (ownership, lifetimes) | Very steep | Gentle | Moderate |
| Concurrency | Fearless (compile-time checked) | Error-prone (data races common) | Simple (goroutines) | Verbose (threads, locks) |
| Binary output | Single static binary | Single static binary | Single static binary | Requires JVM |
| Ecosystem size | Growing fast | Massive | Large, mature | Massive, mature |
| GC pauses | None | None | Yes (low latency) | Yes (can be tuned) |
| Job market (2026) | Growing rapidly | Large, stable | Large, growing | Very large, stable |

## When to Use Rust

- **Latency-sensitive services.** No GC pauses. Predictable sub-millisecond response times.
- **High-throughput network services.** Async I/O with zero-cost abstractions. Rust handles millions of connections efficiently.
- **Security-critical code.** The compiler eliminates entire vulnerability classes (buffer overflows, use-after-free, null dereferences).
- **Resource-constrained environments.** Small memory footprint. Single binary deployment. Good for embedded and containers.
- **Long-running services.** No memory leaks from GC cycles. No runtime degradation over time.
- **Codebases that need to last.** Strong type system and compiler checks make large refactors safe.

## When to Use Something Else

- **Rapid prototyping.** Python or JavaScript get a prototype running faster. Rust's compiler slows the iteration cycle.
- **Simple CRUD APIs.** Go, Python, or Node.js are faster to write and easier to staff. Rust's safety guarantees are overkill for a simple REST API backed by a database.
- **Teams without Rust experience.** Rust takes 3–6 months to learn productively. If your team does not have that time, use what they know.
- **Scripting and automation.** Python, Bash, or Go are better suited. Rust's compile step adds friction.
- **Machine learning.** Python dominates. Rust's ML ecosystem is nascent.
- **Mobile apps.** Kotlin (Android) and Swift (iOS) have better tooling and ecosystem. Rust is used for shared native libraries, not full apps.

## Honest Trade-offs

### What Rust Costs You

1. **Compile time.** Large Rust projects compile in minutes, not seconds. This is the top complaint.
2. **Learning curve.** Ownership and borrowing are unique to Rust. Expect 3–6 months to become productive.
3. **Ecosystem maturity.** Growing fast but smaller than Java or Python. Some domains lack mature libraries.
4. **Hiring.** Fewer experienced Rust developers. You may need to train.

### What Rust Gives You

1. **Correctness.** The compiler catches bugs that would reach production in other languages.
2. **Performance.** Consistently fast with no GC pauses. Benchmark results comparable to C and C++.
3. **Refactoring confidence.** If it compiles after a refactor, it probably works. The compiler is a safety net.
4. **Operational simplicity.** Single binary, no runtime dependencies, small Docker images.

## Decision Framework

| Question | If yes, consider Rust |
|----------|----------------------|
| Do you need predictable low latency? | Yes |
| Are memory safety bugs a serious concern? | Yes |
| Is the service long-running and hard to restart? | Yes |
| Does your team have (or want to build) Rust expertise? | Yes |
| Do you need a prototype by next week? | No — use Python or Go |
| Is the project a simple CRUD app? | No — use Go or Node.js |
| Is hiring Rust developers a blocker? | No — use what your team knows |

## Next

You now understand what Rust is and when to use it. The next module (01-first-code) gets you writing actual Rust code.

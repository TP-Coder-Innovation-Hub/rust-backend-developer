``

# Compiler vs Interpreter

Programs are written in human-readable text. The computer only understands binary (0s and 1s). Something must translate your code into machine instructions. There are two approaches: compilation and interpretation.

## Compiled Languages

A compiler translates your entire program into machine code before it runs. You get an executable file (a binary) that the computer runs directly.

```
Source code  →  [Compiler]  →  Executable binary  →  [CPU runs it]
```

Steps:
1. You write `main.rs`
2. You run `rustc main.rs` (the Rust compiler)
3. The compiler produces `main` (an executable)
4. You run `./main`

The compiler catches errors during step 2. If your code has type errors, syntax errors, or ownership violations, the compiler refuses to produce a binary. You fix the errors and compile again.

**Advantages:**
- Catches many bugs before the program ever runs
- The resulting binary runs fast because it is already translated
- No translation overhead at runtime

**Disadvantages:**
- Compilation takes time (Rust can be slow to compile)
- You must recompile after every change
- The binary only works on the platform you compiled it for

Rust, C, and C++ are compiled languages.

## Interpreted Languages

An interpreter reads and executes your code line by line, at runtime. No separate compilation step.

```
Source code  →  [Interpreter reads and runs each line]
```

Steps:
1. You write `app.py`
2. You run `python app.py`
3. The Python interpreter reads each line, translates it, and executes it immediately

**Advantages:**
- Start running immediately, no compile step
- Easy to test small snippets interactively
- Same code runs anywhere the interpreter is available

**Disadvantages:**
- Slower execution (translation happens during runtime)
- Errors are found at runtime, when the interpreter reaches the broken line
- No ahead-of-time type checking

Python, JavaScript, and Ruby are interpreted languages.

## Just-In-Time (JIT) Compilation

Some languages use a hybrid approach. The code is interpreted at first, but frequently-run code gets compiled to machine code during execution. Java and C# work this way. They compile to bytecode first, then the JVM or CLR JIT-compiles to machine code at runtime.

```
Source  →  [Compiler]  →  Bytecode  →  [JIT at runtime]  →  Machine code
```

## Comparison

| Property | Compiled (Rust) | Interpreted (Python) | JIT (Java) |
|----------|----------------|---------------------|------------|
| When errors are found | Before running | During running | Mixed |
| Execution speed | Fast | Slow | Medium to fast |
| Compile time | Yes (slow) | None | Startup cost |
| Portability | Per-platform | Anywhere interpreter runs | Anywhere JVM/CLR runs |
| Type safety | Compile time | Runtime (or not at all) | Compile time |

## Why This Matters for Rust

Rust is compiled ahead-of-time. This has two major implications:

1. **The compiler catches bugs before your code runs.** Rust's compiler checks ownership rules, type correctness, and lifetime validity. Many bugs that would crash a Python program at runtime are impossible in Rust because the compiler rejects the code.

2. **The output is a single static binary.** No runtime dependency. No interpreter. No JVM. You compile once, deploy the binary. This is why Rust is popular for containers and embedded systems — the deployment artifact is tiny and self-contained.

The trade-off is compile time. Rust compiles slowly compared to Go or C. The Rust team is actively improving this, but it remains the most common complaint. You will spend time waiting for the compiler. In exchange, you spend less time debugging runtime errors.

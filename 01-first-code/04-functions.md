`[Entry]`

# Functions

Functions are reusable blocks of code. Every Rust program has at least one function: `main`.

## Basic Syntax

```rust
fn greet() {
    println!("Hello!");
}

fn main() {
    greet();    // calls the function, prints "Hello!"
}
```

Step by step:
- `fn greet()` — declare a function named `greet` with no parameters
- `println!("Hello!")` — the function body
- `greet()` in main — call the function

## Parameters

```rust
fn print_sum(a: i32, b: i32) {
    println!("The sum is: {}", a + b);
}

fn main() {
    print_sum(3, 4);    // prints: The sum is: 7
}
```

Step by step:
- `a: i32` — parameter named `a` of type `i32` (32-bit signed integer)
- `b: i32` — parameter named `b` of type `i32`
- You must declare types for parameters. Rust does not infer parameter types.

## Return Values

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b       // no semicolon — this is the return value
}

fn main() {
    let result = add(5, 3);
    println!("Result: {result}");    // prints: Result: 8
}
```

Step by step:
- `-> i32` — the function returns an `i32`
- `a + b` — the last expression in a function is the return value. No `return` keyword needed.
- **No semicolon on the last expression.** A semicolon turns an expression into a statement, which returns `()` (nothing). This is a common beginner mistake.

You can use the `return` keyword for early returns:

```rust
fn absolute_value(x: i32) -> i32 {
    if x < 0 {
        return -x;     // early return
    }
    x                   // normal return (no semicolon)
}
```

## Statements vs Expressions

This distinction is critical in Rust:

| Concept | Returns a value? | Ends with semicolon? |
|----------|-----------------|---------------------|
| Statement | No | Yes |
| Expression | Yes | No |

```rust
fn main() {
    // Statement — does not return a value
    let x = 5;

    // Expression — returns a value
    let y = {
        let x = 3;
        x + 1       // no semicolon: this block returns 4
    };

    println!("y is: {y}");    // prints: y is: 4
}
```

Step by step:
- The block `{ let x = 3; x + 1 }` is an expression
- Inside the block, `let x = 3;` is a statement (semicolon)
- `x + 1` at the end has no semicolon, so it is the block's return value
- `y` gets the value 4

## Functions as Parameters

```rust
fn apply(value: i32, f: fn(i32) -> i32) -> i32 {
    f(value)
}

fn double(x: i32) -> i32 {
    x * 2
}

fn main() {
    let result = apply(5, double);
    println!("Result: {result}");    // prints: Result: 10
}
```

Step by step:
- `apply` takes a value and a function, calls the function with the value
- `f: fn(i32) -> i32` — parameter `f` is a function that takes an `i32` and returns an `i32`
- `apply(5, double)` — call `apply` with value 5 and function `double`

## Common Types Cheat Sheet

| Type | Description |
|------|-------------|
| `i32` | Signed 32-bit integer (-2B to +2B) |
| `u32` | Unsigned 32-bit integer (0 to 4B) |
| `i64` | Signed 64-bit integer |
| `f64` | 64-bit floating point |
| `bool` | `true` or `false` |
| `char` | Single Unicode character |
| `String` | Heap-allocated string (growable) |
| `&str` | String slice (reference to string data) |
| `()` | Unit type (no value, like `void`) |

## Common Mistakes

1. **Adding a semicolon after the return expression.** This returns `()` instead of the value. The compiler error is clear: "expected `i32`, found `()`."

2. **Forgetting the return type annotation.** `-> i32` is required if the function returns a value. Omit it only for functions that return nothing.

3. **Not declaring parameter types.** `fn add(a, b)` is a compile error. You must write `fn add(a: i32, b: i32)`.

## Next

The next file covers structs and enums — how to define custom data types in Rust.

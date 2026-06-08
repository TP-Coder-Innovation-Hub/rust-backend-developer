``

# Variables and Mutability

Rust variables are immutable by default. You must explicitly opt into mutation. This is the opposite of most languages, and it is one of Rust's best design decisions.

## Immutable by Default

```rust
fn main() {
    let x = 5;
    x = 6; // ERROR: cannot assign twice to immutable variable
}
```

Step by step:
- `let x = 5` — create a variable named `x` and bind the value 5 to it. This variable cannot be changed.
- `x = 6` — the compiler rejects this. `x` is immutable.

Why? Because most variables do not need to change. Making immutability the default prevents accidental mutation bugs. If you read `let x = 5`, you know `x` is 5 forever. No surprises.

## Mutable Variables

Add `mut` to allow changes:

```rust
fn main() {
    let mut x = 5;   // x can change
    println!("x is: {x}");  // prints: x is: 5
    x = 6;           // this is allowed
    println!("x is: {x}");  // prints: x is: 6
}
```

Step by step:
- `let mut x = 5` — create a mutable variable. The `mut` keyword signals: "this value will change." Readers of your code know to watch for mutations.
- `{x}` inside the string — Rust's inline format syntax. It inserts the value of `x` into the string.

Use `mut` when you need it. Use plain `let` when you do not. The compiler will warn you if you mark a variable `mut` but never change it.

## Shadowing

You can declare a new variable with the same name as a previous one. The new variable "shadows" the old one.

```rust
fn main() {
    let x = 5;
    let x = x + 1;     // shadows previous x. New x is 6.
    let x = x * 2;     // shadows again. New x is 12.
    println!("x is: {x}");
}
```

Shadowing vs mutation:

| Property | `let mut x` | Shadowing with `let x` |
|----------|-------------|----------------------|
| Can change value | Yes | Yes (new binding) |
| Can change type | No | Yes |
| Scope aware | Same variable | New variable per `let` |

Shadowing can change the type. Mutation cannot:

```rust
let spaces = "   ";        // type: &str (string slice)
let spaces = spaces.len(); // type: usize (number). Shadowing allows this.

let mut spaces = "   ";
spaces = spaces.len();     // ERROR: expected &str, found usize
```

Step by step:
- First example: `spaces` starts as a string, then gets shadowed as its length. Both are called `spaces` but they are different variables with different types.
- Second example: trying to assign a number to a string variable fails because `mut` does not allow type changes.

## Constants

```rust
const MAX_POINTS: u32 = 100_000;
```

- `const` instead of `let`
- Type annotation required (`u32` — unsigned 32-bit integer)
- Cannot be mutable
- Value must be computable at compile time
- Named in SCREAMING_SNAKE_CASE by convention
- Underscores in numbers are for readability (100_000 == 100000)

Use constants for values that are truly constant: configuration limits, mathematical constants, magic numbers.

## Why Rust Chose This

| Design choice | Reasoning |
|---------------|-----------|
| Immutable by default | Prevents accidental mutation. Makes code easier to reason about. |
| Explicit `mut` | Signals intent. A `mut` variable stands out during code review. |
| Shadowing | Allows type transformations without inventing new variable names. |

In most languages, mutation is the default and you must work to prevent it. Rust flips this. The result: fewer bugs from unexpected state changes, especially in concurrent code.

## Next

The next file covers control flow: how to make decisions and repeat actions in Rust.

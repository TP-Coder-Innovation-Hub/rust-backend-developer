`` ``

# References and Borrowing

Ownership means one variable owns each value. But passing ownership back and forth is tedious. Borrowing lets you use a value without taking ownership.

## The Concept

A **reference** is a pointer to data you do not own. **Borrowing** is creating a reference to a value. The original owner keeps ownership. When the reference is done, the owner still has the value.

Analogy: you borrow a book from a friend. You can read it. You do not own it. When you return it, your friend still has it.

## Immutable References — &T

```rust
fn calculate_length(s: &String) -> usize {
    s.len()
}   // s goes out of scope, but it does not own the data, so nothing is dropped

fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);       // borrow s1

    println!("'{}' has length {}", s1, len);   // s1 is still valid
}
```

Step by step:
- `&s1` — create a reference to `s1`. Does not take ownership.
- `s: &String` — the parameter is a reference to a String, not a String itself.
- `s.len()` — read the length through the reference.
- After the function returns, `s1` is still valid and usable.

The `&` symbol means "borrow this value for reading."

## Mutable References — &mut T

```rust
fn append_world(s: &mut String) {
    s.push_str(", world");
}

fn main() {
    let mut s = String::from("hello");
    append_world(&mut s);             // mutably borrow s
    println!("{s}");                   // prints: hello, world
}
```

Step by step:
- `let mut s` — the variable must be mutable to allow mutable borrows
- `&mut s` — create a mutable reference
- `s: &mut String` — the parameter accepts a mutable reference
- `s.push_str(...)` — modify the String through the reference
- `s` in main still owns the modified String

The `&mut` symbol means "borrow this value for writing."

## The Borrow Rules

> 🖼️ **[IMAGE_PLACEHOLDER]** — Rust borrow checker rules multiple immutable OR one mutable reference

These two rules are enforced at compile time:

1. **At any given time, you can have EITHER multiple immutable references OR exactly one mutable reference.**
2. **References must always be valid** (no dangling references).

### Rule 1: Many Readers OR One Writer

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;        // immutable borrow
    let r2 = &s;        // another immutable borrow — OK, multiple readers allowed
    println!("{r1} {r2}"); // both used here

    let r3 = &mut s;    // mutable borrow — OK, r1 and r2 are no longer used
    r3.push_str(" world");
    println!("{r3}");
}
```

This compiles because `r1` and `r2` are not used after `r3` is created. Rust's borrow checker tracks when references are last used (Non-Lexical Lifetimes, NLL).

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &mut s;    // ERROR: cannot borrow as mutable because it is also borrowed as immutable

    println!("{r1} {r2}");
}
```

This fails. You cannot have a mutable reference while an immutable reference exists. The reason: if you are reading through `r1`, a mutable reference `r2` could change the data under you. That would be a data race in concurrent code.

### Rule 1 Prevents Data Races

A data race occurs when:
1. Two or more pointers access the same data at the same time
2. At least one of them is writing
3. There is no synchronization

Rust's borrow rules make this impossible at compile time. This is why Rust calls concurrency "fearless."

## Rule 2: No Dangling References

```rust
fn dangle() -> &String {       // ERROR: missing lifetime specifier
    let s = String::from("hello");
    &s                          // return reference to local variable
}                               // s is dropped here! reference points to freed memory
```

The compiler catches this. `s` is created inside the function and dropped when the function ends. Returning a reference to it would create a dangling pointer. Rust forbids this.

The fix: return ownership instead:

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s                           // move ownership to caller
}
```

## Reference Cheat Sheet

| Syntax | Meaning | Can read? | Can write? |
|--------|---------|-----------|------------|
| `&T` | Immutable reference | Yes | No |
| `&mut T` | Mutable reference | Yes | Yes |
| `T` | Ownership | Yes | Yes (if `mut`) |

| Rule | Explanation |
|------|------------|
| Many `&T` | Multiple readers allowed |
| One `&mut T` | Only one writer at a time |
| `&T` and `&mut T` cannot coexist | No reading while writing |
| References must be valid | No dangling pointers |

## Next

Sometimes references outlive the data they point to. Lifetimes are how Rust tracks this. The next file explains when and why you need lifetime annotations.

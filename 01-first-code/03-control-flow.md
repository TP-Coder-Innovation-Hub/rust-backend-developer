`[Entry]`

# Control Flow

Rust has the standard control flow constructs plus `match`, which is more powerful than a typical switch statement.

## if / else

```rust
fn main() {
    let score = 85;

    if score >= 90 {
        println!("Grade: A");
    } else if score >= 80 {
        println!("Grade: B");    // this one prints
    } else if score >= 70 {
        println!("Grade: C");
    } else {
        println!("Grade: F");
    }
}
```

Step by step:
- `if score >= 90` — is 85 >= 90? No.
- `else if score >= 80` — is 85 >= 80? Yes. Print "Grade: B". Skip remaining branches.

Key difference from other languages: **the condition must be a `bool`.** Rust will not implicitly convert integers, strings, or other types to boolean. `if 1 { }` is a compile error.

`if` is an expression, not a statement. It returns a value:

```rust
let grade = if score >= 90 { "A" }
    else if score >= 80 { "B" }
    else { "F" };
```

Both branches must return the same type. You cannot return a string from one branch and a number from another.

## loop

`loop` repeats forever until you explicitly break:

```rust
fn main() {
    let mut count = 0;

    loop {
        count += 1;

        if count == 3 {
            println!("Three!");
            break;        // exit the loop
        }
    }

    println!("Final count: {count}");  // prints 3
}
```

Step by step:
- `loop { }` — start an infinite loop
- `count += 1` — increment count (1, 2, 3...)
- When count reaches 3, print and `break` out of the loop

`loop` can return a value:

```rust
let mut counter = 0;
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;    // loop returns 20
    }
};
println!("Result: {result}");
```

## while

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");  // 3!, 2!, 1!
        number -= 1;
    }

    println!("Liftoff!");
}
```

Step by step:
- Check: is `number` (3) not 0? Yes. Print "3!". Decrement to 2.
- Check: is 2 not 0? Yes. Print "2!". Decrement to 1.
- Check: is 1 not 0? Yes. Print "1!". Decrement to 0.
- Check: is 0 not 0? No. Exit loop. Print "Liftoff!".

## for

The most common loop in Rust. Iterates over collections:

```rust
fn main() {
    let fruits = ["apple", "banana", "cherry"];

    for fruit in fruits {
        println!("I like {fruit}");
    }
}
```

Step by step:
- `fruits` is an array of 3 strings
- `for fruit in fruits` — bind each element to `fruit` in turn
- Prints "I like apple", "I like banana", "I like cherry"

Range-based iteration:

```rust
for number in 1..=5 {
    println!("{number}");  // prints 1, 2, 3, 4, 5
}
```

- `1..=5` — inclusive range: 1 through 5
- `1..5` — exclusive range: 1 through 4

Prefer `for` over `while` when you know the number of iterations. It is safer (no off-by-one errors) and more idiomatic.

## match

`match` compares a value against patterns and executes the matching arm:

```rust
fn main() {
    let day = "Monday";

    match day {
        "Monday" | "Tuesday" | "Wednesday" | "Thursday" | "Friday" => {
            println!("Weekday");
        }
        "Saturday" | "Sunday" => {
            println!("Weekend");
        }
        _ => {
            println!("Not a day");   // _ matches anything (default case)
        }
    }
}
```

Step by step:
- Compare `day` ("Monday") against each pattern
- `"Monday"` matches the first arm. Print "Weekday".
- The `|` operator combines multiple patterns (OR)
- `_` is the wildcard: matches anything not covered above

`match` must be exhaustive. If you do not cover all possible values, the compiler rejects your code. The `_` wildcard covers "everything else."

`match` is also an expression:

```rust
let description = match temperature {
    0..=32 => "freezing",
    33..=55 => "cold",
    56..=75 => "pleasant",
    76..=100 => "hot",
    _ => "extreme",
};
```

`match` works with enums, structs, ranges, guards, and destructuring. It is one of Rust's most powerful features and you will use it constantly.

## Comparison

| Construct | Use when |
|-----------|----------|
| `if / else` | Simple boolean conditions |
| `loop` | Repeat until a dynamic condition (or forever) |
| `while` | Repeat while a condition is true |
| `for` | Iterate over a collection or range |
| `match` | Compare against multiple specific patterns |

## Next

The next file covers functions: how to define reusable blocks of code with parameters and return values.

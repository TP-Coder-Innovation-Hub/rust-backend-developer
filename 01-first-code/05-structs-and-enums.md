`[Entry]`

# Structs and Enums

Structs hold data. Enums hold choices. `impl` blocks add behavior. These three constructs define custom types in Rust.

## Structs — Grouping Data

A struct groups related fields into one type:

```rust
struct User {
    username: String,
    email: String,
    age: u32,
    active: bool,
}

fn main() {
    // Create a User
    let user = User {
        username: String::from("alice"),
        email: String::from("alice@example.com"),
        age: 30,
        active: true,
    };

    // Access fields with dot notation
    println!("User: {}, age: {}", user.username, user.age);
}
```

Step by step:
- `struct User { ... }` — define a struct type named `User` with four fields
- Each field has a name and a type, separated by `:`
- `String::from("alice")` — create a `String` from a string literal
- `user.username` — access the `username` field

Mutable structs:

```rust
let mut user = User {
    username: String::from("alice"),
    email: String::from("alice@example.com"),
    age: 30,
    active: true,
};

user.age = 31;      // allowed because user is mut
```

The entire struct is mutable or immutable. You cannot make individual fields mutable.

### Tuple Structs

When you need a named type but field names are unnecessary:

```rust
struct Color(u8, u8, u8);       // RGB values
struct Point(f64, f64, f64);    // x, y, z coordinates

let red = Color(255, 0, 0);
let origin = Point(0.0, 0.0, 0.0);

println!("Red component: {}", red.0);    // access by index
```

### Unit Structs

No fields at all. Useful when you need a type but no data:

```rust
struct AlwaysEqual;
```

## Enums — Defining Choices

An enum represents a value that can be one of several variants:

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn move_player(dir: Direction) {
    match dir {
        Direction::Up => println!("Moving up"),
        Direction::Down => println!("Moving down"),
        Direction::Left => println!("Moving left"),
        Direction::Right => println!("Moving right"),
    }
}

fn main() {
    move_player(Direction::Up);     // prints: Moving up
}
```

Step by step:
- `enum Direction` — define an enum with four variants
- `Direction::Up` — the `Up` variant (use `::` to access variants)
- `match` — handle each variant. Exhaustive: you must handle all variants.

### Enums with Data

Variants can hold data:

```rust
enum Shape {
    Circle(f64),                           // radius
    Rectangle(f64, f64),                   // width, height
    Triangle { base: f64, height: f64 },   // named fields
}

fn area(shape: &Shape) -> f64 {
    match shape {
        Shape::Circle(radius) => {
            3.14159 * radius * radius
        }
        Shape::Rectangle(width, height) => {
            width * height
        }
        Shape::Triangle { base, height } => {
            0.5 * base * height
        }
    }
}

fn main() {
    let shapes = [
        Shape::Circle(5.0),
        Shape::Rectangle(3.0, 4.0),
        Shape::Triangle { base: 6.0, height: 3.0 },
    ];

    for shape in &shapes {
        println!("Area: {:.2}", area(shape));
    }
}
```

Step by step:
- `Shape::Circle(5.0)` — create a Circle variant with radius 5.0
- `Shape::Rectangle(3.0, 4.0)` — create a Rectangle variant with width 3.0 and height 4.0
- In the `match`, `Shape::Circle(radius)` extracts the radius value
- `&shapes` — borrow the array (reference instead of moving it)

### Option — The Most Important Enum

Rust has no `null`. Instead, it uses `Option<T>`:

```rust
enum Option<T> {
    Some(T),    // a value exists
    None,       // no value
}
```

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 1 {
        Some(String::from("Alice"))   // found
    } else {
        None                           // not found
    }
}

fn main() {
    match find_user(1) {
        Some(name) => println!("Found: {name}"),
        None => println!("Not found"),
    }
}
```

The compiler forces you to handle both `Some` and `None`. You cannot forget to check for absence. This eliminates null pointer errors entirely.

## impl — Adding Behavior

`impl` blocks add methods to structs and enums:

```rust
struct Rectangle {
    width: f64,
    height: f64,
}

impl Rectangle {
    // Associated function (like a static method)
    // Called as Rectangle::new(3.0, 4.0)
    fn new(width: f64, height: f64) -> Self {
        Self { width, height }
    }

    // Method — takes &self (borrows the struct)
    fn area(&self) -> f64 {
        self.width * self.height
    }

    // Method that needs mutation
    fn scale(&mut self, factor: f64) {
        self.width *= factor;
        self.height *= factor;
    }
}

fn main() {
    let mut rect = Rectangle::new(3.0, 4.0);
    println!("Area: {}", rect.area());     // 12.0
    rect.scale(2.0);
    println!("New area: {}", rect.area()); // 48.0
}
```

Step by step:
- `fn new(...) -> Self` — an associated function. No `&self` parameter. Like a constructor. Called with `Rectangle::new(...)`.
- `fn area(&self)` — a method. `&self` borrows the struct immutably. Called with `rect.area()`.
- `fn scale(&mut self, factor)` — a method that mutates. `&mut self` borrows mutably.
- `Self` is an alias for the type name (`Rectangle`).

## Struct vs Enum — When to Use Which

| Use | When |
|-----|------|
| Struct | A value has all fields at once (a User has a name AND email AND age) |
| Enum | A value is one of several choices (a Shape is a Circle OR Rectangle OR Triangle) |

## Next

You can write basic Rust programs. The next module (02-ownership-and-safety) teaches the concept that makes Rust unique: ownership.

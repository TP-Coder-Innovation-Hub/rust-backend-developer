`[Entry]`

# Programming Paradigms

A paradigm is a style of organizing code. Different paradigms solve the same problems with different structures. Most modern languages support multiple paradigms. Rust is multi-paradigm but oriented toward systems programming.

## The Four Main Paradigms

### Imperative

You give the computer step-by-step instructions. "Do this, then do that." Most beginners start here because it matches how people think about tasks.

```
SET total TO 0
FOR EACH number IN [1, 2, 3, 4, 5]
    SET total TO total + number
END FOR
PRINT total
```

This adds 1+2+3+4+5 and prints 15. Each line is a command.

### Object-Oriented (OOP)

You group data and behavior together into "objects." Objects communicate by sending messages to each other. Java and C# are OOP-first languages.

```
CLASS Dog
    HAS name
    HAS breed

    METHOD bark()
        PRINT name + " says woof"
    END METHOD
END CLASS

CREATE my_dog AS Dog WITH name="Rex", breed="Shepherd"
CALL my_dog.bark()
```

Key ideas: encapsulation (hide internal details), inheritance (create specialized versions), polymorphism (treat different types the same way).

Rust does not have classes or inheritance. It uses structs for data and traits for shared behavior. This avoids common OOP pitfalls like the diamond problem and fragile base classes.

### Functional

You compose functions. Data flows through a pipeline of transformations. No mutable state. Functions are pure: same input always produces same output.

```
ADD = FUNCTION(a, b) RETURN a + b
MULTIPLY = FUNCTION(a, b) RETURN a * b

numbers = [1, 2, 3, 4, 5]
doubled = MAP(numbers, FUNCTION(x) RETURN MULTIPLY(x, 2))
total = REDUCE(doubled, 0, ADD)
PRINT total
```

`MAP` transforms each element. `REDUCE` combines all elements into one value. The result is 30 (2+4+6+8+10).

Rust supports functional patterns: iterators, map, filter, fold. These are idiomatic and efficient.

### Systems Programming

Direct control over hardware resources: memory, threads, network sockets. You decide exactly where data lives and when it gets freed. C and C++ are traditional systems languages.

Systems programming demands that you understand what the computer is actually doing. There is no garbage collector hiding memory management from you.

## Where Rust Fits

| Paradigm | Rust support |
|----------|-------------|
| Imperative | Full. Control flow, variables, loops. |
| Object-oriented | Partial. No classes or inheritance. Structs + traits instead. |
| Functional | Strong. Iterators, closures, pattern matching. |
| Systems | Full. Direct memory control, no garbage collector. |

Rust is a systems language that borrows good ideas from functional programming. You write imperative code, compose functions like a functional programmer, and control memory like a systems programmer.

## Next

Now that you know how code can be organized, the next file covers the three fundamental building blocks that all paradigms reduce to: sequence, decision, and iteration.

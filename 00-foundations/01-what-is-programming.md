`[Entry]`

# What Is Programming

Computers execute instructions. Programming is writing those instructions in a language the machine can follow.

Think of a recipe. A recipe tells a cook:

1. Get ingredients
2. Chop onions
3. Heat oil to medium
4. Add onions, stir for 5 minutes
5. Serve

Programming works the same way. You write a sequence of steps. The computer follows them exactly, in order, one at a time.

The difference is precision. A recipe can say "cook until golden." A program must say "cook for 300 seconds at 175 degrees." Computers do not guess. They do exactly what you tell them.

## What Code Looks Like

Here is a program in pseudocode (not a real language, just structured English):

```
SET temperature TO 20
IF temperature > 25 THEN
    PRINT "It is hot"
ELSE
    PRINT "It is fine"
END IF
```

Step by step:
- `SET temperature TO 20` — store the value 20 in a named container called "temperature"
- `IF temperature > 25 THEN` — check a condition. Is 20 greater than 25? No.
- `ELSE` — since the condition was false, do this branch instead
- `PRINT "It is fine"` — output text to the screen

Every programming language does three things:
1. **Remember things** — store data in variables
2. **Make decisions** — check conditions and choose different paths
3. **Repeat things** — do the same operation many times

These three building blocks are enough to build any program. Operating systems, web servers, video games — all of them reduce to these three operations.

## Why So Many Languages

Different languages optimize for different things:

| Goal | Example languages |
|------|-------------------|
| Easy to learn | Python, JavaScript |
| Maximum performance | C, C++, Rust |
| Safety and reliability | Rust, Java |
| Concurrent systems | Go, Rust, Erlang |

Rust targets the intersection of performance, safety, and concurrency. It runs as fast as C++ but prevents entire categories of bugs that C++ allows. That is why this guide teaches Rust for backend development.

## What You Will Learn Next

The next file explains programming paradigms — the different ways languages let you organize instructions. Understanding paradigms helps you understand why Rust code looks the way it does.

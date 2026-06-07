`[Entry]`

# Sequential, Decision, Iteration

Every program, no matter how complex, uses only three building blocks:

1. **Sequence** — do one thing after another
2. **Decision** — choose between paths based on a condition
3. **Iteration** — repeat something

Master these three and you can write any program.

## Sequence

Instructions execute in order, top to bottom. This is the default. No special syntax needed.

```
SET width TO 10
SET height TO 5
SET area TO width * height
PRINT area
```

Step by step:
1. Store 10 in `width`
2. Store 5 in `height`
3. Multiply `width` by `height`, store result (50) in `area`
4. Print 50

The order matters. If you swap line 3 and line 4, the program prints nothing because `area` has not been calculated yet.

## Decision

The program checks a condition and takes different paths.

```
SET age TO 17

IF age >= 18 THEN
    PRINT "You can vote"
ELSE IF age >= 16 THEN
    PRINT "You can drive"
ELSE
    PRINT "Too young"
END IF
```

Step by step:
1. Is age (17) >= 18? No. Skip first branch.
2. Is age (17) >= 16? Yes. Print "You can drive". Skip remaining branches.

Only one branch executes. The program never prints "Too young" because the second condition matched first.

## Iteration

The program repeats a block of code.

```
SET countdown TO 5

WHILE countdown > 0
    PRINT countdown
    SET countdown TO countdown - 1
END WHILE

PRINT "Liftoff"
```

Step by step:
1. countdown is 5. Is 5 > 0? Yes. Print 5. countdown becomes 4.
2. countdown is 4. Is 4 > 0? Yes. Print 4. countdown becomes 3.
3. countdown is 3. Is 3 > 0? Yes. Print 3. countdown becomes 2.
4. countdown is 2. Is 2 > 0? Yes. Print 2. countdown becomes 1.
5. countdown is 1. Is 1 > 0? Yes. Print 1. countdown becomes 0.
6. countdown is 0. Is 0 > 0? No. Exit loop.
7. Print "Liftoff".

## Combining the Three

Real programs nest these blocks inside each other. A web server is:

```
LOOP FOREVER                          ← iteration
    REQUEST = wait_for_request()
    
    IF REQUEST.path == "/users" THEN  ← decision
        users = get_users()           ← sequence
        respond(users)
    ELSE IF REQUEST.path == "/health"
        respond("OK")
    ELSE
        respond("Not found", 404)
    END IF
END LOOP
```

That is a web server. Three building blocks. Everything else — databases, authentication, caching — is more of the same, just with more steps inside each block.

## Why This Matters for Rust

Rust gives you specific syntax for each building block:

| Building block | Rust syntax |
|----------------|-------------|
| Sequence | Statements execute top to bottom |
| Decision | `if` / `else if` / `else`, `match` |
| Iteration | `loop`, `while`, `for` |

You will learn the exact Rust syntax in 01-first-code/03-control-flow.md. For now, understand the concept: every program is sequence, decision, and iteration combined.

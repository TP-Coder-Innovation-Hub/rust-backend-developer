``

# Setup

Install Rust, set up an editor, and run your first program. Step by step.

## Step 1: Install rustup

rustup is the Rust installer and version manager. It installs the compiler (`rustc`), the build tool (`cargo`), and the standard library.

Open a terminal and run:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Follow the prompts. Choose the default option (press 1 or Enter).

After installation, restart your terminal. Verify:

```bash
rustc --version
cargo --version
```

You should see version numbers for both. If not, restart your terminal or run `source $HOME/.cargo/env`.

## Step 2: Install an Editor

VS Code with the `rust-analyzer` extension is the recommended setup.

1. Install [VS Code](https://code.visualstudio.com/)
2. Open VS Code
3. Go to Extensions (Cmd+Shift+X on Mac)
4. Search for "rust-analyzer" and install it

rust-analyzer provides:
- Autocompletion
- Inline error messages
- Go to definition
- Rename refactoring

Alternative editors: JetBrains RustRover (dedicated Rust IDE), Neovim with rust-analyzer LSP, Helix (built-in LSP support).

## Step 3: Create Your First Project

Cargo creates and builds Rust projects. Create a new project:

```bash
cargo new hello
cd hello
```

Cargo creates this structure:

```
hello/
  Cargo.toml      # Project metadata and dependencies
  src/
    main.rs       # Your code goes here
```

Open `src/main.rs`. Cargo generated a starter program:

```rust
fn main() {
    println!("Hello, world!");
}
```

Step by step:
- `fn main()` — every Rust program starts here. `main` is the entry point.
- `println!` — prints text to the terminal. The `!` means it is a macro (not a function). You will learn the difference later.
- `"Hello, world!"` — the string to print.

## Step 4: Run It

```bash
cargo run
```

Output:
```
   Compiling hello v0.1.0 (/path/to/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s)
     Running `target/debug/hello`
Hello, world!
```

Step by step:
1. `cargo run` compiles your code (if changed) and runs the binary
2. First run compiles. Subsequent runs are faster if code has not changed.
3. The output "Hello, world!" is from your `println!` call.

## Step 5: Build a Release Binary

```bash
cargo build --release
```

This creates an optimized binary at `target/release/hello`. Release builds are slower to compile but significantly faster to run. Use `cargo run` during development and `cargo build --release` for production.

## Common Cargo Commands

| Command | What it does |
|---------|-------------|
| `cargo new <name>` | Create a new project |
| `cargo run` | Compile and run (dev mode) |
| `cargo build` | Compile without running |
| `cargo build --release` | Compile with optimizations |
| `cargo check` | Check for errors without building (faster) |
| `cargo test` | Run tests |
| `cargo fmt` | Format your code |
| `cargo clippy` | Run the linter |

Use `cargo check` during development. It is faster than `cargo build` because it skips code generation.

## Troubleshooting

- "command not found: cargo" — restart your terminal or run `source $HOME/.cargo/env`
- "linker 'cc' not found" — install a C compiler. On Linux: `sudo apt install build-essential`. On Mac: `xcode-select --install`.
- Compilation errors — copy the error message. Rust errors are detailed and often tell you exactly what to fix.

## Next

Now that your environment works, the next file teaches variables and mutability — the first Rust concept that surprises people coming from other languages.

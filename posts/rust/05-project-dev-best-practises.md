---
title: Learning Rust 05: Project Development Best Practices
date: "2024-08-24"
category: Rust
tags: rust
summary: "Learn about Rust's project organization, cargo, toolchain, modules & 
visibility, crates, packages, workspaces, formatting, and documentation, 
testing, and unsafe code."
---

## Table of Contents

- [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Cargo](#cargo)
  - [Rust Toolchain](#rust-toolchain)
  - [Modules \& Visibility](#modules--visibility)
  - [Prelude](#prelude)
  - [Crates \& Dependencies](#crates--dependencies)
  - [Packages](#packages)
  - [Workspaces](#workspaces)
  - [Formatting](#formatting)
  - [Documentation](#documentation)
  - [Testing](#testing)
  - [Unsafe Code](#unsafe-code)

### Introduction

Rust's project development best practices involve organizing code efficiently,
using tools effectively, and ensuring code quality through testing and
documentation.

### Cargo

Cargo is Rust's package manager and build system. Common commands:

- `cargo build` - Compiles the project and creates an executable
- `cargo run` - Builds and runs the project
- `cargo check` - Checks if code compiles without producing an executable
- `cargo test` - Runs all tests in the project
- `cargo doc` - Generates documentation for the project
- `cargo publish` - Publishes a library to crates.io

### Rust Toolchain

- `rustc` - The Rust compiler
- `rustfmt` - Formats code
- `rust-analyzer` - Language server for IDEs
- `cargo` - Package manager and build system
- `clippy` - Lints code for common mistakes

### Modules & Visibility

Modules help organize code. You can control visibility with `pub`.

Example:

```rust
mod my_module {
    pub fn public_function() {
        println!("This is a public function.");
    }

    fn private_function() {
        println!("This is a private function.");
    }
}

fn main() {
    my_module::public_function();
    // my_module::private_function(); // Error: function is private
}
```

- `self`: Refers to the current module. In the call_self function,
  `self::inner_function()` is used to call `inner_function` within the same
  module (inner).
- `super`: Refers to the parent module. In the call_super function,
  `super::outer_function()` is used to call `outer_function` from the parent
  module (outer).

```rust
mod outer {
    pub mod inner {
        pub fn inner_function() {
            println!("Called inner_function");
        }

        pub fn call_self() {
            self::inner_function();
        }

        pub fn call_super() {
            super::outer_function();
        }
    }

    pub fn outer_function() {
        println!("Called outer_function");
    }
}

fn main() {
    outer::inner::call_self();   // Calls inner_function
    outer::inner::call_super();  // Calls outer_function
}
```

### Prelude

The prelude is a set of items that are automatically imported into every Rust
module. It includes commonly used types, traits, and functions.

Rust's standard prelude includes common utilities like `Option`, `Result` etc.

You can create your own prelude by importing items into a module and then
importing that module into your main file.

Example:

```rust
// prelude.rs
pub fn greet(name: &str) {
    println!("Hello, {}!", name);
}

pub fn farewell(name: &str) {
    println!("Goodbye, {}!", name);
}
```

Using the prelude:

```rust
// main.rs
mod prelude;

use prelude::prelude::*;

fn main() {
    // Using functions from the custom prelude
    greet("Alice");
    farewell("Alice");

    // Rust standard prelude
    let some_option: Option<i32> = Some(5);
    if let Some(value) = some_option {
        println!("The value is: {}", value);
    }
}
```

### Crates & Dependencies

Crates are the basic unit of code distribution. Dependencies are specified in
`Cargo.toml`.

```toml
[dependencies]
rand = "0.8" # Example dependency
```

```rust
// src/main.rs
use rand::Rng;

fn main() {
    let mut rng = rand::thread_rng();
    let random_number: u32 = rng.gen();
    println!("Random number: {}", random_number);
}
```

### Packages

A package is a collection of crates. It includes a `Cargo.toml` file.

```toml
[package]
name = "my_package"
version = "0.1.0"
edition = "2018"

[dependencies]
```

```rust
// my_package/src/main.rs
fn main() {
    println!("Hello from my_package!");
}
```

### Workspaces

Workspaces manage multiple packages with a shared `Cargo.lock`.

Workspace Structure:

```
my_workspace/
├── Cargo.toml
├── package1/
│   └── Cargo.toml
└── package2/
    └── Cargo.toml
```

Workspace Cargo.toml:

```toml
[workspace]
members = ["package1", "package2"]
```

Each package (package1, package2) will have its own `Cargo.toml` file and source
files, but they will share the same `Cargo.lock` and output directory.

### Formatting

Rustfmt is used to format Rust code according to style guidelines. Consistent
formatting helps maintain readability and reduces code review overhead.

Example:

```bash
# Format all Rust files in the current project
cargo fmt
```

### Documentation

Rust provides built-in support for documentation through comments and the
`cargo doc` command, which generates HTML documentation from your code.

Example:

````rust
/// Adds two numbers together.
///
/// # Examples
///
/// ```
/// let sum = add(2, 3);
/// assert_eq!(sum, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
````

To generate documentation:

```bash
cargo doc --open
```

### Testing

Rust's testing framework is built into the language. You can write unit tests,
integration tests, and documentation tests to ensure your code works as
expected.

Example:

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }
}
```

To run tests:

```bash
cargo test
```

### Unsafe Code

Unsafe code bypasses Rust's safety guarantees to perform low-level operations
like raw pointer manipulation and calling unsafe functions. Use it only when
necessary and document why it's needed. Unsafe code must be wrapped in an
`unsafe` block.

Example:

```rust
fn main() {
    let mut num = 5;

    // Create a raw pointer to num
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;

    unsafe {
        // Dereference raw pointers within an unsafe block
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
}
```

Raw pointers (`*const T`, `*mut T`) in Rust are C/C++-like pointers that can
point to any memory location. Dereferencing them requires unsafe blocks since
they bypass Rust's safety guarantees and can cause undefined behavior. They're
primarily used for FFI with C libraries and performance-critical code where
manual safety checks are acceptable.

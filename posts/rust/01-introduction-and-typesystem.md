---
title: "Learning Rust 01: Introduction and Basics"
date: '2024-08-15'
category: Rust
tags: rust
summary: "Learn about Rust's features, installation, variables and mutability, data types, expressions, control flow, functions, modules, and comments."
---


## Table of Contents

- [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
    - [Why Rust?](#why-rust)
    - [When Rust?](#when-rust)
    - [Rust over other languages](#rust-over-other-languages)
    - [Problems with Rust itself](#problems-with-rust-itself)
  - [Installation and Getting Started](#installation-and-getting-started)
  - [Your First Rust Program](#your-first-rust-program)
  - [Variables and Mutability](#variables-and-mutability)
  - [References](#references)
  - [Data Types](#data-types)
  - [Custom Types](#custom-types)
    - [Structs](#structs)
    - [Enums](#enums)
  - [Expressions and Control Flow](#expressions-and-control-flow)
  - [Error Handling](#error-handling)
  - [Comments](#comments)

### Introduction

#### Why Rust?

Rust is a programming language that helps you write faster, and more reliable
software. It combines the low-level control and performance of languages like
C/C++ with modern safety features that prevent common bugs and concurrency
issues, making it an ideal choice for building reliable, high-performance
systems.

Rust is about confidence. Rust is about clarity. I'm very happy that I'm
confident about my code that it won't break suddenly at 4am.

#### When Rust?

Rust is an excellent choice for building **high-performance, low-level
systems**, where **safety** and **concurrency** are key concerns, especially in
**operating systems**, **embedded systems**, **game development**,
**cryptographic applications**, and **networking systems**. If you are working
on performance-critical software and want to avoid common bugs found in
languages like C/C++, Rust ensures that issues like **memory leaks**, **buffer
overflows**, **dangling pointers**, and **data races** are caught at compile
time.

#### Rust over other languages

- **C/C++**: Offers similar performance but lacks Rust's memory safety
  guarantees.
- **Go**: Has runtime overhead due to garbage collection.
- **Java**: Slower due to JVM and garbage collection.
- **Python**: Slower due to dynamic typing and interpreted nature.
- **JavaScript**: Slower because it runs in a virtual machine and is
  interpreted.

#### Problems with Rust itself

**Steep learning curve** due to the ownership model and borrow checker. **Longer
compilation time** due to rigorous checks to ensure safety.

---

### Installation and Getting Started

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Create new project
cargo new hello_world
cd hello_world
```

Project structure:

```
my_project/
├── Cargo.toml     # Dependencies and metadata
├── src/
│   └── main.rs    # Entry point
└── target/        # Build artifacts
```

Rust has many tools to help you build your project:

- **rustc**: the Rust compiler which turns .rs files into binaries and other
  intermediate formats.
- **cargo**: the Rust dependency manager and build tool.
- **rustup**: the Rust toolchain installer and updater.

---

### Your First Rust Program

```rust
fn main() {
    println!("Hello, world!");
}
```

Key concepts:

- `main()`: Program entry point
- `println!`: Macro for formatted output
- `;`: Statement terminator
- `{}`: Scope blocks

Build and run:

```bash
cargo run              # Development
cargo build --release  # Production
```

---

### Variables and Mutability

Variable is a name given to a memory location. In Rust, variables are immutable
by default.

```rust
// immutable variable
let x = 5;
```

```rust
// mutable variable
let mut x = 5;
x = 6;
```

**Const** is a variable that cannot be changed.

```rust
const PI: f64 = 3.14;
```

**Static** is a variable that lives for the entire program execution and cannot
be moved. It is accessible from any part of the module where it is declared.

```rust
static mut COUNTER: u32 = 0;
```

**Shadowing** allows redeclaring a variable with the same name.

```rust
let x = 5;
let x = x + 1; // x = 6
```

---

### References

References in Rust come in two flavors: shared (`&`) and exclusive (`&mut`).

```rust
// Shared references (multiple allowed)
let x = 5;
let r1 = &x;    // OK
let r2 = &x;    // OK - multiple shared references allowed
println!("{} {}", r1, r2);

// Exclusive references (only one allowed)
let mut y = 5;
let m1 = &mut y;    // OK
// let m2 = &mut y; // Error - only one mutable reference allowed
*m1 = 6;           // Dereference to modify
```

**Slices** are references to a contiguous sequence of elements:

```rust
let array = [1, 2, 3, 4, 5];
let slice = &array[1..4];  // References elements 2, 3, 4

// String slices
let string = String::from("hello world");
let hello = &string[0..5];  // "hello"
let world = &string[6..];   // "world"
```

**Reference Validity**: The Rust compiler ensures references are always valid:

- References cannot outlive their referent
- Mutable references cannot coexist with other references
- References must always point to valid data

---

### Data Types

Rust is a statically typed language, which means all variables must have a known
type at compile time. The compiler can usually infer the type, but explicit type
annotations are sometimes needed.

```rust
// Scalar Types
let integer: i32 = 42;      // i8 to i128, u8 to u128
let float: f64 = 3.14;      // f32, f64
let boolean: bool = true;    // true, false
let character: char = 'A';   // Unicode scalar

// Compound Types
let tuple: (i32, f64) = (42, 3.14);
let array: [i32; 3] = [1, 2, 3];
let slice: &[i32] = &array[1..];

// String Types
let str_literal: &str = "Hello";           // String slice
let string: String = String::from("Hello"); // Owned string
```

---

### Custom Types

#### Structs

Structs are custom data types that let you package related values together. Rust
offers three types of structs:

```rust
// Named struct
struct Person {
    name: String,
    age: u32,
    is_student: bool
}

// Tuple struct
struct Point(i32, i32);

// Unit struct (no fields)
struct Unit;

// Implementing methods
impl Person {
    // Constructor (convention: new)
    fn new(name: String, age: u32) -> Person {
        Person {
            name,
            age,
            is_student: false
        }
    }

    // Instance method
    fn celebrate_birthday(&mut self) {
        self.age += 1;
    }
}

// Usage and destructuring
let mut alice = Person::new(String::from("Alice"), 25);
alice.celebrate_birthday();

let point = Point { x: 0, y: 7 };
let Point { x, y } = point;  // Destructuring
println!("x: {}, y: {}", x, y);

// Pattern matching with struct
match point {
    Point { x: 0, y } => println!("On y axis: {}", y),
    Point { x, y: 0 } => println!("On x axis: {}", x),
    Point { x, y } => println!("At coordinates: ({}, {})", x, y),
}
```

Structs are fundamental to organizing code in Rust, offering:

- Strong type safety
- Encapsulation of related data
- Method implementation through `impl` blocks
- Pattern matching capabilities
- Derivable traits (`Debug`, `Clone`, etc.)

#### Enums

Enums in Rust are incredibly powerful and form the backbone of Rust's type
system. They can encode variants, hold data, and implement methods:

```rust
// Basic enum
enum Result<T, E> {
    Ok(T),    // Success variant with data
    Err(E),   // Error variant with data
}

// Complex enum example
enum Message {
    Quit,                       // Unit variant
    Move { x: i32, y: i32 },   // Named fields
    Write(String),             // Single value
    ChangeColor(i32, i32, i32) // Multiple values
}

// Implementing methods and pattern matching
impl Message {
    fn call(&self) {
        // Destructuring in match
        match self {
            Message::Quit => println!("Quitting"),
            Message::Move { x, y } => println!("Moving to ({}, {})", x, y),
            Message::Write(text) => println!("Text message: {}", text),
            Message::ChangeColor(r, g, b) => println!("Color: RGB({},{},{})", r, g, b),
        }
    }
}

// Usage with destructuring
let msg = Message::Move { x: 3, y: 4 };
if let Message::Move { x, y } = msg {
    println!("Moving to ({}, {})", x, y);
}
```

Rust's enums are particularly powerful because they:

- Can contain data of any type
- Support pattern matching exhaustively
- Enable null-safety through `Option<T>`
- Handle errors elegantly with `Result<T, E>`
- Allow encoding of state machines
- Can implement traits and methods

---

### Expressions and Control Flow

1. **Block Expressions**:

   ```rust
   let y = {
       let x = 3;
       x + 1  // No semicolon = return value
   };  // y = 4
   ```

2. **If Expressions**:

   ```rust
   // As expression
   let number = if condition { 5 } else { 6 };

   // As control flow
   if number < 5 {
       println!("less than 5");
   } else if number > 10 {
       println!("greater than 10");
   } else {
       println!("between 5 and 10");
   }
   ```

3. **Match Expressions**:

   ```rust
   match value {
       // Match literals
       1 => println!("One"),

       // Match multiple values
       2 | 3 => println!("Two or three"),

       // Match range
       4..=6 => println!("Four through six"),

       // Catch-all pattern
       _ => println!("Something else"),
   }

   // if let for single pattern
   if let Some(x) = optional_value {
       println!("Got value: {}", x);
   }

   // while let for repeated matching
   while let Some(top) = stack.pop() {
       println!("Top: {}", top);
   }
   ```

4. **Loops**:

   ```rust
   // loop with break value
   let result = loop {
       counter += 1;
       if counter == 10 { break counter; }
   };

   // while loop
   while number != 0 {
       number -= 1;
   }

   // for loop
   for number in 1..4 {
       println!("{}!", number);
   }
   ```

---

### Error Handling

Rust provides robust error handling through its type system and helpful compiler
error messages that guide you to fix issues with detailed explanations and
suggestions.

```rust
// Result for recoverable errors
fn divide(x: i32, y: i32) -> Result<i32, String> {
    if y == 0 { return Err("Division by zero".into()); }
    Ok(x / y)
}

// Option for nullable values
let numbers = vec![1, 2, 3];
let first: Option<&i32> = numbers.first();

// Error propagation
fn process() -> Result<i32, String> {
    let result = divide(10, 2)?;  // Returns Err early
    Ok(result * 2)
}
```

---

### Comments

```rust
// Single line comment

/* Multi-line comment */

/// Doc comment for items (functions, structs, etc.)
/// Supports markdown

//! Doc comment for the module itself
```

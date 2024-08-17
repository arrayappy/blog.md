---
title: Learning Rust 02: Abstractions and Metaprogramming
date: "2024-08-17"
category: Rust
tags: rust
summary: "Learn about Rust's generics, traits, trait objects, and macros."
---

## Table of Contents

- [Table of Contents](#table-of-contents)
  - [Methods and Traits](#methods-and-traits)
    - [Methods](#methods)
    - [Traits](#traits)
    - [Deriving](#deriving)
  - [Generics](#generics)
    - [Generic Functions](#generic-functions)
    - [Generic Data Types](#generic-data-types)
    - [Generic Traits](#generic-traits)
    - [Trait Bounds](#trait-bounds)
    - [impl Trait](#impl-trait)
    - [dyn Trait](#dyn-trait)
  - [Object-Oriented Concepts in Rust](#object-oriented-concepts-in-rust)
  - [Macros](#macros)
    - [Declarative Macros](#declarative-macros)
    - [Procedural Macros](#procedural-macros)
    - [Functions vs Macros](#functions-vs-macros)

### Methods and Traits

#### Methods

Methods are functions associated with a type (usually a struct or enum). They
are defined within an `impl` block and can take `self` as their first parameter.

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Associated function (static method)
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }

    // Instance method
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // Mutable method
    fn resize(&mut self, width: u32, height: u32) {
        self.width = width;
        self.height = height;
    }
}
```

Types of methods:

- **Associated functions** (like `new`): Don't take `self`, called with `::`
- **Instance methods**: Take `&self` or `&mut self`, called with `.`
- **Consuming methods**: Take ownership with `self`

#### Traits

Traits define shared behavior across types, similar to interfaces in other
languages. They specify method signatures that implementing types must provide.

```rust
// Define a trait
trait Animal {
    // Required method
    fn make_sound(&self) -> String;

    // Default method implementation
    fn description(&self) -> String {
        String::from("An animal")
    }
}

// Implement the trait
struct Dog {
    name: String,
}

impl Animal for Dog {
    fn make_sound(&self) -> String {
        String::from("Woof!")
    }

    // Override default implementation
    fn description(&self) -> String {
        format!("A dog named {}", self.name)
    }
}
```

**Common Traits:**

- `Display`: Custom string formatting
- `Debug`: Debug formatting with `{:?}`
- `Clone`: Explicit value duplication
- `Copy`: Implicit value duplication
- `Default`: Default value creation
- `PartialEq`, `Eq`: Equality comparison
- `PartialOrd`, `Ord`: Ordering comparison

#### Deriving

Rust can automatically implement common traits using the `#[derive]` attribute:

```rust
#[derive(Debug, Clone, Copy, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```

Common derivable traits:

- `Debug`: Enables debug printing
- `Clone`, `Copy`: Value duplication
- `PartialEq`, `Eq`: Equality comparison
- `PartialOrd`, `Ord`: Ordering
- `Hash`: Hash computation
- `Default`: Default values

### Generics

#### Generic Functions

Generic functions work with multiple types while maintaining type safety. Type
parameters are specified in angle brackets:

```rust
fn print_value<T: std::fmt::Display>(value: T) {
    println!("Value: {}", value);
}

// Multiple type parameters
fn swap<T>(a: T, b: T) -> (T, T) {
    (b, a)
}
```

#### Generic Data Types

Structs and enums can be generic over types:

```rust
// Generic struct
struct Pair<T> {
    first: T,
    second: T,
}

// Generic enum
enum Result<T, E> {
    Ok(T),
    Err(E),
}

// Implementation for generic type
impl<T> Pair<T> {
    fn new(first: T, second: T) -> Self {
        Pair { first, second }
    }
}
```

#### Generic Traits

Traits can be generic and can be implemented for generic types:

```rust
trait Container<T> {
    fn contains(&self, item: &T) -> bool;
    fn add(&mut self, item: T);
}

struct Stack<T> {
    items: Vec<T>,
}

impl<T: PartialEq> Container<T> for Stack<T> {
    fn contains(&self, item: &T) -> bool {
        self.items.contains(item)
    }

    fn add(&mut self, item: T) {
        self.items.push(item)
    }
}
```

#### Trait Bounds

Trait bounds specify what functionality a type must provide. They're used to
restrict generic types to those that implement specific traits:

```rust
// Single trait bound
fn print<T: Display>(value: T) {
    println!("{}", value);
}

// Multiple trait bounds using +
fn print_debug<T: Display + Debug>(value: T) {
    println!("{} {:?}", value, value);
}

// Using where clause for clearer bounds
fn process<T, U>(t: T, u: U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // Implementation
    0
}
```

#### impl Trait

`impl Trait` is a way to specify a return type that implements a trait without
naming the concrete type:

```rust
// Return type that implements Iterator
fn counter() -> impl Iterator<Item = i32> {
    (0..5).into_iter()
}

// Useful for closures
fn get_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}

// Can't return different types implementing same trait
fn returns_closure(a: i32) -> impl Fn(i32) -> i32 {
    if a > 0 {
        |x| x + 1  // Works
    } else {
        |x| x - 1  // Same type as above
    }
}
```

#### dyn Trait

`dyn Trait` is used for dynamic dispatch, allowing different types implementing
the same trait to be used interchangeably at runtime:

```rust
trait Drawable {
    fn draw(&self);
}

struct Circle {
    radius: f64,
}

impl Drawable for Circle {
    fn draw(&self) {
        println!("Drawing circle with radius {}", self.radius);
    }
}

struct Square {
    side: f64,
}

impl Drawable for Square {
    fn draw(&self) {
        println!("Drawing square with side {}", self.side);
    }
}

// Using trait objects with dyn
fn draw_shapes(shapes: Vec<Box<dyn Drawable>>) {
    for shape in shapes {
        shape.draw();
    }
}

// Usage
let shapes: Vec<Box<dyn Drawable>> = vec![
    Box::new(Circle { radius: 1.0 }),
    Box::new(Square { side: 2.0 }),
];
```

Key differences:

- `impl Trait`: Static dispatch, better performance, compile-time resolution
- `dyn Trait`: Dynamic dispatch, runtime flexibility, slight performance
  overhead

### Object-Oriented Concepts in Rust

1. Structs instead of classes: Rust doesn't have classes, but it has something
   similar: structs. Structs are used to create custom data types, and they can
   contain data just like a class in C++. Implementing methods: You can add
   methods to structs using impl (short for "implement") blocks. This is
   somewhat like adding methods to a class in C++.
2. Traits as interfaces: Rust uses traits instead of interfaces or abstract base
   classes. A trait describes behavior that types can have in common. When a
   type impl-ements a trait, it guarantees it provides the behavior declared by
   that trait.
3. No inheritance: Rust doesn't have inheritance, which is a core feature of
   many OOP languages. However, you can use traits to achieve polymorphism and
   share common behavior between different types. When combined with enum, this
   gives you a very powerful way of expressing complex hierarchies without
   inheritance.
4. Composition over inheritance: Rust encourages composition over inheritance.
   Rather than inheriting properties from a "parent" class, you build complex
   functionality by combining simpler, independent pieces.

### Macros

Macros of Rust are similar to functions but they are executed at compile time.
There are two types of macros: declarative and procedural.

#### Declarative Macros

Used for pattern matching and code generation.

Declarative macros are defined using the `macro_rules!` macro.
$literal is a
placeholder for a literal value, $expr is a placeholder for an expression,
$ident
is a placeholder for an identifier, $ty is a placeholder for a type, $pat is a
placeholder for a pattern, $block is a placeholder for a block of code.

There are different tokens that can be used in the macro:

- `$x:expr` matches any expression
- `$x:literal` matches any literal
- `$x:ident` matches any identifier
- `$x:ty` matches any type
- `$x:stmt` matches any statement
- `$x:pat` matches any pattern
- `$x:path` matches any path

Example:

```rust
macro_rules! sum {
    ($x:expr, $y:expr) => {
        $x + $y
    };
}

fn main() {
    let result = sum!(1, 2);
    println!("Result: {}", result);
}
```

#### Procedural Macros

More advanced macros, usually implemented in separate crates, that operate on
the abstract syntax tree (AST) of the code.

- Custom Derive: For automatically implementing traits (e.g., #[derive(Debug)]).
- Attribute-like macros: Used to create custom attributes (e.g., #[some_macro]).
- Function-like macros: Can be invoked with arguments like a regular function
  but work at the syntax level.

For this example, we will create a simple derive-like procedural macro that
automatically adds a greet method to any struct.

First, you'll need two crates:

- A procedural macro crate: This is where the macro logic will reside.
- A main crate: This will use the procedural macro.

```bash
cargo new procedural_macro_example --lib
cargo new procedural_macro_example_main --bin
```

**Procedural Macro Crate**

1. Add the dependencies in `Cargo.toml` of the `procedural_macro_example` crate:

```toml
[dependencies]
syn = "1.0"
quote = "1.0"
proc-macro2 = "1.0"

[lib]
proc-macro = true
```

1. Create the macro in `src/lib.rs`:

```rust
extern crate proc_macro;

use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Greeter)]
pub fn greeter_derive(input: TokenStream) -> TokenStream {
    // Parse the input tokens into a syntax tree
    let input = parse_macro_input!(input as DeriveInput);

    // Extract the struct's name
    let struct_name = input.ident;

    // Generate the code for the greet method
    let expanded = quote! {
        impl #struct_name {
            pub fn greet(&self) {
                println!("Hello from the {}!", stringify!(#struct_name));
            }
        }
    };

    // Convert the generated code into a TokenStream and return it
    TokenStream::from(expanded)
}
```

**Using the Procedural Macro in the Main Crate**

1. Add the procedural macro crate to the `Cargo.toml` of the
   `procedural_macro_example_main` crate:

```toml
[dependencies]
procedural_macro_example = { path = "../procedural_macro_example" }
```

2. Add the `Greeter` attribute to the struct in `src/main.rs`:

```rust
// src/main.rs
use procedural_macro_example::Greeter;

#[derive(Greeter)]
struct Person {
    name: String,
}

fn main() {
    let person = Person { name: "Alice".to_string() };
    person.greet();
}
```

#### Functions vs Macros

- Function run at runtime but macros run at compile time
- Functions accept fixed number of args but macros can receive dynamic number of
  args
- Downside is complex code meaning that you are writing code that writes other
  code which will be harder to read, understand and maintain

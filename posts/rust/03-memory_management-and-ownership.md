---
title: "Learning Rust 03: Memory Management and Ownership"
date: '2024-08-19'
category: Rust
tags: rust
summary: "Learn about Rust's memory management, ownership, borrowing, lifetimes, smart pointers, and interior mutability."
---

## Table of Contents

- [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Smart Pointers](#smart-pointers)
  - [Interior Mutability](#interior-mutability)
  - [Lifetimes](#lifetimes)

### Introduction

Programs allocate memory in two ways. Stack, continuous allocation of memory for
fixed sized variables. Heap, dynamic sized variables determined at runtime.

Languages like C/C++ give programmers full control over memory but are
error-prone. Others like Java and Python manage memory automatically but add
runtime overhead. Rust takes a unique approach by enforcing memory safety at
compile-time, providing both control and safety.

**Ownership:**

Each value in Rust has a single owner at a time. Ownership can be moved, and the
previous owner becomes invalid.

```rust
let s1 = String::from("Hello");
let s2 = s1;  // Ownership moved
```

**Borrowing:**

References (`&T` for immutable, `&mut T` for mutable) provide temporary access
without ownership transfer. Borrowing ensures safe access without data races or
dangling references.

Rust enforces strict borrowing rules:

1. You can have:

   - One mutable reference (`&mut T`) **OR**
   - Multiple immutable references (`&T`) at a time.
   - Use `T` if you need to move/drop the data.

2. References must always be valid.

```rust
let s = String::from("Hello");
let s_ref = &s; // Immutable borrow
```

Check [interior mutability](#interior-mutability) to see how to mutate data
behind a immutable reference.

**Ownership & Borrowing Relationship**:

- Ownership ensures memory safety; borrowing provides temporary, safe access
  without ownership transfer.
- Rules:
  - One mutable reference or multiple immutable references, but not both at the
    same time.
  - References must be valid.

**Move Semantics**:

An assignment will transfer ownership between variables:

```rust
let s1 = String::from("Hello");
let s2 = s1;  // Ownership moved
```

When `s1` goes out of scope, it owns nothing, and when `s2` goes out of scope,
the data is freed.

**Copy and Clone**:

- **`Copy`**: For fixed-size types (e.g., `integers`, `bool`, `char`).
  - Performs a **bitwise copy**, meaning it simply duplicates the value in
    memory without allocating new heap memory.
    ```rust
    let x = 5;
    let y = x;  // `x` is copied, no move happens, both `x` and `y` are independent.
    ```
- **`Clone`**: Explicit deep copying for more complex types.
  - Performs a **deep copy**, meaning it allocates new memory and copies the
    data.
    ```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();  // `s2` is a separate copy, and `s1` remains valid.
    ```

**Drop**

The `Drop` trait is automatically implemented for all types that have owned
resources. It allows you to define custom cleanup logic when the resource is no
longer needed.

```rust
struct MyStruct {
    data: String,
}

impl Drop for MyStruct {
    fn drop(&mut self) {
        // This code runs automatically when MyStruct goes out of scope
        println!("Dropping MyStruct with data: {}", self.data);
    }
}
```

---

### Smart Pointers

**`Box<T>`**: A heap-allocated pointer with single ownership.

- Store heap-allocated data, especially for recursive types like trees.
  ```rust
  let boxed = Box::new(5); // Storing an integer on the heap
  println!("{}", boxed);
  ```

**`Rc<T>` (Reference Counted)**: Shared ownership of immutable data within a
single-threaded context.

- Use when multiple owners need read-only access.

  ```rust
  use std::rc::Rc;

  let shared = Rc::new(5); // Single owner
  let shared2 = Rc::clone(&shared); // Shared ownership
  println!("{}", shared); // Both share ownership and can be used
  ```

**`Arc<T>` (Atomic Reference Counted)**: Thread-safe version of `Rc`.

- Use in multithreaded environments.

  ```rust
  use std::sync::Arc;
  use std::sync::Mutex;

  let counter = Arc::new(Mutex::new(0));
  let counter_clone = Arc::clone(&counter);

  // Use in a thread-safe context
  ```

**Owned Trait Objects**:

Owned trait objects in Rust, such as `Box<dyn Trait>`, `Rc<dyn Trait>`, and
`Arc<dyn Trait>` are smart pointers that take ownership of heap-allocated data
implementing a trait, managing its lifecycle automatically. Unlike borrowed
trait objects (`&dyn Trait`), they enable dynamic dispatch with safe,
predictable memory management, supporting single ownership (`Box`) or shared
ownership in single-threaded (`Rc`) and multi-threaded (`Arc`) contexts.

---

### Interior Mutability

Interior mutability allows you to modify data even when it’s behind an immutable
reference. Common patterns:

**`Cell<T>`**: Provides interior mutability for `Copy` types (like integers,
`bool`, `char`) in a single-threaded context. Cell copies the data in and out.

```rust
use std::cell::Cell;

let cell = Cell::new(5);
cell.set(10); // Mutate the value inside the cell
println!("{}", cell.get()); // Access the modified value
```

**`RefCell<T>`** : Provide interior mutability with runtime borrow-checking. Use
for non-`Copy` types (like structs).

```rust
use std::cell::RefCell;

let cell = RefCell::new(5);
*cell.borrow_mut() = 10; // Mutate the value inside
println!("{}", cell.borrow()); // Access the modified value
```

**`Mutex<T>`**: Provides interior mutability with thread-safety (exclusive
access).

```rust
use std::sync::Mutex;

let mutex = Mutex::new(5);
let mut data = mutex.lock().unwrap(); // Acquire the lock
*data = 10; // Mutate the value inside the mutex
println!("{}", *data); // Access the modified value
```

**`RwLock<T>`**: Provides safe read-write locks for concurrent access across
threads.

```rust
use std::sync::RwLock;

let rwlock = RwLock::new(5);
let read = rwlock.read().unwrap(); // Multiple threads can read concurrently
let mut write = rwlock.write().unwrap(); // Only one thread can write at a time
*write = 10;
```

**Difference Between Box, Rc, Arc, Cell, RefCell, Mutex, RwLock**

- **Box**: For single ownership. A great use case is to use this when we want to
  store primitive types (stored on stack) on the heap.
- **Rc**: For multiple ownership, for single-threaded contexts.
- **Arc**: For multiple ownership, with thread-safety.
- **Cell**: For "interior mutability" for `Copy` types; that is, when you need
  to mutate something behind a `&T`. `Cell` is similar to `RefCell`, except that
  instead of giving references to the inner value, the value is copied in and
  out of the `Cell`.
- **RefCell**: For "interior mutability"; that is, when you need to mutate
  something behind a `&T`.
- **Mutex**: Offers interior mutability that’s safe to use across threads.
- **RwLock**: Safe read-write locks for concurrent access in multithreaded
  environments.

---

### Lifetimes

The Rust compiler tracks how long references are valid to ensure memory safety.

A lifetime defines how long a reference is valid, with the borrow checker
ensuring references never outlive their values. While usually implicit,
lifetimes can be explicitly annotated (like `&'a Point`). Lifetimes become more
complicated when considering passing values to and returning values from
functions.

**Lifetime Elision**: Rust often infers lifetimes based on the function
signature.

The rules are:

- **Rule 1**: Each reference parameter gets its own lifetime.
- **Rule 2**: If there’s one reference input, the output lifetime is the same as
  the input.
- **Rule 3**: For multiple input references, the output reference must specify
  which input lifetime it depends on.

**Lifetime Types**:

- `&`: Implicit lifetime inferred by the compiler.
- `'a`: Explicit user-defined lifetime.
- `'static`: Longest possible lifetime (used for static data).
- `fn bounded_lifetime<'a, 'b: 'a>` – `'b` must outlive `'a`.

**Struct Lifetimes**:

```rust
struct Point {
    x: i32, // x is an i32
    y: i32, // y is an i32
}

struct Point<'a> {
    x: &'a i32, // x is a reference to an i32
    y: &'a i32, // y is a reference to an i32
}
```

The first `Point` struct owns its data directly, while the second stores
references and requires lifetime annotations to ensure the references remain
valid.

References:

- https://whiztal.io/rust-tips-rc-box-arc-cell-refcell-mutex/

---

<!--
### **4. Slices and Strings**

- **`&[T]` (Slices)**: Borrow part of a collection (like an array or vector).
  - Example:
    ```rust
    let arr = [1, 2, 3];
    let slice = &arr[0..2];
    ```
- **`String` vs. `&str`**:
  - `String` is owned and mutable.
  - `&str` is borrowed and immutable. -->

---
title: Learning Rust 04: Concurrency and Parallelism
date: "2024-08-22"
category: Rust
tags: rust
summary: "Learn about Rust's concurrency and parallelism, including threads, 
channels, and async programming."
---

## Table of Contents

- [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Threads](#threads)
  - [Channels](#channels)
  - [Send and Sync](#send-and-sync)
  - [Shared State](#shared-state)
  - [Async Programming](#async-programming)
    - [Async/Await Syntax](#asyncawait-syntax)
    - [Runtimes](#runtimes)
    - [Concurrency Primitives and Tasks](#concurrency-primitives-and-tasks)
  - [Futures \& Tasks](#futures--tasks)
  - [Common Pitfalls](#common-pitfalls)
  - [Further Reading](#further-reading)

### Introduction

Concurrency involves doing or appearing to do multiple tasks simultaneously,
while parallelism involves literally executing tasks at the same time using
multiple CPU cores. In sequential execution, tasks are completed one after
another. Concurrent execution interleaves tasks, potentially improving
responsiveness but not overall completion time. Parallel execution uses multiple
processors to perform tasks simultaneously, speeding up the process.

In this guide, we'll cover:

- Basic thread management and synchronization
- Thread safety with Send and Sync traits
- Shared state and inter-thread communication
- Async programming with futures and tasks
- Common pitfalls and best practices

Let's start with the fundamental building blocks: threads.

### Threads

Rust provides two types of threads:

- **Plain threads**: Standard threads that run independently
- **Scoped threads**: Share scope from parent thread, allowing access to local
  variables

```rust
use std::thread;

// Plain thread example
thread::spawn(|| {
    println!("Running in a separate thread");
});

// Scoped thread example
thread::scope(|scope| {
    scope.spawn(|| {
        println!("Can access parent thread's variables");
    });
});
```

### Channels

Channels enable safe communication between threads. Rust provides both bounded
and unbounded channels through `mpsc` (Multi-Producer, Single-Consumer).

```rust
use std::sync::mpsc;

// Unbounded channel
let (tx, rx) = mpsc::channel();

// Bounded channel (with capacity 3)
let (tx, rx) = mpsc::sync_channel(3);

// Usage example
thread::spawn(move || {
    tx.send("Hello from thread").unwrap();
});

println!("Received: {}", rx.recv().unwrap());
```

Key differences:

- **Unbounded**: Never blocks on `send()`
- **Bounded**: Blocks when channel is full
- Both support multiple producers via `tx.clone()`

### Send and Sync

How does Rust know if a type is safe to send to another thread?

The answer is `Send` and `Sync` unsafe traits.

A type `T` is `Send` if it is safe to move a `T` value to another thread.

A type `T` is `Sync` if it is safe to access a `T` value from multiple threads
at the same time. More precisely, a type `T` is `Sync` if and only if `&T` is
`Send`. Send.

### Shared State

Rust provides several types for safe shared state:

1. **Arc (Atomic Reference Counting)**

`Arc<T>` allows shared read-only access via `Arc::clone`.

```rust
use std::sync::Arc;

let shared = Arc::new(vec![1, 2, 3]);
let clone = Arc::clone(&shared);
```

2. **Mutex (Mutual Exclusion)**

`Mutex<T>` provides a thread-safe way to access immutable data.

```rust
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
let counter_clone = Arc::clone(&counter);

thread::spawn(move || {
    *counter_clone.lock().unwrap() += 1;
});
```

3. **RwLock (Read-Write Lock)**

`RwLock<T>` provides safe read-write locks for concurrent access across threads.

```rust
use std::sync::RwLock;

let data = RwLock::new(5);
// Multiple readers
let _r1 = data.read().unwrap();
// Single writer
let mut _w = data.write().unwrap();
```

### Async Programming

Async Rust allows concurrent execution without threads, using an async runtime
to manage tasks. This approach is efficient for handling many concurrent tasks
with low memory overhead. Async programming provides fine-grained control over
task execution, offering features like cancellation and various concurrency
constructs.

#### Async/Await Syntax

In Rust, `async` is used to define asynchronous functions, and `await` is used
to pause execution until a future is ready. This allows writing asynchronous
code in a sequential style.

```rust
async fn process_data() -> Result<(), Error> {
    let data = fetch_data().await?;
    process(data).await?;
    Ok(())
}
```

#### Runtimes

Rust's async runtimes manage task execution and scheduling, interacting with the
OS for async IO. Unlike other languages, Rust doesn't have a built-in runtime,
offering flexibility with options like:

- **Tokio**: Feature-rich and production-ready.
- **async-std**: Similar to the standard library.
- **smol**: Lightweight alternative.

```rust
#[tokio::main]
async fn main() {
    let task = tokio::spawn(async {
        println!("Running async task");
    });

    task.await.unwrap();
}
```

#### Concurrency Primitives and Tasks

Concurrency in Rust involves composing futures and tasks. Unlike threads, tasks
are parallelizable futures. Rust's async runtimes, like Tokio, provide tools for
managing these tasks.

- **Join**: Waits for all futures to complete, similar to `Promise.all` in
  JavaScript.

  ```rust
  use anyhow::Result;
  use futures::future;
  use reqwest;
  use std::collections::HashMap;

  async fn size_of_page(url: &str) -> Result<usize> {
      let resp = reqwest::get(url).await?;
      Ok(resp.text().await?.len())
  }

  #[tokio::main]
  async fn main() {
      let urls = ["https://google.com", "https://httpbin.org/ip", "https://play.rust-lang.org/", "BAD_URL"];
      let futures_iter = urls.iter().map(size_of_page);
      let results = future::join_all(futures_iter).await;
      let page_sizes: HashMap<_, _> = urls.iter().zip(results).collect();
      println!("{:?}", page_sizes);
  }
  ```

- **Select**: Waits for any future to complete, similar to `Promise.race`.

  ```rust
  use tokio::sync::mpsc;
  use tokio::time::{sleep, Duration};

  #[tokio::main]
  async fn main() {
      let (tx, mut rx) = mpsc::channel(32);
      let listener = tokio::spawn(async move {
          tokio::select! {
              Some(msg) = rx.recv() => println!("got: {msg}"),
              _ = sleep(Duration::from_millis(50)) => println!("timeout"),
          };
      });
      sleep(Duration::from_millis(10)).await;
      tx.send(String::from("Hello!")).await.expect("Failed to send greeting");

      listener.await.expect("Listener failed");
  }
  ```

### Futures & Tasks

Futures represent deferred computations and are the core of async concurrency.
They can be combined to form larger futures. An async task is a future that is
executed, often managed by a runtime like Tokio.

- **Future Example**:

```rust
use futures::future;

// Join (like Promise.all)
let results = future::join_all(futures).await;

// Select (like Promise.race)
tokio::select! {
    result = future1 => handle_result(result),
    _ = timeout => handle_timeout(),
}

// Task spawning
use tokio::task;
let handle = task::spawn(async {
    println!("Running async task");
});
handle.await.unwrap();
```

### Common Pitfalls

1. **Blocking the Executor**: Avoid blocking the executor with CPU-bound tasks.
   Use `tokio::task::spawn_blocking` for such tasks.

   ```rust
   // Bad: Blocks the async executor
   std::thread::sleep(Duration::from_secs(1));

   // Good: Async sleep
   tokio::time::sleep(Duration::from_secs(1)).await;
   ```

2. **Async Cancellation**: Futures can be dropped at any `.await` point. Use
   `JoinHandle` to prevent premature cancellation.

   ```rust
   use tokio::time;

   async fn count_to(count: i32) {
       for i in 0..count {
           println!("Count in task: {i}!");
           time::sleep(time::Duration::from_millis(100)).await;
       }
   }

   #[tokio::main]
   async fn main() {
       let handle = tokio::spawn(count_to(10)); // Hold onto the handle

       time::sleep(time::Duration::from_millis(250)).await;
       println!("Main task finished!");

       handle.await.unwrap(); // Wait for the spawned task to finish
   }
   ```

3. **Async Traits**: Use the `async_trait` crate for async traits, but be aware
   of performance overhead due to heap allocations.

   ```rust
   #[async_trait]
   trait AsyncProcessor {
       async fn process(&self) -> Result<(), Error>;
   }
   ```

### Further Reading

For more advanced topics and detailed explanations, consider exploring:

- The Rust Async Book: https://rust-lang.github.io/async-book/
- Tokio documentation: https://tokio.rs/

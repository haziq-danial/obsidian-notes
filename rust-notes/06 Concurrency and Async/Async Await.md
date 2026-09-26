---
tags: [rust, async, concurrency]
---

# Async/Await

> [!summary] Summary
> `async`/`await` lets Rust code perform many concurrent operations — typically I/O-bound ones like network requests — using far fewer OS threads than one-thread-per-task, by compiling `async` functions into state machines that an executor drives cooperatively.

![[async-await-state-machine.svg]]

## 1. Why async, when threads already exist

OS threads (see [[Threads and Message Passing]]) work well for CPU-bound parallelism, but each thread carries real overhead (megabytes of stack space, OS scheduling cost) — spawning tens of thousands of OS threads to handle tens of thousands of concurrent network connections doesn't scale well. `async` tasks are much lighter-weight (their state lives in a small, heap-allocated state machine rather than a full OS stack), allowing a handful of OS threads to multiplex thousands of concurrent async tasks.

## 2. Basic syntax

```rust
async fn fetch_data(url: &str) -> Result<String, reqwest::Error> {
    let response = reqwest::get(url).await?;
    response.text().await
}

#[tokio::main]
async fn main() {
    let result = fetch_data("https://example.com").await;
    println!("{result:?}");
}
```

An `async fn` doesn't run its body immediately when called — calling it produces a **Future**, a value representing a computation that hasn't completed yet. Nothing happens until that future is `.await`ed (or otherwise polled by an executor).

## 3. Futures are state machines

```rust
trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<Self::Output>;
}

enum Poll<T> {
    Ready(T),
    Pending,
}
```

The compiler transforms an `async fn`'s body into a state machine, with one state per `.await` point — each `.await` is a potential suspension point where, if the awaited future isn't ready yet, control returns to the executor (`Poll::Pending`) instead of blocking the OS thread. When the underlying resource (e.g., a socket) becomes ready, a **waker** notifies the executor to poll that task again, resuming exactly where it left off.

> [!note] Why this needs no OS-level context switch
> Because the entire "call stack" of a suspended async task is just its state machine's data sitting on the heap (not an OS thread's stack), suspending and resuming a task is comparatively cheap — no OS-level context switch, no dedicated stack memory reserved per task. This is the core efficiency gain over one-OS-thread-per-task.

## 4. Executors — async needs a runtime

Unlike some languages, Rust's standard library deliberately provides **only** the `Future` trait and `async`/`await` syntax — no bundled executor. You need an async runtime crate to actually drive futures to completion:

| Runtime | Notes |
|---|---|
| **tokio** | the dominant, most widely used async runtime — multi-threaded work-stealing scheduler by default, huge ecosystem (networking, timers, filesystem, sync primitives) |
| **async-std** | designed to mirror the standard library's own API shapes |
| **smol** | a minimal, lightweight runtime |

```rust
#[tokio::main]                    // macro that sets up a tokio runtime and runs this as the entry point
async fn main() {
    // ...
}
```

> [!warning] Mixing runtimes / blocking calls in async code
> Calling a blocking (synchronous, long-running) function directly inside an `async fn` blocks the underlying OS thread the executor is using — starving every other task scheduled on that thread, since the executor never gets a chance to switch to something else. Runtimes provide dedicated escape hatches (e.g., tokio's `spawn_blocking`) to run genuinely blocking work on a separate thread pool without stalling the async scheduler.

## 5. Concurrent execution: join! and spawn

```rust
// Run two futures concurrently, wait for both
let (a, b) = tokio::join!(fetch_data(url1), fetch_data(url2));

// Spawn a task onto the runtime, running independently in the background
let handle = tokio::spawn(async move {
    fetch_data(url3).await
});
let result = handle.await.unwrap();
```

`.await`ing futures sequentially, one after another, runs them one at a time (even though each individually yields control while waiting) — `join!` or separately `spawn`ing tasks is what actually achieves concurrency between multiple async operations.

## 6. async vs threads — when to use which

| | Threads | async |
|---|---|---|
| Best for | CPU-bound parallelism | I/O-bound concurrency (many connections/requests) |
| Overhead per unit of work | High (MB-scale stack, OS scheduling) | Low (state machine on the heap) |
| Ecosystem requirement | None — standard library only | Needs a runtime crate (tokio, etc.) |
| Blocking calls | Fine — that's what OS threads are for | Must avoid — use spawn_blocking or async-native alternatives |

Many real systems use **both**: an async runtime (tokio) handling thousands of concurrent I/O-bound connections, occasionally dispatching genuinely CPU-heavy work to a separate thread pool (`spawn_blocking` or `rayon`) so it doesn't stall the async scheduler.

## See also
- [[Threads and Message Passing]]
- [[Shared State Concurrency]]
- [[Essential Crates]] — tokio

#rust #async #concurrency

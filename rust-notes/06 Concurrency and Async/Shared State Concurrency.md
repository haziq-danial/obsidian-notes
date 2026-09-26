---
tags: [rust, concurrency, mutex, arc]
---

# Shared State Concurrency

> [!summary] Summary
> When multiple threads genuinely need to access the **same** data (rather than passing ownership via a channel, see [[Threads and Message Passing]]), Rust requires that access to be mediated by synchronization primitives like `Mutex` — and, unusually, the type system itself enforces that you can't forget to lock.

![[smart-pointers-comparison.svg]]

## 1. Mutex<T> — mutual exclusion

```rust
use std::sync::Mutex;

let m = Mutex::new(5);

{
    let mut num = m.lock().unwrap();   // blocks until the lock is acquired
    *num = 6;
}   // MutexGuard's Drop releases the lock automatically here

println!("m = {m:?}");
```

> [!note] The lock IS the data, not a separate thing next to it
> Unlike many languages where a mutex is a separate object you must remember to lock before touching the *actual* shared variable (and can forget to, or lock the wrong mutex for the wrong data), Rust's `Mutex<T>` **wraps** the data it protects. There is no way to get at the inner `T` without going through `.lock()` — the type system makes "accessing shared data without holding its lock" a compile error, not just a code-review discipline.

`.lock()` returns a `MutexGuard<T>`, a smart pointer (see [[Box RC and Interior Mutability]]) that derefs to the inner data and automatically releases the lock via `Drop` when it goes out of scope — you cannot forget to unlock, since there's no manual unlock call to forget.

## 2. Combining Mutex with Arc for shared ownership across threads

A `Mutex` alone doesn't let multiple threads *own* the same data — for that, wrap it in `Arc` (atomic reference-counted pointer, the thread-safe counterpart to `Rc`, see [[Box RC and Interior Mutability]]):

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);   // clones the Arc (cheap, bumps ref count), not the data
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("Result: {}", *counter.lock().unwrap());   // 10
```

`Arc<Mutex<T>>` is one of the most common patterns in real-world concurrent Rust: `Arc` provides shared ownership across threads, `Mutex` provides safe mutual-exclusion access to the shared data inside.

## 3. Deadlocks — the one thing the compiler can't prevent

> [!warning] Rust prevents data races, not deadlocks
> Rust's compile-time guarantees rule out **data races** (undefined-behavior-causing concurrent access) entirely, but a **deadlock** — two threads each waiting on a lock the other holds — is still entirely possible and is a runtime/logical bug the compiler does not catch:
> ```rust
> // Thread 1: locks A, then tries to lock B
> // Thread 2: locks B, then tries to lock A
> // → both threads block forever
> ```
> The standard mitigation is a **consistent lock ordering** convention across the whole codebase (always acquire locks in the same defined order, e.g. always A before B), enforced by code review/design discipline rather than the compiler.

## 4. RwLock — multiple readers or one writer

```rust
use std::sync::RwLock;

let lock = RwLock::new(5);
{
    let r1 = lock.read().unwrap();     // any number of concurrent readers
    let r2 = lock.read().unwrap();
    println!("{} {}", *r1, *r2);
}
{
    let mut w = lock.write().unwrap();  // exclusive — blocks until all readers/writers finish
    *w += 1;
}
```

`RwLock<T>` mirrors the `&T`/`&mut T` borrowing rules (see [[References and Borrowing]]) but enforced at **runtime** across threads rather than at compile time within one thread — useful when reads vastly outnumber writes, since concurrent readers don't block each other the way a plain `Mutex` would serialize them.

## 5. Atomics — lock-free primitives for simple values

For simple counters/flags, atomic types (`AtomicUsize`, `AtomicBool`, etc.) provide lock-free, hardware-instruction-backed operations — lower overhead than a `Mutex` for simple cases, at the cost of a more restrictive, lower-level API (`.load()`, `.store()`, `.fetch_add()`, with explicit memory-ordering parameters).

## See also
- [[Threads and Message Passing]]
- [[Box RC and Interior Mutability]]
- [[Async Await]]

#rust #concurrency #mutex

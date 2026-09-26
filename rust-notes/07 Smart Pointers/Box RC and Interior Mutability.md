---
tags: [rust, smart-pointers, box, rc, refcell]
---

# Box, Rc, and Interior Mutability

> [!summary] Summary
> When strict single-owner [[Ownership]] is too restrictive for a given data structure, Rust's standard library offers smart pointers as controlled escape hatches: `Box` for simple heap allocation, `Rc`/`Arc` for shared ownership, and `RefCell`/`Mutex` for mutability despite shared, immutable-looking access ("interior mutability").

![[smart-pointers-comparison.svg]]

## 1. Box<T> — simple heap allocation, single owner

```rust
let b = Box::new(5);   // an i32 on the heap instead of the stack
println!("{b}");        // auto-derefs transparently
```

`Box<T>` is the simplest smart pointer: it just puts a value on the heap, with a single owner, and no extra runtime bookkeeping (no reference count, no borrow tracking) — the ownership rules from [[Ownership]] apply exactly as they would to a stack value.

### Why Box is needed for recursive types

```rust
enum List {
    Cons(i32, List),    // ← compile error: infinite size — List directly contains a List
    Nil,
}
```

```rust
enum List {
    Cons(i32, Box<List>),   // fine — Box is a fixed-size pointer, regardless of what it points to
    Nil,
}
```

Rust must know every type's size at compile time. A directly-recursive type has no finite size (`List` contains a `List` contains a `List`, forever) — `Box<T>` breaks the cycle because a pointer's size is always fixed (one machine word), regardless of what it points to.

### Box for trait objects
`Box<dyn Trait>` is also the standard way to own a trait object on the heap — see [[Trait Objects and Dynamic Dispatch]].

## 2. Rc<T> — shared ownership (single-threaded)

```rust
use std::rc::Rc;

let a = Rc::new(5);
let b = Rc::clone(&a);   // cheap — just increments the reference count, no data copied
println!("count: {}", Rc::strong_count(&a));   // 2
```

`Rc<T>` ("reference counted") allows **multiple owners** of the same heap data — each `Rc::clone` bumps a count, and the data is only dropped once the count reaches zero (the last owner goes out of scope). Useful for data structures like graphs or trees where a node might legitimately need multiple parents/owners and it's not obvious which one should be considered the sole owner.

> [!warning] Rc<T> is not thread-safe
> `Rc`'s reference count is a plain, non-atomic integer — incrementing/decrementing it from multiple threads simultaneously is a data race. The compiler enforces this: `Rc<T>` doesn't implement `Send` (see [[Threads and Message Passing]] §4), so attempting to share one across threads is a compile error, not a runtime bug. Use `Arc<T>` (atomic Rc) instead for anything crossing thread boundaries — same API, atomic reference count, slightly higher overhead.

## 3. RefCell<T> — interior mutability

```rust
use std::cell::RefCell;

let data = RefCell::new(5);
{
    let mut val = data.borrow_mut();   // runtime-checked mutable borrow
    *val += 1;
}
println!("{}", data.borrow());
```

`RefCell<T>` moves the borrow-checking rules from [[References and Borrowing]] (either many shared borrows or one exclusive borrow) from **compile time to runtime**. `.borrow()`/`.borrow_mut()` panic if the rules would be violated (e.g., calling `.borrow_mut()` while a `.borrow()` is still alive), rather than being caught by the compiler.

> [!example] Why trade compile-time checking for runtime checking?
> The compiler's static borrow checker is conservative — some patterns are actually safe but the compiler can't prove it (a classic case: a struct wanting to mutate one field while holding an immutable reference derived from reading another field, in a way that's logically sound but not visible to the borrow checker's local analysis). `RefCell` lets you express these patterns, accepting the trade-off that a genuine violation becomes a runtime panic instead of a compile error — useful specifically in the narrow cases where you're confident the access pattern is correct but can't get the compiler to agree.

## 4. Combining them: Rc<RefCell<T>>

```rust
use std::rc::Rc;
use std::cell::RefCell;

let shared = Rc::new(RefCell::new(vec![1, 2, 3]));

let shared2 = Rc::clone(&shared);
shared2.borrow_mut().push(4);

println!("{:?}", shared.borrow());   // [1, 2, 3, 4] — both handles see the same underlying data
```

This combination — **multiple owners, each able to mutate** — is one of the most common non-trivial patterns in single-threaded Rust: `Rc` provides the shared ownership, `RefCell` provides the interior mutability neither alone can offer. Its thread-safe counterpart is `Arc<Mutex<T>>` (see [[Shared State Concurrency]]), the same idea with atomic reference counting and a real lock instead of a runtime-panicking borrow check.

## 5. Cell<T> — interior mutability for Copy types

For simple `Copy` types, `Cell<T>` is a lighter-weight alternative to `RefCell<T>`: it allows getting/setting the value without ever handing out a reference at all (just `.get()`/`.set()`, copying the value in and out), so there's no borrow-checking (even at runtime) needed or possible — appropriate specifically when `T: Copy` and you never need a reference into the data itself.

## 6. Reference cycles — the memory leak Rc *can* cause

`Rc<RefCell<T>>` structures that reference each other (e.g., a doubly-linked structure where each node holds an `Rc` to its neighbor) can form a **reference cycle**: each node keeps the other's count above zero forever, so neither is ever dropped — a genuine memory leak, one of the few Rust allows (memory-safe, but definitely a leak). The standard fix is `Weak<T>` (`Rc::downgrade`), a non-owning reference that doesn't keep the count alive, used for back-references in cyclic structures (e.g., a child's pointer back to its parent).

## See also
- [[Ownership]]
- [[Shared State Concurrency]]
- [[Deref and Drop Traits]]

#rust #smart-pointers

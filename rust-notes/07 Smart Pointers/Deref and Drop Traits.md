---
tags: [rust, deref, drop, traits]
---

# Deref and Drop Traits

> [!summary] Summary
> `Deref` and `Drop` are the two traits that make Rust's smart pointers feel like ordinary values: `Deref` lets a smart pointer be used almost exactly like the thing it points to, and `Drop` lets it run custom cleanup logic automatically when it goes out of scope.

## 1. Deref — treating a smart pointer like a reference

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> { MyBox(x) }
}

impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T {
        &self.0
    }
}

let b = MyBox::new(5);
println!("{}", *b);   // works because `*b` desugars to `*(b.deref())`
```

Implementing `Deref` is what lets the `*` operator (and, via **deref coercion**, method calls and function arguments) work transparently through a custom smart pointer — this is exactly how the standard library's `Box<T>`, `Rc<T>`, and `RefCell::borrow()`'s guard types all let you call methods on the wrapped value directly, without manual unwrapping.

### Deref coercion

```rust
fn hello(name: &str) { println!("Hello, {name}!"); }

let m = MyBox::new(String::from("Rust"));
hello(&m);   // &MyBox<String> → &String → &str, automatically, via chained Deref coercion
```

Rust automatically inserts as many `.deref()` calls as needed to make types match at a function call or method call site — this is why you can pass `&String` where `&str` is expected, and `&Box<T>` where `&T` is expected, without any manual conversion syntax.

## 2. Drop — cleanup on scope exit

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

{
    let c = CustomSmartPointer { data: String::from("my stuff") };
    println!("CustomSmartPointer created.");
}   // "Dropping CustomSmartPointer with data `my stuff`!" printed here, automatically
```

`Drop::drop` is called automatically when a value goes out of scope — this is the mechanism underlying all of Rust's automatic resource management: `Box`/`Vec`/`String` free their heap allocation, `File` closes its file handle, `MutexGuard` releases its lock (see [[Shared State Concurrency]]), all via `Drop` implementations, with no garbage collector and no explicit `close()`/`free()` calls required from the programmer.

### You cannot call .drop() manually

```rust
let c = CustomSmartPointer { data: String::from("stuff") };
c.drop();   // ← compile error: explicit destructor calls not allowed
```

Calling `Drop::drop` directly is forbidden because it would leave the value's memory in a state where the automatic end-of-scope drop would run *again* on the same value — a **double free**. To force early cleanup, use `std::mem::drop(c)` instead, a free function that simply takes ownership of the value and immediately lets it go out of scope, triggering the (single, correctly-ordered) automatic drop early.

```rust
let c = CustomSmartPointer { data: String::from("stuff") };
drop(c);   // fine — forces the value to be dropped right here, not at the end of the block
println!("dropped before the end of main!");
```

## 3. Drop order

Values are dropped in the reverse of their declaration order within a scope (last declared, first dropped) — mirroring how a stack unwinds. For struct fields, drop order follows field declaration order. This ordering matters when one resource's cleanup logic depends on another still being valid (e.g., a wrapper that must flush before closing an underlying file handle).

## 4. RAII — the broader pattern this enables

`Drop` is Rust's implementation of **RAII** (Resource Acquisition Is Initialization, a term from C++): tying a resource's lifetime directly to an object's scope, so acquiring the resource happens in a constructor-equivalent and releasing it happens automatically and unconditionally (even during a panic's stack unwinding, see [[Panics and Unwinding]]) via `Drop`. This is the single unifying mechanism behind memory deallocation, file handle closing, lock releasing, and any other "must clean this up eventually" resource in Rust — there is no separate `finally`/`using`/`with` construct needed, because scope exit itself is the cleanup trigger.

## See also
- [[Box RC and Interior Mutability]]
- [[Ownership]]
- [[Panics and Unwinding]]

#rust #deref #drop

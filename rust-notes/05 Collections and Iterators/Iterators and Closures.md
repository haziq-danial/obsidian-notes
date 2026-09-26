---
tags: [rust, iterators, closures]
---

# Iterators and Closures

> [!summary] Summary
> Iterators provide a lazy, composable way to process sequences, and closures (anonymous functions that capture their environment) are what make iterator adapters so expressive. Both compile down to code as efficient as a hand-written loop — another instance of Rust's zero-cost abstraction philosophy.

## 1. Closures

```rust
let add_one = |x: i32| x + 1;              // type annotations optional, usually inferred
let is_even = |x: &i32| x % 2 == 0;

let y = 10;
let add_y = |x: i32| x + y;                 // captures `y` from the environment
println!("{}", add_y(5));                    // 15
```

Closures capture their environment automatically, in the least restrictive way the body requires (by reference first, falling back to by-value/move only if needed) — or explicitly forced with `move`:

```rust
let data = vec![1, 2, 3];
let closure = move || println!("{data:?}");   // `move` forces ownership of `data` into the closure
```

`move` is required whenever a closure needs to outlive the scope it was created in (e.g., passed to a new thread — see [[Threads and Message Passing]]), since without it the closure would only borrow `data`, and a borrow can't outlive its source.

### The three closure traits
| Trait | What the closure can do |
|---|---|
| `FnOnce` | can be called at least once; may consume captured variables |
| `FnMut` | can be called multiple times; may mutate captured variables |
| `Fn` | can be called multiple times; only reads captured variables |

Every closure implements at least `FnOnce`; most closures that don't move/consume anything also implement `FnMut` and `Fn`. Function parameters accepting a closure choose the least restrictive trait bound their use case requires, maximizing which closures callers can pass in.

## 2. The Iterator trait

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // dozens of default methods (map, filter, fold, ...) built on top of next()
}
```

Every iterator adapter, no matter how elaborate the chain, ultimately reduces to repeated calls to this one `next()` method — everything else (`.map()`, `.filter()`, `.zip()`, `.fold()`, …) is a default-implemented convenience method built on top of it.

```rust
let v = vec![1, 2, 3];
let v_iter = v.iter();          // yields &i32 — borrows, doesn't consume v
let v_into = v.into_iter();     // yields i32 — takes ownership, consumes v
let mut v2 = vec![1, 2, 3];
let v_mut = v2.iter_mut();      // yields &mut i32 — allows mutating in place
```

## 3. Laziness — iterators do nothing until consumed

```rust
let v = vec![1, 2, 3];
let doubled = v.iter().map(|x| x * 2);   // nothing has happened yet — map() is lazy
let result: Vec<i32> = doubled.collect(); // NOW the iteration actually runs
```

> [!note] Why laziness matters
> Because adapters like `.map()`/`.filter()` don't do any work until a **consuming** method (`.collect()`, `.sum()`, `for`, `.fold()`, …) actually drives the iterator, long chains of adapters process each element exactly once through the whole pipeline rather than materializing an intermediate collection at every step — the compiler can (and does) optimize the entire chain into code as tight as a manually written loop with no intermediate allocations.

## 4. Common iterator adapters

```rust
let v = vec![1, 2, 3, 4, 5, 6];

let evens: Vec<_> = v.iter().filter(|&&x| x % 2 == 0).collect();
let sum: i32 = v.iter().sum();
let doubled: Vec<i32> = v.iter().map(|x| x * 2).collect();
let first_over_3 = v.iter().find(|&&x| x > 3);          // Option<&i32>
let any_negative = v.iter().any(|&x| x < 0);              // bool
let total: i32 = v.iter().fold(0, |acc, x| acc + x);      // manual reduction
let pairs: Vec<_> = v.iter().zip(v.iter().skip(1)).collect(); // sliding pairs
let enumerated: Vec<_> = v.iter().enumerate().collect();  // (index, &value) pairs
```

## 5. Iterators vs manual loops — the zero-cost claim

```rust
// Manual loop
let mut sum = 0;
for i in 0..v.len() {
    if v[i] % 2 == 0 {
        sum += v[i];
    }
}

// Iterator chain — compiles to equivalent (often identical) machine code
let sum: i32 = v.iter().filter(|&&x| x % 2 == 0).sum();
```

The iterator version is not only more concise but also **eliminates bounds-check-prone manual indexing** — the compiler can frequently prove iterator-based bounds are respected and elide redundant runtime checks that a manual index-based loop might not as easily be optimized around.

## See also
- [[Common Collections]]
- [[Functions and Control Flow]]
- [[Traits]]

#rust #iterators #closures

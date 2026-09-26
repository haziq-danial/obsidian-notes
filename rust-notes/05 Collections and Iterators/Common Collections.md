---
tags: [rust, collections]
---

# Common Collections

> [!summary] Summary
> Rust's standard library provides growable, heap-allocated collections for the most common data-organization needs: `Vec<T>` (a growable array), `String` (a growable UTF-8 text buffer), and `HashMap<K, V>` (a hash table) are the three used in the overwhelming majority of programs.

## 1. Vec<T> — a growable array

```rust
let mut v: Vec<i32> = Vec::new();
v.push(1);
v.push(2);

let v2 = vec![1, 2, 3];   // macro shorthand with initial values

let third = &v2[2];             // panics if out of bounds
let third_safe = v2.get(2);      // returns Option<&i32> — None if out of bounds, no panic
```

- Owns its elements; stored contiguously on the heap, growing (reallocating, typically doubling capacity) as needed.
- Iterating and indexing are both bounds-checked; `.get()` is the panic-free alternative to `[]` indexing.

> [!warning] Can't hold multiple mutable references into the same Vec at once
> ```rust
> let mut v = vec![1, 2, 3, 4, 5];
> let first = &v[0];
> v.push(6);              // ← compile error: cannot borrow `v` as mutable
> println!("{first}");     //   while it's immutably borrowed by `first`
> ```
> This isn't the borrow checker being overly cautious: `push` might need to reallocate the entire backing buffer to a new memory location, which would leave `first` pointing at freed memory — exactly the dangling-reference scenario [[References and Borrowing]] exists to prevent.

## 2. String — growable UTF-8 text

```rust
let mut s = String::from("hello");
s.push_str(", world");
s.push('!');

let s2 = String::from("foo") + &String::from("bar");   // + takes ownership of the left operand
let s3 = format!("{s2}-{s}");    // format! never takes ownership — the idiomatic way to combine
```

> [!warning] No direct indexing into a String
> ```rust
> let s = String::from("héllo");
> let c = s[0];   // ← compile error: String cannot be indexed by an integer
> ```
> Since `String` is UTF-8 encoded, a byte index doesn't necessarily correspond to one character (`é` is 2 bytes) — direct integer indexing would be misleading about what it actually returns, so Rust disallows it entirely. Use `.chars()`, `.bytes()`, or `&s[start..end]` slicing (with the UTF-8-boundary caveat from [[Slices]]) instead.

## 3. HashMap<K, V>

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let score = scores.get("Blue").copied().unwrap_or(0);   // returns Option<&i32>

for (key, value) in &scores {
    println!("{key}: {value}");
}
```

### The entry API — idiomatic conditional insertion

```rust
let text = "hello world wonderful world";
let mut word_count = HashMap::new();

for word in text.split_whitespace() {
    let count = word_count.entry(word).or_insert(0);
    *count += 1;
}
```

`entry(key).or_insert(default)` returns a mutable reference to the value, inserting `default` first only if the key wasn't already present — avoiding a separate "check if exists, then insert-or-update" step (and the awkward double-lookup or borrow-checker friction that would otherwise require).

> [!note] Ownership and HashMap keys/values
> Inserting an owned value (like a `String`) into a `HashMap` **moves** it into the map — the map now owns it, per ordinary [[Ownership]] rules. Inserting references instead requires the referenced data to outlive the map (see [[Lifetimes]]).

## 4. Other standard collections

| Type | Use case |
|---|---|
| `VecDeque<T>` | double-ended queue — efficient push/pop at both ends |
| `HashSet<T>` | unique, unordered elements (a `HashMap<T, ()>` conceptually) |
| `BTreeMap<K, V>` / `BTreeSet<T>` | like `HashMap`/`HashSet` but kept sorted by key, at the cost of O(log n) instead of amortized O(1) operations |
| `BinaryHeap<T>` | priority queue — efficient access to the maximum element |
| `LinkedList<T>` | rarely the right choice — a `Vec`/`VecDeque` nearly always outperforms it due to cache locality; included mainly for completeness |

> [!tip] Default choice: Vec
> Unless you have a specific reason (need key lookup → HashMap; need sorted order → BTreeMap; need both-end queue operations → VecDeque), `Vec<T>` is almost always the right default collection in Rust — it has excellent cache locality and the lowest overhead of the growable collections.

## See also
- [[Slices]]
- [[Iterators and Closures]]
- [[Ownership]]

#rust #collections

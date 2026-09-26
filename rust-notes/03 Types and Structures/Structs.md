---
tags: [rust, structs]
---

# Structs

> [!summary] Summary
> A struct groups related named data into a single custom type — Rust's equivalent of a class's data (without inheritance), with behavior added separately via `impl` blocks and [[Traits]].

## 1. Defining and instantiating

```rust
struct User {
    username: String,
    email: String,
    active: bool,
    sign_in_count: u64,
}

let user1 = User {
    username: String::from("alice"),
    email: String::from("alice@example.com"),
    active: true,
    sign_in_count: 1,
};
```

### Field init shorthand and struct update syntax

```rust
fn build_user(email: String, username: String) -> User {
    User { email, username, active: true, sign_in_count: 1 }  // shorthand: field == variable name
}

let user2 = User {
    email: String::from("bob@example.com"),
    ..user1   // take all other fields from user1
};
```

> [!warning] Struct update syntax moves non-Copy fields
> `..user1` **moves** any non-`Copy` fields (like `username: String`) out of `user1` into the new struct — after this, `user1` as a whole can no longer be used if any of its moved-out fields were used in `user2`, per ordinary [[Ownership]] rules. Only `Copy` fields (like `active: bool`) remain independently usable in both.

## 2. Tuple structs and unit structs

```rust
struct Point(i32, i32, i32);          // tuple struct — fields accessed by index
let origin = Point(0, 0, 0);
println!("{}", origin.0);

struct Marker;                         // unit struct — no fields at all
let m = Marker;                        // useful for implementing a trait with no data needed
```

Tuple structs are useful when field names would add no clarity and you mainly want a distinct **type** (e.g., `Meters(f64)` vs `Feet(f64)` — same underlying representation, but the type system now prevents accidentally mixing them up, a lightweight form of the "newtype pattern").

## 3. Methods via impl blocks

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // associated function (no `self`) — acts like a "static method" / constructor
    fn square(size: u32) -> Self {
        Self { width: size, height: size }
    }

    // method — takes &self, borrows the instance
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // method taking &mut self — can mutate the instance
    fn double(&mut self) {
        self.width *= 2;
        self.height *= 2;
    }
}

let sq = Rectangle::square(10);        // associated function, called via ::
println!("{}", sq.area());              // method, called via .
```

- `&self` — borrows the instance immutably; the method only reads.
- `&mut self` — borrows mutably; the method can modify fields.
- `self` (no `&`) — takes ownership of the instance; the method consumes it (common for builder-style methods that transform and return a new value).

Multiple `impl` blocks for the same struct are allowed and commonly used to group methods logically (e.g., separating trait implementations from inherent methods).

## 4. Deriving common traits

Rather than hand-implementing behavior like equality, ordering, cloning, or debug-printing, Rust lets you `#[derive(...)]` many standard traits automatically when every field itself supports them:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash, Default)]
struct Point {
    x: i32,
    y: i32,
}

let p1 = Point { x: 1, y: 2 };
println!("{:?}", p1);          // Debug — requires the {:?} formatter
let p2 = p1.clone();
assert_eq!(p1, p2);              // PartialEq
let default_point = Point::default(); // Default — { x: 0, y: 0 }
```

| Derivable trait | Enables |
|---|---|
| `Debug` | `{:?}` formatting for debugging output |
| `Clone` | `.clone()` — explicit deep copy |
| `Copy` | implicit bitwise copy instead of move (requires all fields also be `Copy`) |
| `PartialEq` / `Eq` | `==` comparison |
| `PartialOrd` / `Ord` | `<`, `>`, sorting |
| `Hash` | usable as a `HashMap`/`HashSet` key |
| `Default` | `Type::default()` — a sensible zero-value instance |

## See also
- [[Traits]]
- [[Enums and Pattern Matching]]
- [[Generics]]

#rust #structs

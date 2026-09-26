---
tags: [rust, generics]
---

# Generics

> [!summary] Summary
> Generics let you write code that works over many types without duplicating it per type, while remaining fully type-checked at compile time and imposing **zero runtime cost** via a compilation technique called monomorphization.

## 1. Generic functions

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

let numbers = vec![34, 50, 25, 100, 65];
println!("{}", largest(&numbers));    // works for i32

let chars = vec!['y', 'm', 'a', 'q'];
println!("{}", largest(&chars));       // and for char — same function, no duplication
```

`T: PartialOrd` is a **trait bound** (see [[Traits]]) — it constrains `T` to only types that support comparison (`>`), which is exactly what the function body needs. Without this bound, the compiler would reject `item > largest` since arbitrary `T` might not support ordering at all.

## 2. Generic structs and enums

```rust
struct Point<T> {
    x: T,
    y: T,
}

let integer_point = Point { x: 5, y: 10 };
let float_point = Point { x: 1.0, y: 4.0 };
```

```rust
struct Point2<T, U> {   // two independent type parameters
    x: T,
    y: U,
}
let mixed = Point2 { x: 5, y: 4.0 };   // fine — T and U can differ
```

The standard library's own `Option<T>` and `Result<T, E>` are ordinary generic enums (see [[Enums and Pattern Matching]] §5) — there's no special-case mechanism beyond what's available to any user-defined generic type.

## 3. Monomorphization: generics cost nothing at runtime

> [!note] How zero-cost generics actually work
> At compile time, Rust generates a **separate, fully concrete version** of a generic function/struct for every distinct set of types it's actually used with — this process is called **monomorphization**. `largest::<i32>` and `largest::<char>` become two separate, specialized functions in the compiled binary, each exactly as fast as if you'd hand-written it for that one type. The genericity exists only in the source code; by the time the program runs, there is no generic machinery left, no runtime type dispatch, no boxing — hence "zero-cost abstraction."

This is the same underlying mechanism as C++ templates, and it's why generics are the default choice for performance-sensitive polymorphism in Rust, in contrast to [[Trait Objects and Dynamic Dispatch|trait objects]] (`dyn Trait`), which trade this zero runtime cost for the ability to have a single, non-generic type handle multiple concrete implementations at runtime.

## 4. Generic methods with impl

```rust
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

impl Point<f64> {   // methods can also be added for ONE specific concrete type
    fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

`distance_from_origin` only exists on `Point<f64>`, not `Point<i32>` or any other instantiation — a useful pattern for methods that only make sense for a specific concrete type.

## 5. Trait bounds in depth

```rust
fn notify<T: Summary + Display>(item: &T) { ... }   // T must implement BOTH traits

fn notify2(item: &(impl Summary + Display)) { ... }  // equivalent, "impl Trait" sugar

fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,   // `where` clause: clearer for many/complex bounds
    U: Clone + Debug,
{
    42
}
```

## See also
- [[Traits]]
- [[Trait Objects and Dynamic Dispatch]]
- [[Structs]]

#rust #generics

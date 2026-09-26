---
tags: [rust, traits]
---

# Traits

> [!summary] Summary
> A trait defines shared behavior — a set of methods a type can implement — analogous to an interface in other languages, but with default method implementations, and used pervasively for both generic bounds and dynamic polymorphism (see [[Trait Objects and Dynamic Dispatch]]). Rust has no class inheritance; traits are the mechanism for all polymorphism.

## 1. Defining and implementing a trait

```rust
trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {                        // default implementation
        format!("(Read more from {}...)", self.summarize_author())
    }
}

struct Tweet {
    username: String,
    content: String,
}

impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
    // summarize() not overridden — uses the default implementation
}

let tweet = Tweet { username: String::from("rustlang"), content: String::from("hi") };
println!("{}", tweet.summarize());   // uses the default method
```

Methods without a default implementation **must** be implemented by every type that implements the trait; methods with a default may be selectively overridden.

## 2. The orphan rule

> [!warning] You can only implement a trait for a type if you own the trait or the type
> Rust forbids implementing a **foreign trait** for a **foreign type** — e.g., you cannot implement the standard library's `Display` trait for the standard library's `Vec<T>` from your own crate. This "orphan rule" (part of Rust's **coherence** rules) exists to guarantee that any given `(Trait, Type)` pair has at most one implementation across the entire dependency graph — without it, two different crates could each define conflicting implementations of the same trait for the same type, and the compiler would have no principled way to decide which one to use.

## 3. Trait bounds and generics

Traits are how [[Generics|generic]] functions constrain what operations they can rely on:

```rust
fn notify(item: &impl Summary) {         // "impl Trait" syntax — accepts any Summary-implementing type
    println!("Breaking news! {}", item.summarize());
}
```

This is sugar for the fully generic form `fn notify<T: Summary>(item: &T)` — see [[Generics]] §5 for the full trait-bound syntax including `where` clauses and multiple bounds.

## 4. Common standard library traits

| Trait | Purpose |
|---|---|
| `Display` | user-facing formatting via `{}` |
| `Debug` | developer-facing formatting via `{:?}` (usually `#[derive]`d) |
| `Clone` | explicit deep duplication (`.clone()`) |
| `Copy` | implicit bitwise duplication instead of move (see [[Ownership]] §3) |
| `PartialEq`/`Eq` | equality comparison (`==`) |
| `PartialOrd`/`Ord` | ordering comparison (`<`, sorting) |
| `Iterator` | powers `for` loops and the entire iterator adapter ecosystem (see [[Iterators and Closures]]) |
| `Deref`/`DerefMut` | custom dereference behavior, what makes smart pointers transparent (see [[Deref and Drop Traits]]) |
| `Drop` | custom cleanup logic run when a value goes out of scope |
| `From`/`Into` | value-to-value conversions |
| `Send`/`Sync` | marker traits describing thread-safety properties (see [[Threads and Message Passing]]) |

## 5. Supertraits

A trait can require that implementors also implement another trait first:

```rust
trait OutlinePrint: std::fmt::Display {   // requires Display as a prerequisite
    fn outline_print(&self) {
        let output = self.to_string();     // can rely on Display's to_string() being available
        println!("*{}*", "-".repeat(output.len()));
        println!("|{output}|");
        println!("*{}*", "-".repeat(output.len()));
    }
}
```

## 6. Static dispatch vs dynamic dispatch — a preview

Trait bounds (`fn f<T: Trait>`, `impl Trait` parameters) resolve to a specific concrete type at **compile time** via monomorphization (see [[Generics]] §3) — this is **static dispatch**, zero runtime cost. When you need a single collection or return type that can hold *different* concrete types sharing a trait, you need **dynamic dispatch** via trait objects (`dyn Trait`) instead — covered fully in [[Trait Objects and Dynamic Dispatch]].

## See also
- [[Generics]]
- [[Trait Objects and Dynamic Dispatch]]
- [[Structs]]

#rust #traits

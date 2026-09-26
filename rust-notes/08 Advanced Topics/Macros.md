---
tags: [rust, macros, metaprogramming]
---

# Macros

> [!summary] Summary
> Macros generate code at compile time — Rust's metaprogramming facility. Unlike C's textual preprocessor macros, Rust macros operate on the language's syntax tree, making them far safer and more powerful, at the cost of being noticeably more involved to write.

## 1. Declarative macros — macro_rules!

```rust
macro_rules! square {
    ($x:expr) => {
        $x * $x
    };
}

let y = square!(5);   // expands to: 5 * 5
```

A pattern-matching system over Rust syntax fragments (`expr`, `ident`, `ty`, `block`, …), similar in spirit to `match` but operating on code structure rather than runtime values:

```rust
macro_rules! my_vec {
    () => { Vec::new() };
    ( $( $x:expr ),* ) => {
        {
            let mut v = Vec::new();
            $( v.push($x); )*
            v
        }
    };
}

let v: Vec<i32> = my_vec![1, 2, 3];   // expands the repetition $()* once per comma-separated item
```

This is (a simplified version of) how the standard library's own `vec!` macro is actually implemented — macros are how a variadic-looking, ergonomic literal syntax gets built on top of an otherwise fixed-arity function-call language.

## 2. Why macros instead of functions, sometimes

- **Variable number of arguments**: `println!("{} {} {}", a, b, c)` accepts any number of format arguments — impossible for an ordinary Rust function, which always has a fixed arity.
- **Operating on syntax, not values**: a macro can generate a `struct` definition, an `impl` block, or field names — things a function, which only ever receives and returns *values*, cannot do.
- **Compile-time code generation**: macros run during compilation, so they can generate code tailored to information only available at compile time (like a struct's field names, via derive macros below).

## 3. Procedural macros — the derive macros you already use

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
struct Point { x: i32, y: i32 }
```

Every `#[derive(...)]` you've used (see [[Structs]] §4) is a **derive macro** — a procedural macro that receives the struct/enum's syntax tree as input and generates an `impl` block as output (e.g., `#[derive(Debug)]` generates a full `impl Debug for Point { ... }` mechanically from the field list). `serde`'s `Serialize`/`Deserialize` derives (see [[Essential Crates]]) are the most widely used real-world example — they generate an entire hand-written-quality serializer/deserializer implementation purely from a struct's field definitions.

### The three kinds of procedural macros
| Kind | Invocation | Purpose |
|---|---|---|
| Derive | `#[derive(MyTrait)]` | generate a trait implementation from a struct/enum definition |
| Attribute-like | `#[my_attribute]` | transform an item (function, struct, module) it's attached to — e.g., `#[tokio::main]` (see [[Async Await]]) rewrites `async fn main` into a runtime-bootstrapping regular `fn main` |
| Function-like | `my_macro!(...)` | look like a `macro_rules!` invocation but implemented with full procedural (Rust code manipulating syntax trees) power, e.g., `sqlx::query!` for compile-time-checked SQL |

Procedural macros are written as their own separate crate (`proc-macro = true` in `Cargo.toml`), operating on `TokenStream`s using crates like `syn` (parsing) and `quote` (code generation) — genuinely writing Rust code that manipulates other Rust code as data.

## 4. Macros vs generics/functions — when to reach for one

> [!tip] Default to functions/generics; reach for macros only when you must
> Macros are harder to read, harder to debug (errors can point into generated code rather than your source), and more complex to write correctly than functions. Reach for a macro only when you need something a function fundamentally cannot express — variadic arguments, generating repetitive boilerplate (like a trait impl per field), or operating on syntax itself. If a function or generic can do the job, prefer it.

## See also
- [[Structs]]
- [[Essential Crates]] — serde's derive macros
- [[Traits]]

#rust #macros

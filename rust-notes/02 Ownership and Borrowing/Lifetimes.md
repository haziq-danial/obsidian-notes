---
tags: [rust, lifetimes]
---

# Lifetimes

> [!summary] Summary
> A lifetime is the compiler's way of tracking **how long a reference remains valid**, ensuring no reference ever outlives the data it points to. Lifetime annotations don't change how long anything actually lives — they describe relationships between existing lifetimes so the borrow checker can verify them.

![[lifetime-diagram.svg]]

## 1. The problem lifetimes solve

Every reference has an implicit lifetime — the borrow checker already tracks this internally for simple cases (see [[References and Borrowing]] §3, the dangling reference example). Lifetime *annotations* become necessary specifically when the compiler can't infer the relationship on its own, most commonly: a function takes multiple references and returns one, and the compiler needs to know which input(s) the output's validity is tied to.

```rust
fn longest(x: &str, y: &str) -> &str {   // ← compile error: missing lifetime specifier
    if x.len() > y.len() { x } else { y }
}
```

The compiler can't tell whether the returned reference should be valid as long as `x`, as long as `y`, or something else — it refuses to guess.

## 2. Lifetime annotation syntax

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

This reads as: "for some lifetime `'a`, given two string references that both live at least as long as `'a`, return a string reference that also lives at least as long as `'a`." It's a **constraint on the caller**, not a mechanism that extends anything's actual lifetime — the annotation just makes an already-true relationship explicit enough for the compiler to check calls against it.

> [!note] Lifetime annotations describe, they don't control
> A common early misconception is that lifetime annotations change how long a value lives. They don't — they're purely descriptive, telling the compiler about relationships that already exist in the code so it can verify every call site respects them. The actual lifetime of any given value is still determined by its scope, exactly as in [[Ownership]].

## 3. The borrow checker's job with `longest`

```rust
fn main() {
    let s1 = String::from("long string is long");
    {
        let s2 = String::from("xyz");
        let result = longest(s1.as_str(), s2.as_str());
        println!("Longest: {result}");   // fine — s1 and s2 both still alive here
    }
    // println!("Longest: {result}");    // would be an error if result escaped the inner block
}
```

Because the signature says the return value's lifetime is tied to *both* inputs, the compiler correctly infers the returned reference is only valid as long as the **shorter** of the two borrows — using it after `s2` (the shorter-lived one) goes out of scope is rejected.

## 4. Lifetime elision — most code needs no annotations

The vast majority of Rust functions never need explicit lifetime annotations because the compiler applies a small set of **elision rules** automatically:

1. Each elided input reference gets its own distinct lifetime parameter.
2. If there's exactly one input lifetime, it's assigned to all elided output lifetimes.
3. If one of the inputs is `&self` or `&mut self` (a method), the output's lifetime is assigned from `self`'s lifetime.

```rust
fn first_word(s: &str) -> &str { ... }   // sugar for fn first_word<'a>(s: &'a str) -> &'a str
```

Explicit annotations are needed only when a signature doesn't fit these patterns — most commonly, multiple input references where the output could plausibly relate to more than one of them (as in `longest`).

## 5. The 'static lifetime

`'static` means a reference is valid for the **entire duration of the program** — the longest possible lifetime. String literals are `'static` (they're baked directly into the compiled binary):

```rust
let s: &'static str = "I live for the whole program";
```

> [!warning] 'static is not a shortcut to satisfy the borrow checker
> A common beginner mistake is reaching for `'static` to make a stubborn lifetime error go away. This is almost always wrong unless the data genuinely needs to live for the entire program — it typically just relocates the underlying design problem (often solved instead by restructuring ownership, or reaching for [[Box RC and Interior Mutability|Rc/Arc]] to share ownership properly rather than faking a borrow's duration).

## 6. Lifetimes in structs

A struct holding a reference must annotate that reference's lifetime, tying the struct's own validity to the data it borrows:

```rust
struct Excerpt<'a> {
    part: &'a str,
}

let novel = String::from("Call me Ishmael. Some years ago...");
let first_sentence = novel.split('.').next().unwrap();
let excerpt = Excerpt { part: first_sentence };   // excerpt cannot outlive `novel`
```

This is why many beginner-friendly Rust structs simply own their data (`String` instead of `&str`) rather than borrowing it — avoiding lifetime parameters entirely at the cost of an extra allocation/clone, a very common and often the *correct* early simplification while learning.

## See also
- [[References and Borrowing]]
- [[Slices]]
- [[Structs]]

#rust #lifetimes

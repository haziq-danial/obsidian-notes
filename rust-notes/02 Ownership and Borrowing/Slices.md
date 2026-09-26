---
tags: [rust, slices]
---

# Slices

> [!summary] Summary
> A slice is a reference to a contiguous sequence of elements within a collection, rather than the whole collection — it borrows a "view" into existing data without copying it or taking ownership.

## 1. String slices

```rust
let s = String::from("hello world");
let hello = &s[0..5];     // "hello" — a &str, borrowing part of s
let world = &s[6..11];    // "world"
let whole = &s[..];       // the entire string, still just a borrow
```

A **string slice** (`&str`) is a reference to a portion of a `String` (or any UTF-8 text) — it doesn't own the data, just points into it plus a length.

> [!warning] Slices must fall on valid UTF-8 character boundaries
> `String` is stored as UTF-8 bytes, and multi-byte characters can't be sliced in the middle without producing invalid UTF-8 — Rust panics at runtime if a slice range falls inside a multi-byte character rather than silently returning corrupted text.

## 2. Why slices make APIs safer

> [!example] The classic motivating problem
> ```rust
> fn first_word(s: &String) -> usize {   // returns an index — fragile!
>     match s.find(' ') {
>         Some(i) => i,
>         None => s.len(),
>     }
> }
>
> let mut s = String::from("hello world");
> let word_end = first_word(&s);   // word_end = 5
> s.clear();                        // s is now empty — word_end is now meaningless!
> // word_end still equals 5, but there's no "5th byte" of meaning left in s
> ```
> Returning a plain index has no connection to the string it came from — the index can silently become invalid after further mutation, with no compiler warning. A slice-returning version fixes this categorically:
> ```rust
> fn first_word(s: &str) -> &str {
>     match s.find(' ') {
>         Some(i) => &s[..i],
>         None => s,
>     }
> }
>
> let mut s = String::from("hello world");
> let word = first_word(&s);        // word borrows from s
> // s.clear();                     // ← compile error: cannot borrow s as mutable,
>                                     //   it's still immutably borrowed by `word`
> ```
> The borrow checker now catches the exact bug the index-based version allowed — `s.clear()` (which needs `&mut s`) can't happen while `word` (a `&s` borrow) is still in use. The API's safety is enforced by the type system itself, not by careful discipline.

## 3. `&str` vs `String`

| | `&str` | `String` |
|---|---|---|
| Ownership | Borrowed (a view into data owned elsewhere) | Owned |
| Mutability | Immutable (the slice itself never grows/shrinks) | Growable via `.push_str()`, etc. |
| Where it lives | Could point into a `String`, a string literal (`'static` lifetime), or elsewhere | Always heap-allocated |

> [!tip] Function parameter convention
> Prefer `&str` over `&String` for function parameters that only need to *read* string data — a `&str` parameter accepts both `&String` (via deref coercion) and string literals directly, making the function strictly more flexible for callers with no loss of capability.

## 4. Slices of other collections

The same idea generalizes beyond strings — `&[T]` is a slice of any array or [[Common Collections|Vec]]:

```rust
let a = [1, 2, 3, 4, 5];
let middle: &[i32] = &a[1..4];   // [2, 3, 4]

fn sum(numbers: &[i32]) -> i32 {   // accepts a slice from an array OR a Vec
    numbers.iter().sum()
}
```

Writing a function to accept `&[T]` rather than `&Vec<T>` or `&[T; N]` makes it maximally flexible — it works uniformly whether the caller has a `Vec`, a fixed-size array, or a slice of either.

## See also
- [[References and Borrowing]]
- [[Common Collections]]
- [[Lifetimes]] — every slice is itself a reference with an associated lifetime

#rust #slices

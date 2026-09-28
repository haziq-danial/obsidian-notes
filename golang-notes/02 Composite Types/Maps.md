---
tags: [go, maps]
---

# Maps

> [!summary] Summary
> Go's built-in map type is a hash table with reference-like semantics (like slices, copying a map copies the header, not the data), randomized iteration order by design, and a distinctive two-value read syntax for distinguishing "absent" from "present but zero."

## 1. Declaring and using maps

```go
m := make(map[string]int)
m["alice"] = 30
m["bob"] = 25

ages := map[string]int{"alice": 30, "bob": 25}   // map literal

delete(m, "bob")
```

## 2. The zero value trap and the comma-ok idiom

```go
count := m["carol"]   // "carol" doesn't exist — count is 0, the zero value for int, NOT an error
```

> [!warning] A missing key looks identical to a present key with the zero value
> `m["carol"]` returns `0` whether `"carol"` maps to `0` or doesn't exist in the map at all — there's no way to tell them apart from that expression alone. The **comma-ok idiom** resolves this ambiguity:
> ```go
> value, ok := m["carol"]
> if !ok {
>     fmt.Println("carol not found")
> }
> ```
> `ok` is `true` only if the key genuinely exists, regardless of what value it maps to — this pattern appears throughout Go for exactly this "did it actually happen, or did I just get a zero value" disambiguation (also used with [[Channels and Select|channel receives]] and [[Interfaces|type assertions]]).

## 3. A nil map is read-safe but write-unsafe

```go
var m map[string]int      // nil map, not initialized with make()
v := m["key"]              // fine — reading a nil map returns the zero value, no panic
m["key"] = 1               // PANIC: assignment to entry in nil map
```

A `nil` map behaves like an empty map for reads (`len()`, lookups, `range` all work fine) but **panics** on any write — always initialize with `make()` or a map literal before writing to it. This is a common source of confusing nil-pointer-adjacent panics for newcomers, since the zero value silently "looks fine" until the first write.

## 4. Maps are unordered — always

```go
for k, v := range m {
    fmt.Println(k, v)   // order is randomized EVERY run, deliberately
}
```

As covered in [[Control Flow]] §1, Go's runtime deliberately randomizes map iteration order to prevent code from silently depending on an implementation detail that was never a guarantee. To iterate in a specific order, extract and sort the keys explicitly:

```go
keys := make([]string, 0, len(m))
for k := range m {
    keys = append(keys, k)
}
sort.Strings(keys)
for _, k := range keys {
    fmt.Println(k, m[k])
}
```

## 5. Maps as sets

Go has no built-in set type — the idiomatic substitute is `map[T]struct{}` (or, more casually, `map[T]bool`):

```go
set := make(map[string]struct{})
set["a"] = struct{}{}
_, exists := set["a"]   // true
```

`struct{}` (the empty struct) occupies **zero bytes**, making `map[T]struct{}` the most memory-efficient way to express "a set of T" — using `map[T]bool` works too and is more common in casual code, at the cost of one wasted byte per entry.

## 6. Maps and concurrency

> [!warning] Maps are not safe for concurrent use
> Concurrent reads are fine, but a concurrent read/write or write/write on a plain map is a **data race** — Go's runtime will actively detect and crash on many such cases (`fatal error: concurrent map read and map write`) rather than silently corrupting data, but it's still a bug to fix, not something to rely on catching. Use a [[The sync Package|sync.Mutex]] to guard shared map access, or reach for `sync.Map` (a specialized concurrent map, better suited for specific access patterns like write-once/read-many, not a general drop-in replacement).

## 7. Keys must be comparable

Map keys must be a type supporting `==` (numbers, strings, bools, pointers, interfaces, arrays, and structs made only of comparable fields) — slices, maps, and functions cannot be used as map keys directly, since they're not comparable, only nil-comparable.

## See also
- [[Arrays and Slices]]
- [[The sync Package]]
- [[Structs]]

#go #maps

---
tags: [go, slices, arrays]
---

# Arrays and Slices

> [!summary] Summary
> Arrays are fixed-size and rarely used directly in idiomatic Go; slices — a small header (pointer, length, capacity) over an underlying array — are the workhorse sequence type, and understanding that header is essential to avoiding a whole class of surprising aliasing bugs.

![[slice-internals.svg]]

## 1. Arrays: fixed size, part of the type

```go
var a [5]int                  // [5]int, all zeros
b := [3]string{"a", "b", "c"}
c := [...]int{1, 2, 3}         // length inferred from the literal: [3]int
```

An array's length is part of its **type** — `[5]int` and `[10]int` are different, incompatible types. Arrays are value types: assigning or passing one **copies the entire array**, unlike slices. This is precisely why arrays are rarely used directly — slices give reference-like, resizable behavior with far less copying overhead.

## 2. Slices: the header over an array

```go
s := []int{1, 2, 3}        // slice literal — Go allocates a backing array automatically
s2 := make([]int, 3, 5)     // len=3, cap=5, explicit backing array
```

A slice value is a **three-word header**: a pointer to the first accessible element, a length, and a capacity. Copying a slice copies the header (cheap — three words), not the underlying array — both the original and the copy point at the *same* backing array.

```go
a := []int{1, 2, 3}
b := a          // b shares a's backing array
b[0] = 99
fmt.Println(a)  // [99 2 3] — a is affected too!
```

## 3. Re-slicing and shared backing arrays

```go
s := []int{0, 1, 2, 3, 4}
mid := s[1:3]     // [1 2], len=2, cap=4 (capacity extends to the end of s's backing array)
mid[0] = 100
fmt.Println(s)     // [0 100 2 3 4] — mid and s share memory
```

> [!warning] Slicing doesn't copy — full-slice expressions and copy() are the escape hatches
> `s[low:high]` never allocates a new backing array; it just computes a new header pointing into the same memory. To guarantee independence, either use `copy(dst, src)` to make an explicit new array, or the three-index form `s[low:high:max]` to cap the shared capacity (preventing an `append` on the sub-slice from silently overwriting elements the original slice still considers "beyond its length" but within its capacity).

## 4. append() and reallocation

```go
s := make([]int, 3, 5)
s = append(s, 9)         // len 4, still ≤ cap 5 → SAME backing array
s = append(s, 1, 2, 3)   // len would be 7 > cap 5 → NEW backing array, contents copied over
```

When `append` needs more room than the current capacity, Go allocates a new, larger backing array (growth is roughly by doubling for smaller slices, tapering to ~1.25x for larger ones) and copies every existing element over — the original backing array is left untouched, and any other slice still pointing at it no longer aliases the appended result.

> [!example] The classic aliasing surprise
> ```go
> original := make([]int, 3, 5)
> alias := original[:2]       // len=2, cap=5, SAME backing array as original
> alias = append(alias, 99)    // len 3 ≤ cap 5 → writes into original's backing array!
> fmt.Println(original)         // original[2] is now 99, even though we never touched `original` directly
> ```
> This is exactly why passing sub-slices around and appending to them without care is a classic Go gotcha — always be certain whether a slice's remaining capacity is safely "owned" before appending.

## 5. Multi-dimensional slices

```go
grid := make([][]int, rows)
for i := range grid {
    grid[i] = make([]int, cols)
}
```

Go has no true multi-dimensional array/slice type — a "2D slice" is a slice of slices, each row independently allocated. This means rows aren't necessarily contiguous in memory (unlike a true 2D array in C), a performance consideration for numeric code.

## 6. Common slice operations

| Operation | Idiom |
|---|---|
| Append | `s = append(s, x)` |
| Remove element at i | `s = append(s[:i], s[i+1:]...)` |
| Copy | `dst := make([]T, len(src)); copy(dst, src)` |
| Check emptiness | `len(s) == 0` (works correctly even if `s` is `nil`) |
| Full copy of a slice | `s2 := append([]T(nil), s...)` |

## See also
- [[Maps]]
- [[Pointers]]
- [[Escape Analysis]]

#go #slices #arrays

---
tags: [go, pointers]
---

# Pointers

> [!summary] Summary
> Go has pointers for direct memory addressing, but none of C's pointer arithmetic — a Go pointer can only point at a single value (or be nil), which the compiler tracks and the garbage collector keeps valid for as long as needed.

## 1. Basic syntax

```go
x := 5
p := &x           // p is a *int, holding x's address
fmt.Println(*p)    // 5 — dereference to read the pointed-to value
*p = 10             // dereference to write through the pointer
fmt.Println(x)      // 10 — x itself changed
```

`&` takes a value's address; `*T` is "pointer to T"; `*p` dereferences a pointer to get/set the value it points to.

## 2. Why pointers matter: avoiding copies and enabling mutation

```go
type BigStruct struct {
    Data [1000]int
}

func processCopy(b BigStruct) { }     // copies all 1000 ints — expensive
func processPtr(b *BigStruct) { }      // copies one pointer (8 bytes) — cheap
```

Two independent reasons to use a pointer argument:
1. **Avoid copying** a large value on every call (a performance concern).
2. **Allow the function to mutate the caller's original** value rather than a throwaway copy (a correctness/intent concern — see [[Structs]] §2 and [[Methods]] §2).

These are separate concerns: a small struct might still use a pointer receiver purely for mutation, while a large read-only struct might be passed by pointer purely to avoid the copy cost.

## 3. No pointer arithmetic

```c
// C: legal, and a classic bug source
int *p = arr;
p = p + 1;   // now points at arr[1]
```

```go
// Go: no equivalent — this doesn't compile
p := &arr[0]
p = p + 1   // compile error: invalid operation
```

Go pointers cannot be incremented, cast to arbitrary addresses, or used to walk off the end of an allocation — this rules out an entire category of memory-corruption bugs (buffer overruns via pointer manipulation) that plague C. `unsafe.Pointer` exists as a deliberately-named, clearly-marked escape hatch for the rare cases (cgo interop, certain low-level optimizations) that genuinely need to bypass this.

## 4. nil pointers and panics

```go
var p *int          // nil, the zero value for any pointer type
fmt.Println(*p)      // PANIC: runtime error: invalid memory address or nil pointer dereference
```

Dereferencing a `nil` pointer panics immediately and loudly (see [[Panic and Recover]]) rather than silently reading garbage memory or corrupting state, as an equivalent bug might in C. This is Go's general philosophy: fail fast and visibly rather than continue in a broken state.

## 5. Pointers to structs: automatic dereferencing

```go
type Point struct{ X, Y int }
p := &Point{X: 1, Y: 2}
fmt.Println(p.X)     // no need to write (*p).X — Go does this automatically
p.X = 10               // same automatic dereferencing for writes
```

Go automatically dereferences a pointer for field access and method calls, so `p.X` and `(*p).X` are equivalent — this is purely syntactic convenience, not a different underlying mechanism.

## 6. new() vs make()

| | `new(T)` | `make(T, ...)` |
|---|---|---|
| Returns | `*T`, a pointer to a zero-valued T | `T` itself, initialized and ready to use |
| Works on | any type | only slices, maps, and channels |
| Use case | rare in idiomatic Go — usually `&T{}` is preferred for structs | the standard way to initialize slices/maps/channels with sizing (`make([]int, 0, 10)`, `make(chan int, 5)`) |

`new(T)` allocates zeroed memory for `T` and returns a pointer to it — functionally similar to `&T{}` for structs, but `make` is fundamentally different: it returns an *initialized, usable* slice/map/channel value (not a pointer), because these types need internal setup (a backing array, hash table, or buffer) that a plain zeroed allocation wouldn't provide.

## 7. Pointers and escape analysis

Whether a pointed-to value ends up on the stack or the heap is decided by the compiler's [[Escape Analysis|escape analysis]], not by whether you used `&`/`*` — a pointer to a genuinely short-lived local value can still be stack-allocated if the compiler proves it never outlives its function.

## See also
- [[Structs]]
- [[Methods]]
- [[Escape Analysis]]

#go #pointers

---
tags: [go, generics]
---

# Generics

> [!summary] Summary
> Generics arrived in Go 1.18 (2022) — deliberately later than most statically-typed languages, after years of the Go team searching for a design that stayed simple and didn't compromise compile speed. They let functions and types be written once and work over multiple types, with compile-time type safety.

## 1. The problem generics solve

Before 1.18, writing a function that worked over multiple types meant either duplicating code per type, or using `any`/`interface{}` and losing compile-time type safety (plus paying a runtime type-assertion cost, see [[Interfaces]] §6):

```go
// Pre-generics: works, but no compile-time guarantee both slice elements are comparable,
// and a type assertion is needed to get anything useful back out.
func Max(a, b any) any {
    // ...how would you even compare two `any` values here safely?
}
```

## 2. Generic functions

```go
func Max[T cmp.Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

Max(3, 5)         // T inferred as int
Max(3.1, 2.9)      // T inferred as float64
Max("a", "b")       // T inferred as string
```

`[T cmp.Ordered]` declares a **type parameter** `T`, constrained to types supporting `<`/`>`/etc. (`cmp.Ordered`, from the standard library's `cmp` package). The compiler almost always infers `T` from the call's arguments, so most call sites look exactly like an ordinary function call — no explicit `Max[int](3, 5)` needed, though that explicit form is available when inference can't determine the type alone.

## 3. Constraints

```go
type Number interface {
    ~int | ~int64 | ~float64 | ~float32
}

func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums {
        total += n
    }
    return total
}
```

A **constraint** is an interface listing which types (or type sets, via `|`) a type parameter may be instantiated with. The `~` prefix means "this type OR any type whose underlying type is this" — `~int` matches both `int` and `type UserID int`, letting the generic function work with named types built on top of the listed ones, not just the exact listed types themselves.

### Standard constraints (from the `cmp` and `constraints`-style packages)

| Constraint | Allows |
|---|---|
| `any` | literally anything (alias for `interface{}`) |
| `comparable` | any type supporting `==`/`!=` — required for generic map keys |
| `cmp.Ordered` | any type supporting `<`, `<=`, `>`, `>=` (numbers and strings) |

## 4. Generic types

```go
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}
func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.items) == 0 {
        return zero, false
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item, true
}

var intStack Stack[int]
intStack.Push(1)
intStack.Push(2)
v, ok := intStack.Pop()   // v = 2, ok = true
```

Generic types let a single implementation (a stack, a linked list, a binary tree) work correctly and type-safely for any element type, instantiated as `Stack[int]`, `Stack[string]`, `Stack[User]`, etc. — each instantiation behaves like its own concrete type at compile time.

## 5. Generic functions over slices — the slices/maps standard packages

```go
import "slices"

nums := []int{3, 1, 4, 1, 5}
slices.Sort(nums)               // works for any cmp.Ordered element type
idx, found := slices.BinarySearch(nums, 4)
```

The standard library's `slices` and `maps` packages (Go 1.21+) are themselves generic, replacing a lot of hand-written per-type boilerplate (or reflection-based helpers) that used to be common for basic slice/map operations like sorting, searching, and comparing.

## 6. Generics vs interfaces — when to use which

> [!tip] Rule of thumb
> Use **generics** when you need the *same* concrete type preserved through an operation without losing it to an interface's erasure (a `Stack[int]` should give you back `int`s, not `any`s requiring a type assertion). Use **interfaces** when you need to handle genuinely *different* concrete types uniformly through a shared set of behaviors (see [[Interfaces]]) — the two solve related but distinct problems, and idiomatic Go reaches for the plainest tool (often neither, just a concrete type) before generics unless the code duplication or `any`-erasure genuinely justifies them.

## See also
- [[Interfaces]]
- [[Arrays and Slices]]
- [[Methods]]

#go #generics

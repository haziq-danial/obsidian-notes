---
tags: [go, interfaces]
---

# Interfaces

> [!summary] Summary
> A Go interface is a set of method signatures; any type implementing those methods satisfies the interface **automatically** — no explicit `implements` declaration, no inheritance relationship required. This structural typing is central to how Go achieves polymorphism without classes.

![[interface-internals.svg]]

## 1. Defining and satisfying an interface

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}

type ConsoleWriter struct{}
func (ConsoleWriter) Write(p []byte) (int, error) {
    fmt.Print(string(p))
    return len(p), nil
}

var w Writer = ConsoleWriter{}   // satisfies Writer automatically — no "implements" keyword anywhere
```

`ConsoleWriter` never mentions `Writer` at all — it satisfies the interface purely by having a matching method. This is **structural typing** (sometimes called "duck typing, but checked at compile time"): if it has the right methods, it fits, regardless of where or why it was defined.

> [!note] Why this matters for decoupling
> A package can define an interface describing exactly what it needs (`type Fetcher interface { Fetch(url string) ([]byte, error) }`) without the types that will eventually satisfy it needing to import that package or know it exists — this is why idiomatic Go interfaces are often small and defined at the point of *use* (the consumer), not alongside the concrete type that happens to satisfy them.

## 2. Small interfaces are idiomatic

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
type ReadWriter interface {   // composed from smaller interfaces
    Reader
    Writer
}
```

> [!tip] "The bigger the interface, the weaker the abstraction"
> This is a well-known Go proverb (Rob Pike). The standard library's `io.Reader` and `io.Writer` — each just **one method** — are held up as the canonical example: because they demand so little, an enormous number of types can satisfy them (files, network connections, in-memory buffers, compressors, HTTP bodies), making code written against `io.Reader` reusable in contexts its author never anticipated. A large, do-everything interface, by contrast, can only be satisfied by types purpose-built for it.

## 3. The empty interface and `any`

```go
func describe(v any) {   // any is an alias for interface{}, added in Go 1.18 for readability
    fmt.Printf("value: %v, type: %T\n", v, v)
}

describe(42)
describe("hello")
describe(true)
```

`any` (`interface{}`) has **zero methods**, so every type satisfies it — it's Go's way of accepting "a value of any type," used heavily before generics (see [[Generics]]) and still common for things like `fmt.Println(args ...any)` or JSON-decoding into a generic `map[string]any`.

## 4. Type assertions and type switches

```go
var w Writer = ConsoleWriter{}

cw, ok := w.(ConsoleWriter)   // "comma-ok" type assertion — ok is false, not a panic, if it fails
if ok {
    fmt.Println("it's a ConsoleWriter")
}

cw2 := w.(ConsoleWriter)       // single-value form — PANICS if the assertion fails
```

A type assertion recovers the concrete type from an interface value; the comma-ok form is the safe way to check without risking a panic (mirroring the [[Maps|comma-ok idiom]] for map lookups). For branching across several possible concrete types, a [[Control Flow|type switch]] (`switch v := x.(type)`) is the idiomatic alternative to a chain of assertions.

## 5. The nil interface gotcha

```go
func mayFail() error {
    var p *MyError = nil   // p is nil...
    return p                 // ...but this returns a NON-nil error interface!
}

err := mayFail()
fmt.Println(err == nil)   // false — surprising!
```

> [!warning] An interface holding a nil pointer is not itself nil
> An interface value is `nil` only if **both** its type descriptor and its data pointer are nil. Returning a `nil` `*MyError` through an `error`-typed return value gives the interface a non-nil type descriptor (`*MyError`) even though the underlying pointer is nil — so `err == nil` is `false`. The fix is to return a literal untyped `nil` directly (`return nil`) when there's no error, rather than a typed nil pointer variable, and to avoid declaring error variables as a concrete pointer type when they'll be returned as `error`.

## 6. Interfaces and performance

Calling a method through an interface value costs a small indirection (a lookup through the method table, see the diagram above) compared to calling a concrete method directly, which the compiler can often inline. This cost is rarely significant in practice and should not discourage using interfaces where they genuinely improve decoupling — but it's the underlying reason hot numeric loops sometimes avoid interface indirection in favor of concrete types or generics (see [[Generics]]).

## See also
- [[Methods]]
- [[Embedding]]
- [[Generics]]

#go #interfaces

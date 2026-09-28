---
tags: [go, methods]
---

# Methods

> [!summary] Summary
> A method is a function with a receiver argument, attaching behavior to a type without a class body. The choice between a value receiver and a pointer receiver is one of the most consequential decisions in everyday Go code — it determines whether the method can mutate its receiver and how it interacts with interfaces.

## 1. Defining methods

```go
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {          // value receiver
    return r.Width * r.Height
}

func (r *Rectangle) Scale(factor float64) {   // pointer receiver
    r.Width *= factor
    r.Height *= factor
}

rect := Rectangle{Width: 3, Height: 4}
rect.Area()          // 12
rect.Scale(2)         // Go automatically takes &rect since Scale has a pointer receiver
```

The receiver — `(r Rectangle)` or `(r *Rectangle)` — appears between `func` and the method name, distinguishing a method from an ordinary function. `rect.Area()` is syntactic sugar for calling `Area` with `rect` as its first argument; there's no other hidden mechanism.

## 2. Value receiver vs pointer receiver

| | Value receiver `(r Rectangle)` | Pointer receiver `(r *Rectangle)` |
|---|---|---|
| Receives | a **copy** of the value | the original, via its address |
| Can mutate the caller's value? | No | Yes |
| Cost for large structs | Copies the whole struct on every call | Cheap — copies one pointer |
| Works on a `nil` pointer? | N/A | Yes, if the method doesn't dereference `r` |

> [!tip] The consistency rule
> If **any** method on a type needs a pointer receiver (usually because it mutates the receiver), idiomatic Go makes **all** methods on that type use pointer receivers, even ones that only read — this keeps the type's method set consistent and avoids confusing situations where some methods see mutations and others silently don't (because they were working on a stale copy).

## 3. Automatic addressing and dereferencing

```go
rect := Rectangle{3, 4}
rect.Scale(2)        // Go automatically does (&rect).Scale(2)

p := &Rectangle{3, 4}
p.Area()              // Go automatically does (*p).Area()
```

Go automatically takes the address (for calling a pointer-receiver method on an addressable value) or dereferences (for calling a value-receiver method through a pointer) — this convenience is purely at the call site; it doesn't change which receiver type the method itself was declared with.

> [!warning] The one place this convenience doesn't apply: interfaces
> A value of type `Rectangle` only satisfies an [[Interfaces|interface]] requiring `Scale()` if `Scale` has a **value** receiver — if `Scale` has a pointer receiver, only `*Rectangle` satisfies that interface, not `Rectangle` itself. This is because the automatic addressing trick requires an addressable value at the call site, but storing a `Rectangle` in an interface variable erases that addressability. This is one of the most common early points of confusion in Go: "why won't my struct satisfy this interface?" is very often answered by "one of its methods has a pointer receiver, and you're passing a value, not a pointer."

## 4. Methods on non-struct types

```go
type Celsius float64

func (c Celsius) ToFahrenheit() float64 {
    return float64(c)*9/5 + 32
}

temp := Celsius(100)
temp.ToFahrenheit()   // 212
```

Methods can be defined on **any named type** declared in the same package, not just structs — a common pattern for adding validated behavior/formatting to a simple underlying type (like `Celsius` above, or a custom `type UserID int`) without exposing the raw underlying representation everywhere.

> [!note] You can't add methods to types you don't own
> You cannot define a method on a type declared in a different package (including built-ins like `int` or `string` directly) — only on a named type declared in your own package. The workaround is defining your own named type wrapping theirs (`type MyInt int`), which does let you attach methods, at the cost of needing explicit conversions to/from the underlying type.

## See also
- [[Interfaces]]
- [[Structs]]
- [[Pointers]]

#go #methods

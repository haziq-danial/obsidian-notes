---
tags: [go, embedding, composition]
---

# Embedding

> [!summary] Summary
> Go has no class inheritance. Instead, embedding a type inside a struct (or an interface inside another interface) promotes its fields and methods to the outer type — composition standing in for inheritance, with an important difference: no polymorphic dispatch.

## 1. Struct embedding

```go
type Base struct {
    ID int
}
func (b Base) Describe() string {
    return fmt.Sprintf("ID: %d", b.ID)
}

type User struct {
    Base        // embedded — no field name given, just the type
    Name string
}

u := User{Base: Base{ID: 1}, Name: "Alice"}
fmt.Println(u.ID)          // promoted field — same as u.Base.ID
fmt.Println(u.Describe())   // promoted method — same as u.Base.Describe()
```

The embedded field is accessed by its type name (`u.Base`) but its fields/methods are also **promoted** to be accessible directly on the outer struct (`u.ID`, `u.Describe()`) — as if they'd been declared directly on `User`, without actually duplicating them.

## 2. Embedding is composition, not inheritance

> [!warning] No polymorphic dispatch through embedding
> ```go
> func (b Base) Greet() string {
>     return "Hi, " + b.Describe()   // always calls Base's OWN Describe, never an "override"
> }
> ```
> If `User` defines its own `Describe()` method, calling `u.Greet()` **still** invokes `Base`'s `Describe()`, not `User`'s — because `Greet` is a method on `Base`, and `Base` has no idea `User` exists or embeds it. This is the crucial way embedding differs from inheritance in languages like Java/C++/Python: there is no virtual dispatch, no `super`, and no "is-a" relationship enforced by the type system. Each embedded type's methods operate purely on their own embedded value, with no visibility into whatever outer struct might be embedding them.

## 3. Method promotion and overriding by shadowing

```go
type User struct {
    Base
    Name string
}

func (u User) Describe() string {   // this "shadows" Base's Describe when called on a User value
    return fmt.Sprintf("%s (ID: %d)", u.Name, u.ID)
}

u.Describe()        // calls User's own Describe
u.Base.Describe()    // explicitly reach the embedded one instead
```

Defining a method with the same name directly on the outer type doesn't override in the OOP sense — it simply means the outer type's own method takes precedence for direct calls on the outer type, while the embedded type's identically-named method remains reachable explicitly via the field name. Both coexist; nothing is replaced.

## 4. Interface embedding

```go
type Reader interface {
    Read(p []byte) (int, error)
}
type Closer interface {
    Close() error
}
type ReadCloser interface {   // composed from smaller interfaces, no methods of its own
    Reader
    Closer
}
```

Embedding interfaces inside another interface is purely additive — `ReadCloser` requires exactly the union of `Reader`'s and `Closer`'s methods. This is the standard way the standard library builds up richer interfaces (`io.ReadCloser`, `io.ReadWriteCloser`) from small, single-purpose ones (see [[Interfaces]] §2).

## 5. Embedding for "mixins" — a common real-world use

```go
type Logger struct{}
func (Logger) Log(msg string) { fmt.Println("[LOG]", msg) }

type Service struct {
    Logger              // every Service gets a Log method "for free"
    Name string
}

svc := Service{Name: "billing"}
svc.Log("started")   // [LOG] started
```

Embedding a small, reusable type purely to grant its methods to many other types (without needing a shared base class or explicit delegation boilerplate) is a common, idiomatic use — effectively a lightweight mixin, though still without any dynamic dispatch back into the embedding type.

## 6. Ambiguous promotion

If two embedded types both have a method/field of the same name, that name is **not promoted** at all — you must disambiguate explicitly (`u.TypeA.Name` vs `u.TypeB.Name`), and referencing the ambiguous name directly on the outer struct is a compile error.

## See also
- [[Structs]]
- [[Interfaces]]
- [[Methods]]

#go #embedding

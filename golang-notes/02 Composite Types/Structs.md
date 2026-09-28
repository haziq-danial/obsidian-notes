---
tags: [go, structs]
---

# Structs

> [!summary] Summary
> A struct groups named fields into one type — Go's mechanism for defining data shapes, with methods attached separately (see [[Methods]]) rather than baked into a class body, and no inheritance, only composition via [[Embedding]].

## 1. Defining and creating structs

```go
type User struct {
    Name  string
    Email string
    Age   int
}

u1 := User{Name: "Alice", Email: "alice@example.com", Age: 30}   // field names, any order
u2 := User{"Bob", "bob@example.com", 25}                          // positional, order-dependent (fragile)
u3 := User{Name: "Carol"}                                          // other fields get zero values
var u4 User                                                        // all fields zero-valued
```

> [!tip] Prefer named fields in struct literals
> `User{"Bob", "bob@example.com", 25}` breaks silently if the struct's field order ever changes — named fields (`User{Name: "Bob", ...}`) are immune to reordering and self-documenting at every call site. Idiomatic Go strongly favors named struct literals except for very small, stable, well-known types.

## 2. Structs are value types

```go
func birthday(u User) {
    u.Age++   // modifies a COPY — has no effect on the caller's original
}

func birthdayPtr(u *User) {
    u.Age++   // modifies the original, through a pointer
}

birthday(u1)      // u1.Age unchanged
birthdayPtr(&u1)   // u1.Age incremented
```

Like arrays (and unlike slices/maps), assigning or passing a struct **copies every field**. This is why methods that need to mutate their receiver use a pointer receiver (see [[Methods]] §2) — passing/receiving the struct by value would only ever modify a throwaway copy.

## 3. Anonymous structs

```go
point := struct {
    X, Y int
}{X: 1, Y: 2}
```

An anonymous struct type is useful for throwaway, one-off groupings — common in table-driven tests (see [[Testing in Go]]) where each test case is a small ad-hoc struct that never needs a named type of its own.

## 4. Struct tags

```go
type User struct {
    Name  string `json:"name"`
    Email string `json:"email,omitempty"`
    age   int    `json:"-"`   // unexported field: json package can't access it at all, tag is moot
}
```

**Struct tags** are string metadata attached to fields, read via reflection by libraries like `encoding/json` and `encoding/xml` to control serialization behavior (renaming a field, omitting it when empty, excluding it entirely). Tags are plain string literals with no special compiler support — their meaning is entirely up to whatever library chooses to interpret them via `reflect.StructTag`.

## 5. Comparing structs

```go
type Point struct{ X, Y int }
p1 := Point{1, 2}
p2 := Point{1, 2}
fmt.Println(p1 == p2)   // true — structs are comparable if all their fields are
```

Structs support `==`/`!=` automatically as long as every field is itself comparable (no slices, maps, or functions as fields) — this is what makes simple structs usable directly as [[Maps|map keys]] without any extra work.

## 6. Embedding — Go's alternative to inheritance

```go
type Animal struct {
    Name string
}
func (a Animal) Describe() string { return "I am " + a.Name }

type Dog struct {
    Animal      // embedded field — no field name, just the type
    Breed string
}

d := Dog{Animal: Animal{Name: "Rex"}, Breed: "Labrador"}
fmt.Println(d.Name)        // promoted field — accessible directly
fmt.Println(d.Describe())   // promoted method — accessible directly
```

See [[Embedding]] for the full treatment — struct embedding "promotes" the embedded type's fields and methods to the outer struct, giving composition-based code reuse without true inheritance (no polymorphic dispatch, no `super`, no is-a relationship enforced by the type system).

## See also
- [[Methods]]
- [[Embedding]]
- [[Pointers]]

#go #structs

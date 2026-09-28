---
tags: [go, error-handling, errors]
---

# Errors as Values

> [!summary] Summary
> Go has no exceptions for ordinary error handling. An `error` is just an interface value returned alongside a function's result, checked explicitly with `if err != nil` — a deliberate design choice that makes failure paths visible in the source rather than implicit and easy to miss.

## 1. The error interface

```go
type error interface {
    Error() string
}
```

That's the **entire** definition — any type with an `Error() string` method satisfies `error`. This is why custom error types are trivial to define (see [[Error Wrapping]]) and why the standard library's error handling has no special-cased machinery beyond ordinary interface satisfaction.

## 2. The basic idiom

```go
func readConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, err
    }
    var cfg Config
    if err := json.Unmarshal(data, &cfg); err != nil {
        return nil, err
    }
    return &cfg, nil
}

cfg, err := readConfig("app.toml")
if err != nil {
    log.Fatal(err)
}
```

Every fallible operation returns `(result, error)`, and the caller is expected to check `err` **immediately**, before doing anything with `result` — this is why Go code has a distinctive, repetitive-looking `if err != nil { return ... }` rhythm throughout; it's the direct visual cost of making every failure path explicit rather than implicit.

## 3. Creating errors

```go
errors.New("something went wrong")           // a simple, static error
fmt.Errorf("failed to process user %d: %w", id, err)  // formatted, and wraps another error (see Error Wrapping)
```

`errors.New` creates a basic error from a string; `fmt.Errorf` with `%w` does the same but also **wraps** an underlying error, preserving it for later inspection (see [[Error Wrapping]]).

## 4. Sentinel errors

```go
var ErrNotFound = errors.New("not found")

func find(id int) (*Item, error) {
    if !exists(id) {
        return nil, ErrNotFound
    }
    // ...
}

item, err := find(42)
if errors.Is(err, ErrNotFound) {
    // handle specifically
}
```

A **sentinel error** is a package-level `error` value, exported so callers can compare against it directly. `errors.Is` (rather than `==`) is the idiomatic comparison because it also correctly unwraps a chain of wrapped errors (see [[Error Wrapping]]) to check if the sentinel appears anywhere in it — `==` would only match an exact, unwrapped identity.

## 5. Custom error types

```go
type ValidationError struct {
    Field string
    Msg   string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Msg)
}

func validate(age int) error {
    if age < 0 {
        return &ValidationError{Field: "age", Msg: "must be non-negative"}
    }
    return nil
}

var verr *ValidationError
if errors.As(err, &verr) {
    fmt.Println("bad field:", verr.Field)   // access fields beyond just the message
}
```

A custom error type carries structured data (here, which field failed and why) beyond a plain string message — `errors.As` retrieves it back out of a possibly-wrapped error chain, matching by concrete type rather than by identity.

## 6. Why not exceptions?

> [!note] The Go team's stated reasoning
> Go's designers argue that exceptions encourage control flow that's hard to trace (a `throw` deep in a call stack can be caught arbitrarily far away, with the intervening code often unaware an exception could even occur) and that explicit error returns force programmers to consciously decide, at every single call site, what happens on failure. The visible cost is repetitive `if err != nil` blocks; the argued benefit is that failure handling is never accidentally skipped or silently swallowed by a distant, unrelated `catch`.

Go does have [[Panic and Recover|panic/recover]], but it's reserved for genuinely unrecoverable situations (programming bugs, corrupted invariants) — not the general error-handling mechanism.

## See also
- [[Error Wrapping]]
- [[Panic and Recover]]
- [[Functions]]

#go #error-handling

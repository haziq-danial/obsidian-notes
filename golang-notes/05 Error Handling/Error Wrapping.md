---
tags: [go, error-handling, wrapping]
---

# Error Wrapping

> [!summary] Summary
> As errors propagate up a call stack, each layer often wants to add context ("while doing X, this failed") without losing the original underlying error. Go 1.13's `%w` verb and the `errors.Is`/`errors.As`/`errors.Unwrap` functions formalize this into a proper, inspectable chain.

![[error-wrapping-chain.svg]]

## 1. Wrapping with %w

```go
func getUser(id int) (*User, error) {
    row := db.QueryRow("SELECT * FROM users WHERE id = ?", id)
    var u User
    if err := row.Scan(&u.Name, &u.Email); err != nil {
        return nil, fmt.Errorf("get user %d: %w", id, err)   // wraps err, preserving it
    }
    return &u, nil
}
```

`%w` (used only with `fmt.Errorf`) embeds the original error inside the new one, alongside a formatted message — the resulting error's `.Error()` string includes both, but critically, the original error remains programmatically reachable via `Unwrap()`, not just baked irretrievably into a string.

> [!warning] %v or string concatenation breaks the chain
> `fmt.Errorf("get user %d: %v", id, err)` (using `%v` instead of `%w`) produces a similarly-looking message but does **not** implement `Unwrap()` — `errors.Is`/`errors.As` can no longer see past that point. Always use `%w` specifically when the intent is to preserve the underlying error for later inspection, not just to mention it in a message.

## 2. errors.Is — checking for a specific error anywhere in the chain

```go
_, err := getUser(42)
if errors.Is(err, sql.ErrNoRows) {
    // this matches even though err's actual message is "get user 42: sql: no rows in result set"
    fmt.Println("no such user")
}
```

`errors.Is` walks the entire wrap chain (calling `Unwrap()` repeatedly) checking each link against the target — this is why it succeeds even when the sentinel error (see [[Errors as Values]] §4) is buried several layers of wrapping deep, unlike a plain `==` comparison which would only match the outermost error exactly.

## 3. errors.As — extracting a specific error type anywhere in the chain

```go
var validationErr *ValidationError
if errors.As(err, &validationErr) {
    fmt.Println("field:", validationErr.Field)
}
```

`errors.As` walks the same chain but matches by **concrete type** rather than identity, and — on a match — assigns the found error into the target pointer, giving access to any extra fields/methods that type carries beyond the plain `error` interface (see [[Errors as Values]] §5).

## 4. errors.Unwrap — the mechanism underneath

```go
type wrapError struct {
    msg string
    err error
}
func (e *wrapError) Error() string { return e.msg }
func (e *wrapError) Unwrap() error  { return e.err }   // this is what fmt.Errorf's %w generates under the hood
```

Any error type can participate in the chain simply by implementing `Unwrap() error` — this is exactly what `fmt.Errorf`'s `%w` support generates internally, and it's why custom error types can also be made "unwrappable" by adding this one method themselves.

## 5. Multiple wrapped errors (Go 1.20+)

```go
err := errors.Join(err1, err2, err3)   // combines several errors into one
errors.Is(err, err1)   // true — Is/As check ALL joined errors, not just a single linear chain
```

`%w` can also appear **multiple times** in one `fmt.Errorf` call (Go 1.20+), and `errors.Join` combines several independent errors into one that `errors.Is`/`errors.As` will search through — useful for reporting several unrelated failures (e.g., from concurrent goroutines, or multiple independent validation failures) as a single returned error without discarding any of them.

## 6. Guidance on how much context to add

> [!tip] Add context that's useful to the reader of the eventual error message
> A well-wrapped error chain reads like a breadcrumb trail: `"handle request: get user 42: sql: no rows in result set"` tells you exactly what was being attempted at each layer. Avoid wrapping with redundant or unhelpful context (`"error: %w"` adds nothing) — the goal is that whoever eventually reads or logs the final error can understand what actually went wrong and where, without needing to trace back through source code.

## See also
- [[Errors as Values]]
- [[Panic and Recover]]
- [[Testing in Go]]

#go #error-handling #wrapping

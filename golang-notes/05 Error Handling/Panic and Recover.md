---
tags: [go, panic, recover]
---

# Panic and Recover

> [!summary] Summary
> `panic` and `recover` are Go's mechanism for truly unrecoverable situations — not a general try/catch replacement. A panic unwinds the stack, running deferred calls along the way, until either the program crashes or a deferred `recover()` stops it.

![[defer-panic-recover.svg]]

## 1. panic — stopping normal execution

```go
func divide(a, b int) int {
    if b == 0 {
        panic("division by zero")
    }
    return a / b
}
```

A `panic` immediately stops normal execution of the current function. Control doesn't return to the caller in the usual way — instead, the runtime begins **unwinding the stack**, running any [[Functions|deferred]] calls in each frame it passes through, in LIFO order, before either reaching a `recover()` or crashing the program with a stack trace.

Panics also happen automatically at runtime for certain unrecoverable errors: nil pointer dereference (see [[Pointers]] §4), out-of-bounds slice/array indexing, division by zero (for integers), a failed type assertion (non-comma-ok form, see [[Interfaces]] §4), sending on a closed channel, and more.

## 2. defer runs regardless of how a function returns

```go
func process() {
    defer fmt.Println("cleanup")   // runs even if process() panics
    panic("something broke")
}
```

Every deferred call scheduled before a panic still runs as the stack unwinds — this is why `defer conn.Close()` or `defer mu.Unlock()` reliably clean up resources even in the presence of a panic, without needing a separate `finally`-equivalent construct.

## 3. recover — stopping the unwind

```go
func safeDivide(a, b int) (result int, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("recovered from panic: %v", r)
        }
    }()
    return divide(a, b), nil
}
```

`recover()` stops an in-progress panic **only** when called directly inside a deferred function. If it's called and a panic is currently unwinding through that point, `recover()` returns the value passed to `panic(...)` and normal execution resumes — the enclosing function then returns normally to *its* caller (using whatever the named return values were set to inside the deferred function, as shown converting the panic into a plain `error` above).

> [!warning] recover() only works in a directly-deferred function
> ```go
> func doesNotWork() {
>     if r := recover(); r != nil { }   // does nothing — no panic is in flight when called directly like this
> }
> func alsoDoesNotWork() {
>     defer helper()   // helper() calling recover() internally does NOT catch a panic here either
> }
> func helper() {
>     recover()   // too indirect — must be called directly inside the deferred func, not by something it calls
> }
> ```
> `recover()` is only effective when invoked **directly** by a function that is itself directly deferred (typically an anonymous `defer func() { recover() }()`) — any extra layer of indirection makes it a no-op returning `nil`.

## 4. Panics don't cross goroutine boundaries

> [!warning] An unrecovered panic in any goroutine crashes the entire program
> A panic that's never recovered doesn't just kill the one goroutine it occurred in — it takes down the **whole process**, including every other goroutine, printing a stack trace and exiting with a non-zero status. `recover()` in one goroutine cannot catch a panic happening in a different goroutine; each goroutine that might panic and should survive it needs its own `defer`/`recover` pair (a common pattern in servers: recovering per-request-handling-goroutine so one bad request doesn't crash the whole server).

## 5. When panic is appropriate

> [!tip] Reserve panic for programmer errors and unrecoverable states
> Idiomatic Go uses `panic` sparingly: for situations that indicate a bug (an invariant that should be structurally impossible to violate if the code is correct), or truly unrecoverable startup failures (`main()` failing to load essential configuration might reasonably panic or `log.Fatal`, since there's no sensible way to continue). For anything an ordinary caller might reasonably want to handle — a missing file, a malformed request, a network timeout — return an [[Errors as Values|error]] instead. A well-known Go proverb: "don't panic" is the general rule, with the exception carved out narrowly.

Some standard library functions do panic on clearly-invalid input as a documented contract (e.g., `regexp.MustCompile` panics on an invalid pattern) — the `Must`-prefixed naming convention signals this explicitly, distinguishing them from their `error`-returning counterparts (`regexp.Compile`).

## See also
- [[Errors as Values]]
- [[Error Wrapping]]
- [[Functions]]

#go #panic #recover

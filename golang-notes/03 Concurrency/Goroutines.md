---
tags: [go, concurrency, goroutines]
---

# Goroutines

> [!summary] Summary
> A goroutine is a function running concurrently with other goroutines, managed by the Go runtime rather than the OS — cheap enough (starting at ~2KB of stack) to spawn by the thousands or millions, unlike OS threads.

## 1. Starting a goroutine

```go
func sayHello() {
    fmt.Println("hello")
}

go sayHello()          // runs concurrently — doesn't block the caller
go func() {              // anonymous function, launched the same way
    fmt.Println("hi from a closure")
}()
```

The `go` keyword is the entire syntax for concurrency at the call site — there's no thread pool to configure, no explicit `Thread` object to construct and start.

## 2. Goroutines vs OS threads

| | OS thread | Goroutine |
|---|---|---|
| Initial stack size | Fixed, often 1–8MB | ~2KB, grows/shrinks dynamically as needed |
| Creation cost | Relatively expensive (a syscall) | Extremely cheap (a runtime bookkeeping operation) |
| Typical max count | Thousands, before exhausting memory/scheduler overhead | Millions, practically |
| Scheduled by | The OS kernel | The Go runtime's own scheduler (see [[The Go Scheduler]]) |

This is why "spawn a goroutine per incoming request" is standard, idiomatic Go for a server, whereas "spawn an OS thread per request" would be a scalability concern in most other languages.

## 3. The main goroutine and program exit

```go
func main() {
    go doWork()      // this goroutine may never get to run!
}   // main() returns immediately — the whole program exits, doWork() or not
```

> [!warning] The program doesn't wait for goroutines automatically
> When `main()` returns, the entire program exits immediately, taking every still-running goroutine down with it — there's no automatic "wait for all goroutines to finish" behavior. Coordinating this is the programmer's job, typically via a `sync.WaitGroup` (see [[The sync Package]]) or by receiving a completion signal over a channel (see [[Channels and Select]]).

## 4. Closures capturing loop variables — a classic pre-Go-1.22 gotcha

```go
// Go 1.21 and earlier: BUGGY
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i)   // likely prints 3, 3, 3 — all goroutines shared ONE `i`
    }()
}

// The classic fix, still valid and common:
for i := 0; i < 3; i++ {
    i := i               // shadow: each iteration gets its own copy
    go func() {
        fmt.Println(i)   // now prints 0, 1, 2 in some order
    }()
}
```

> [!note] Fixed by language change in Go 1.22
> Go 1.22 (2024) changed `for` loop semantics so that each iteration gets its **own** fresh copy of the loop variable automatically — the shadowing workaround above is no longer necessary in code targeting Go 1.22+, though it remains common in older codebases and is harmless to keep. This was one of the rare cases where the Go team accepted a subtle behavior change specifically because the old semantics were such a persistent, well-known footgun.

## 5. Goroutine leaks

> [!warning] A goroutine that blocks forever never gets garbage collected
> Unlike memory, the Go garbage collector does **not** clean up goroutines — a goroutine blocked forever on a channel receive that will never happen, or stuck in an infinite loop, leaks for the lifetime of the program. Every goroutine you start should have a clear story for how and when it terminates — common causes of leaks include forgetting to close a channel a goroutine is ranging over, or a goroutine sending to a channel nobody will ever receive from.

## 6. Common patterns at a glance

- **Fire-and-forget with synchronization**: `sync.WaitGroup` to know when a batch of goroutines finishes (see [[The sync Package]]).
- **Worker pools**: a fixed number of goroutines pulling work off a shared channel (see [[Concurrency Patterns]]).
- **Pipelines**: goroutines connected by channels, each stage transforming data and passing it to the next (see [[Concurrency Patterns]]).
- **Cancellation**: `context.Context` propagated through goroutines to signal "stop now" (see [[Concurrency Patterns]]).

## See also
- [[Channels and Select]]
- [[The sync Package]]
- [[The Go Scheduler]]

#go #concurrency #goroutines

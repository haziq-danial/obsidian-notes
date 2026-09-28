---
tags: [go, concurrency, channels]
---

# Channels and Select

> [!summary] Summary
> Channels are typed conduits for sending values between goroutines, embodying Go's concurrency motto: "Do not communicate by sharing memory; instead, share memory by communicating." `select` lets a goroutine wait on multiple channel operations at once.

![[channels-select.svg]]

## 1. Creating and using channels

```go
ch := make(chan int)         // unbuffered
ch := make(chan int, 5)       // buffered, capacity 5

ch <- 42          // send
v := <-ch          // receive
v, ok := <-ch       // ok is false if the channel is closed and drained
close(ch)           // signal no more values will be sent
```

## 2. Unbuffered vs buffered channels

| | Unbuffered (`make(chan T)`) | Buffered (`make(chan T, n)`) |
|---|---|---|
| Send blocks until | a receiver is ready **at that instant** | the buffer has free space (or forever if `n=0` is treated as unbuffered) |
| Receive blocks until | a sender is ready | the buffer has at least one value |
| Synchronization | strong — send and receive are a **rendezvous**, guaranteeing the sender's prior work "happened before" the receiver sees it | weaker — sender and receiver can run further ahead/behind each other, up to the buffer size |

> [!note] Unbuffered channels are a synchronization primitive, not just a queue
> Because an unbuffered send only completes once a receiver is actively receiving, a successful unbuffered send/receive pair is a full synchronization point (formally guaranteed by the Go memory model) — everything the sender did before sending is guaranteed visible to the receiver after receiving. This is why "just use an unbuffered channel" is a common, simple way to coordinate two goroutines without a separate mutex.

## 3. Closing channels

```go
close(ch)
v, ok := <-ch   // after close, still returns any buffered values; once drained, ok=false, v=zero value

for v := range ch {   // range over a channel automatically exits when the channel is closed and drained
    fmt.Println(v)
}
```

> [!warning] Rules around closing channels
> - Only the **sender** should close a channel, never the receiver — a receiver doesn't know if more sends are coming.
> - **Sending on a closed channel panics.**
> - **Closing an already-closed channel panics.**
> - Closing is often unnecessary — a channel doesn't need to be closed for its goroutines to be garbage collected once nothing references it; close it specifically when receivers need a signal that no more values are coming (as `range` relies on).

## 4. select — waiting on multiple channels

```go
select {
case msg1 := <-ch1:
    fmt.Println("from ch1:", msg1)
case ch2 <- value:
    fmt.Println("sent to ch2")
case <-time.After(2 * time.Second):
    fmt.Println("timeout")
default:
    fmt.Println("nothing ready right now — don't block")
}
```

`select` blocks until one of its cases can proceed, choosing **pseudo-randomly** among multiple simultaneously-ready cases (deliberately, to prevent code from depending on which one "wins" when several are ready at once). A `default` case makes the whole `select` non-blocking — if nothing else is ready immediately, `default` runs instead of waiting.

### Timeouts and cancellation with select

```go
select {
case result := <-resultCh:
    return result, nil
case <-ctx.Done():
    return nil, ctx.Err()   // context cancelled or deadline exceeded — see Concurrency Patterns
}
```

This `select` + `context.Context` combination is the standard idiom for adding a timeout or cancellation path to any blocking channel operation — see [[Concurrency Patterns]] for the fuller `context` treatment.

## 5. Directional channel types

```go
func producer(out chan<- int) {   // send-only view of the channel
    out <- 42
}
func consumer(in <-chan int) {     // receive-only view of the channel
    v := <-in
}
```

Restricting a function parameter to `chan<-` (send-only) or `<-chan` (receive-only) documents intent and lets the compiler catch misuse (e.g., accidentally trying to receive from a channel a function was only supposed to send to) — a plain `chan T` can always be passed where a directional type is expected, but not the reverse.

## See also
- [[Goroutines]]
- [[The sync Package]]
- [[Concurrency Patterns]]

#go #concurrency #channels

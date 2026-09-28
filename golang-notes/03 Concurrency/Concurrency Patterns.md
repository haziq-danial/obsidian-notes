---
tags: [go, concurrency, patterns]
---

# Concurrency Patterns

> [!summary] Summary
> A handful of idiomatic patterns — worker pools, pipelines, fan-out/fan-in, and context-based cancellation — cover the overwhelming majority of real-world concurrent Go code, all built from [[Goroutines]], [[Channels and Select]], and [[The sync Package]].

## 1. Worker pool

A fixed number of goroutines ("workers") pull tasks from a shared channel, bounding concurrency to a known number regardless of how many tasks arrive:

```go
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        results <- j * 2   // do the work
    }
}

jobs := make(chan int, 100)
results := make(chan int, 100)

for w := 1; w <= 3; w++ {       // exactly 3 workers, however many jobs there are
    go worker(w, jobs, results)
}

for j := 1; j <= 9; j++ {
    jobs <- j
}
close(jobs)   // workers' range loops exit once jobs is drained and closed

for a := 1; a <= 9; a++ {
    <-results
}
```

Bounding concurrency this way prevents, say, an accidental "spawn one goroutine per row of a 10-million-row file" from exhausting memory or overwhelming a downstream service.

## 2. Pipelines

Chaining stages together, each a goroutine reading from one channel and writing to another:

```go
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

for v := range square(generate(1, 2, 3, 4)) {
    fmt.Println(v)   // 1, 4, 9, 16
}
```

Each stage closes its output channel when its input is exhausted, letting the whole pipeline shut down cleanly stage by stage — the pattern generalizes to any number of stages, each independently concurrent with the others.

## 3. Fan-out, fan-in

**Fan-out**: multiple goroutines reading from the same channel to parallelize work across them (the worker pool above is a fan-out). **Fan-in**: multiple goroutines' output channels merged into one:

```go
func merge(cs ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    wg.Add(len(cs))
    for _, c := range cs {
        go func(c <-chan int) {
            defer wg.Done()
            for v := range c {
                out <- v
            }
        }(c)
    }
    go func() {
        wg.Wait()
        close(out)   // safe to close only after ALL input goroutines have finished sending
    }()
    return out
}
```

`sync.WaitGroup` (see [[The sync Package]]) is exactly what makes it safe to close the merged output channel — closing too early, before every input goroutine finishes, would panic on their next send.

## 4. Cancellation with context.Context

```go
func fetchWithTimeout(ctx context.Context, url string) ([]byte, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()

    req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err   // includes context.DeadlineExceeded if the timeout fired
    }
    defer resp.Body.Close()
    return io.ReadAll(resp.Body)
}
```

`context.Context` is the standard way to thread **cancellation** and **deadlines** through a call chain, including across goroutines and network calls — a parent context being cancelled (explicitly, or via a timeout) propagates to every `context.WithCancel`/`WithTimeout` child derived from it, letting a whole tree of in-flight work stop promptly instead of continuing pointlessly (or leaking, per [[Goroutines]] §5) after the caller has stopped waiting on it.

| Function | Use case |
|---|---|
| `context.Background()` | the root context, typically at `main()` or the start of a request |
| `context.WithCancel(parent)` | manual cancellation via the returned `cancel()` function |
| `context.WithTimeout(parent, d)` | automatic cancellation after a duration |
| `context.WithDeadline(parent, t)` | automatic cancellation at an absolute time |
| `context.WithValue(parent, key, val)` | attach request-scoped data (used sparingly — not a general parameter-passing mechanism) |

> [!tip] Convention: context.Context is always the first parameter
> Idiomatic Go functions that can block or need cancellation take `ctx context.Context` as their **first** parameter, named `ctx`, and check `ctx.Done()`/`ctx.Err()` at points where it would otherwise keep blocking — this consistent convention is what lets cancellation compose cleanly across unrelated libraries and layers of a call stack.

## 5. errgroup — WaitGroup plus error propagation

The `golang.org/x/sync/errgroup` package (a widely-used quasi-standard extension) combines a `WaitGroup`-like wait with automatic **error propagation and cancellation**: if any goroutine in the group returns an error, the group's derived context is cancelled and `Wait()` returns that first error — the concurrent equivalent of "the first failure aborts the batch."

## See also
- [[Goroutines]]
- [[Channels and Select]]
- [[The sync Package]]

#go #concurrency #patterns

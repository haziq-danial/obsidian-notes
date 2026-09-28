---
tags: [go, concurrency, sync]
---

# The sync Package

> [!summary] Summary
> While channels are Go's idiomatic tool for coordinating goroutines that communicate, the `sync` package provides the lower-level primitives — mutexes, wait groups, and once-only initialization — needed for protecting genuinely shared memory or waiting on completion without passing data.

## 1. sync.Mutex — mutual exclusion

```go
type SafeCounter struct {
    mu    sync.Mutex
    count int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}
```

`Lock()`/`Unlock()` protect a critical section from concurrent access — this is the standard fix for the "concurrent map/struct field access" data races mentioned in [[Maps]] §6. Pairing `Unlock()` with `defer` immediately after `Lock()` guarantees the lock is released even if the function panics or has multiple return paths.

## 2. sync.RWMutex — multiple readers or one writer

```go
type Cache struct {
    mu   sync.RWMutex
    data map[string]string
}

func (c *Cache) Get(key string) string {
    c.mu.RLock()
    defer c.mu.RUnlock()
    return c.data[key]
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.data[key] = value
}
```

`RWMutex` allows any number of concurrent readers (`RLock`/`RUnlock`) OR exactly one writer (`Lock`/`Unlock`), never both at once — a worthwhile optimization specifically when reads vastly outnumber writes, since concurrent readers don't block each other the way a plain `Mutex` would.

## 3. sync.WaitGroup — waiting for a batch to finish

```go
var wg sync.WaitGroup

for _, url := range urls {
    wg.Add(1)
    go func(u string) {
        defer wg.Done()
        fetch(u)
    }(url)
}

wg.Wait()   // blocks until every Add() has a matching Done()
```

`WaitGroup` is a counter: `Add(n)` increases it, `Done()` (equivalent to `Add(-1)`) decreases it, and `Wait()` blocks until it reaches zero — the standard way to know when a fan-out of goroutines has all finished, directly addressing the "program exits before goroutines finish" issue from [[Goroutines]] §3.

> [!warning] Call Add() before starting the goroutine, not inside it
> Calling `wg.Add(1)` from inside the goroutine it's meant to track creates a race: `wg.Wait()` on the main goroutine might run (and return immediately, seeing a counter of zero) before the spawned goroutine gets a chance to call `Add(1)` at all. Always `Add()` from the goroutine that's doing the spawning, before the `go` statement.

## 4. sync.Once — exactly-once initialization

```go
var once sync.Once
var config *Config

func GetConfig() *Config {
    once.Do(func() {
        config = loadConfig()   // runs exactly once, no matter how many goroutines call GetConfig()
    })
    return config
}
```

`Once.Do(f)` guarantees `f` runs exactly once across all callers, even under concurrent calls from many goroutines simultaneously — the standard pattern for thread-safe lazy initialization (e.g., a singleton, a expensive-to-build shared resource) without a separate explicit mutex-guarded "already initialized?" check.

## 5. sync.Map — a specialized concurrent map

```go
var m sync.Map
m.Store("key", 42)
v, ok := m.Load("key")
m.Range(func(k, v any) bool {
    fmt.Println(k, v)
    return true   // continue iterating
})
```

> [!note] sync.Map is not a general-purpose replacement for map + Mutex
> `sync.Map` is optimized specifically for two access patterns: (1) a key written once and read many times ("write-once, read-many"), or (2) many goroutines each operating on disjoint sets of keys. For general-purpose concurrent map access with lots of mixed reads/writes on overlapping keys, an ordinary `map` guarded by a `sync.Mutex`/`sync.RWMutex` is usually simpler and just as fast, or faster — reach for `sync.Map` only after profiling shows it actually helps your specific access pattern.

## 6. sync/atomic — lock-free primitives

```go
var counter atomic.Int64
counter.Add(1)
v := counter.Load()
```

For simple counters/flags, `sync/atomic`'s typed atomic types (`atomic.Int64`, `atomic.Bool`, `atomic.Pointer[T]`, …) provide lock-free, hardware-instruction-backed operations — lower overhead than a `Mutex` for simple cases, at the cost of a more restrictive API than a general critical section.

## See also
- [[Goroutines]]
- [[Channels and Select]]
- [[Concurrency Patterns]]

#go #concurrency #sync

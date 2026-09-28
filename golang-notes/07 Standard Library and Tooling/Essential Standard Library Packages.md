---
tags: [go, standard-library]
---

# Essential Standard Library Packages

> [!summary] Summary
> Go's standard library is unusually complete for a systems language — production-grade HTTP servers, JSON, cryptography, and more ship in the box, which is part of why Go projects tend to accumulate fewer external dependencies than equivalent projects in many other languages.

## 1. fmt — formatting and printing

```go
fmt.Println("hello", 42)
fmt.Printf("name: %s, age: %d\n", name, age)
s := fmt.Sprintf("formatted: %v", value)
fmt.Errorf("failed: %w", err)   // see Error Wrapping
```

| Verb | Meaning |
|---|---|
| `%v` | default format for the value |
| `%+v` | default format, plus field names for structs |
| `%#v` | Go-syntax representation |
| `%T` | the value's type |
| `%d`, `%s`, `%f`, `%t` | integer, string, float, bool |
| `%w` | wrap an error (only valid with `Errorf`, see [[Error Wrapping]]) |

## 2. net/http — production-capable HTTP out of the box

```go
http.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, %s!", r.URL.Query().Get("name"))
})
log.Fatal(http.ListenAndServe(":8080", nil))
```

A real, concurrent HTTP server (one [[Goroutines|goroutine]] per connection, automatically) is available with no external dependency — this is a significant reason Go became popular for microservices, since a production-viable HTTP server ships in the standard library rather than requiring a third-party framework as a baseline.

## 3. encoding/json — struct tags drive (de)serialization

```go
type User struct {
    Name string `json:"name"`
    Age  int    `json:"age,omitempty"`
}

data, err := json.Marshal(User{Name: "Alice", Age: 30})
var u User
err = json.Unmarshal(data, &u)
```

See [[Structs]] §4 for the struct-tag mechanics this package relies on.

## 4. context — cancellation and deadlines

Covered in depth in [[Concurrency Patterns]] §4 — the standard way to propagate cancellation, timeouts, and (sparingly) request-scoped values through a call chain.

## 5. io and bufio — composable stream abstractions

```go
io.Reader   // one method: Read(p []byte) (n int, err error)
io.Writer   // one method: Write(p []byte) (n int, err error)

reader := bufio.NewReader(file)
line, err := reader.ReadString('\n')

io.Copy(dst, src)   // stream data from any Reader to any Writer, without loading it all into memory
```

`io.Reader`/`io.Writer` are the canonical example of Go's "small interfaces compose broadly" philosophy (see [[Interfaces]] §2) — files, network connections, compressed streams, and in-memory buffers are all interchangeable wherever one of these interfaces is expected.

## 6. time — durations, timestamps, and timers

```go
time.Sleep(2 * time.Second)
deadline := time.Now().Add(5 * time.Minute)
timer := time.NewTimer(1 * time.Second)
ticker := time.NewTicker(500 * time.Millisecond)
elapsed := time.Since(start)
```

`time.Duration` is just an `int64` count of nanoseconds under the hood, but its type distinctness (you can't accidentally add a plain `int` to a `time.Duration` without an explicit conversion) prevents a whole class of unit-confusion bugs common in APIs that take a raw number of milliseconds/seconds.

## 7. strings and strconv — text manipulation and conversion

```go
strings.Split(s, ",")
strings.Contains(s, "sub")
strings.TrimSpace(s)
strings.Builder{}   // efficient string concatenation in a loop, avoiding repeated allocation

n, err := strconv.Atoi("42")          // string → int
s := strconv.Itoa(42)                   // int → string
f, err := strconv.ParseFloat("3.14", 64)
```

> [!tip] Use strings.Builder for concatenation in a loop
> Repeated `s += x` in a loop reallocates and copies the growing string on every iteration (strings are immutable in Go). `strings.Builder` (or a `[]byte` buffer) accumulates writes into a growable buffer, converting to a final string only once — the standard fix for a common, easy-to-write performance mistake.

## 8. log/slog — structured logging (Go 1.21+)

```go
slog.Info("user logged in", "user_id", 42, "ip", req.RemoteAddr)
```

`log/slog` is the standard library's structured logging package, producing key-value (or JSON) formatted log output natively — before its addition, structured logging in Go required a third-party library (`logrus`, `zap`, `zerolog`), all still common and often faster/more feature-rich, but `slog` covers the common case without any external dependency.

## See also
- [[Concurrency Patterns]]
- [[Error Wrapping]]
- [[Go Tooling]]

#go #standard-library

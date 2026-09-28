---
tags: [go, tooling]
---

# Go Tooling

> [!summary] Summary
> Beyond `go build`/`go test` (see [[Go Toolchain]]), a set of first-party and widely-adopted third-party tools cover static analysis, linting, profiling, and documentation — most bundled directly with the `go` command itself.

## 1. go vet — catching suspicious code

```bash
go vet ./...
```

`vet` flags constructs that compile fine but are almost certainly bugs: a `Printf`-style call whose format verbs don't match its arguments, a struct passed by value where it contains a `sync.Mutex` (copying a mutex silently breaks its locking guarantee), unreachable code, and more. `go test` runs a subset of `vet`'s checks automatically before running tests, on the theory that a program `vet` flags is rarely worth testing until the flagged issue is addressed.

## 2. staticcheck and golangci-lint — deeper linting

`go vet` is deliberately conservative (few false positives, but limited scope). Community tools go further:
- **staticcheck**: a much broader static analysis suite (unused code, simplifiable expressions, common performance anti-patterns).
- **golangci-lint**: an aggregator that runs dozens of linters (including `vet` and `staticcheck`) in one fast, configurable pass — the de facto standard for CI lint gates in most Go projects.

## 3. pprof — CPU and memory profiling

```go
import _ "net/http/pprof"
// then: go tool pprof http://localhost:6060/debug/pprof/profile
```

```bash
go test -cpuprofile=cpu.out -bench=.
go tool pprof cpu.out
```

`pprof` (import it as a side-effecting blank import to auto-register HTTP profiling endpoints, or generate profiles directly from benchmarks) produces CPU, memory, goroutine, and blocking profiles, explorable as a call graph, flame graph, or top-N list — the standard tool for answering "where is this program actually spending its time/memory" rather than guessing.

## 4. go doc — documentation from the command line

```bash
go doc fmt.Println
go doc -all net/http
```

Go documentation is written as ordinary comments directly above the declaration they describe (no special docstring syntax, no separate doc-comment format) — `go doc` renders these on demand, and the same comments populate pkg.go.dev automatically for published modules.

```go
// Add returns the sum of a and b.
func Add(a, b int) int {
    return a + b
}
```

> [!tip] Doc comment convention
> A doc comment should be a complete sentence starting with the name being documented (`"Add returns..."` not `"returns the sum..."`) — this convention is what lets tooling extract a clean one-line summary for listings, and it's enforced loosely by `go vet`/`staticcheck` style checks.

## 5. gopls — the language server

`gopls` is Go's official Language Server Protocol implementation, powering autocomplete, jump-to-definition, inline error/lint highlighting, and refactoring support in editors (VS Code's Go extension, Neovim, GoLand, etc.) — nearly all modern Go editor tooling is a thin UI layer over `gopls` rather than a bespoke per-editor implementation.

## 6. Delve — the debugger

```bash
dlv debug main.go
dlv test ./...
```

**Delve** is the de facto standard Go debugger, supporting breakpoints, stepping, goroutine inspection, and variable evaluation — Go's runtime and calling conventions differ enough from C that generic debuggers (`gdb`) work poorly with it, which is why a Go-specific debugger became necessary and standard.

## 7. go generate — code generation

```go
//go:generate stringer -type=Weekday
```

```bash
go generate ./...
```

`go generate` scans for `//go:generate` comments and runs the command each one specifies — commonly used to regenerate boilerplate (String() methods for enum-like constants via `stringer`, mock implementations of interfaces via `mockgen`, protobuf bindings) from a source-of-truth definition, keeping generated code in the repository but clearly regenerable rather than hand-maintained.

## See also
- [[Go Toolchain]]
- [[Testing in Go]]
- [[The Go Scheduler]]

#go #tooling

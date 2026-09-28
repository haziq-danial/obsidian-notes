---
tags: [go, testing]
---

# Testing in Go

> [!summary] Summary
> Testing is built into the toolchain: files named `*_test.go` and functions named `TestXxx` are discovered and run automatically by `go test`, with no external test framework or assertion library required (though several popular ones exist for convenience).

## 1. Basic test structure

```go
// file: mathutil/add_test.go
package mathutil

import "testing"

func TestAdd(t *testing.T) {
    got := Add(2, 3)
    want := 5
    if got != want {
        t.Errorf("Add(2, 3) = %d; want %d", got, want)
    }
}
```

- The file must end in `_test.go` — the `go build` compiler ignores these entirely for normal builds; only `go test` compiles and runs them.
- A test function must be named `TestXxx` (capital first letter after `Test`) and take exactly `t *testing.T`.
- `t.Errorf`/`t.Fatalf` mark the test as failed and log a message; `Fatalf` additionally stops that test function immediately, while `Errorf` lets it continue (useful for reporting multiple independent problems in one test run).

```bash
go test ./...              # run all tests in the module
go test -run TestAdd        # run only tests matching this pattern
go test -v                   # verbose — show each test name and pass/fail
go test -race                # run with the race detector enabled (see below)
go test -cover                # report code coverage percentage
```

## 2. Table-driven tests

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -2, -3, -5},
        {"zero", 0, 5, 5},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)
            if got != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

> [!tip] The standard idiom for testing multiple cases
> **Table-driven tests** — a slice of anonymous [[Structs|struct]] literals describing each case, run in a loop via `t.Run` (which gives each sub-test its own name in output and lets them run independently) — are the overwhelmingly standard way to test a function against many inputs in Go, favored over writing a separate `TestXxx` function per case.

## 3. Benchmarks

```go
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}
```

```bash
go test -bench=. -benchmem
```

A `BenchmarkXxx` function is run repeatedly, with `b.N` automatically adjusted by the testing framework until timing stabilizes — `-benchmem` additionally reports memory allocations per operation, useful for catching unintended allocation in hot paths (related to [[Escape Analysis]]).

## 4. Example tests — runnable, verified documentation

```go
func ExampleAdd() {
    fmt.Println(Add(2, 3))
    // Output: 5
}
```

An `ExampleXxx` function is compiled, run, and its actual stdout compared against the `// Output:` comment — if they don't match, `go test` fails. These examples also appear directly in generated documentation (`go doc`, pkg.go.dev), giving genuinely-tested, always-accurate usage examples instead of documentation comments that can silently drift out of sync with the code.

## 5. The race detector

```bash
go test -race ./...
go run -race main.go
```

> [!tip] Run with -race regularly, especially for concurrent code
> The **race detector** instruments memory accesses at compile time to catch data races — concurrent, unsynchronized access to the same memory where at least one access is a write (exactly the kind of bug [[The sync Package|mutexes]] and channel-based designs exist to prevent). It has real runtime/memory overhead, so it's not used in production, but running the full test suite with `-race` in CI catches an entire class of concurrency bugs that might otherwise only manifest rarely and unpredictably under production load.

## 6. Test helpers and setup/teardown

```go
func TestMain(m *testing.M) {
    setup()
    code := m.Run()
    teardown()
    os.Exit(code)
}

func TestSomething(t *testing.T) {
    t.Cleanup(func() {
        // runs after this test, even if it fails — like a scoped defer for tests
    })
}
```

`TestMain`, if defined in a package, replaces the default test runner entirely — giving a hook for package-wide setup/teardown (spinning up a test database, for instance) around the whole test binary's run. `t.Cleanup` registers a function to run after an individual test finishes, regardless of outcome — the test-scoped equivalent of `defer`.

## 7. Mocking via interfaces

Since Go has no built-in mocking framework, the idiomatic approach is designing dependencies as small [[Interfaces|interfaces]] (see [[Interfaces]] §2) so tests can substitute a hand-written or generated fake implementation — this is one of the practical reasons idiomatic Go favors small, consumer-defined interfaces so heavily.

## See also
- [[Go Toolchain]]
- [[The sync Package]]
- [[Interfaces]]

#go #testing

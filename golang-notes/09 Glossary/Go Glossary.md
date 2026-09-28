---
tags: [go, glossary, reference]
---

# Go Glossary

> [!summary] Summary
> Quick-reference definitions for terms used throughout the vault. Each term links to the note where it's covered in depth.

### A
- **any** — alias for `interface{}`, the empty interface satisfied by every type. See [[Interfaces]].
- **append** — builtin that adds elements to a slice, reallocating its backing array if capacity is exceeded. See [[Arrays and Slices]].

### B
- **Buffered channel** — a channel with capacity > 0; sends only block once the buffer is full. See [[Channels and Select]].

### C
- **Channel** — a typed conduit for sending values between goroutines. See [[Channels and Select]].
- **comma-ok idiom** — the two-value form (`v, ok := ...`) used to check map lookups, type assertions, and channel receives without risking a panic. See [[Maps]], [[Interfaces]].
- **Constraint** — an interface describing which types a generic type parameter may be instantiated with. See [[Generics]].
- **context.Context** — the standard mechanism for propagating cancellation and deadlines through a call chain. See [[Concurrency Patterns]].

### D
- **defer** — schedules a function call to run when the enclosing function returns, regardless of how. See [[Functions]], [[Panic and Recover]].

### E
- **Embedding** — including one type inside a struct or interface to promote its fields/methods, Go's alternative to inheritance. See [[Embedding]].
- **Escape analysis** — the compiler's determination of whether a value can be stack-allocated or must go on the heap. See [[Escape Analysis]].
- **Exported identifier** — a name starting with an uppercase letter, visible outside its declaring package. See [[Packages and Visibility]].

### G
- **G, M, P** — Goroutine, Machine (OS thread), Processor (scheduling context) — the three abstractions of the Go scheduler. See [[The Go Scheduler]].
- **Generics** — type parameters allowing functions/types to work over multiple types with compile-time safety. See [[Generics]].
- **Goroutine** — a lightweight, runtime-managed concurrent unit of execution. See [[Goroutines]].
- **go.mod / go.sum** — the module manifest and checksum-lock files for Go Modules. See [[Go Modules]].
- **GOMAXPROCS** — the number of P's, capping true parallelism. See [[The Go Scheduler]].
- **Garbage collector** — Go's concurrent tri-color mark-and-sweep memory reclaimer. See [[Garbage Collection]].

### I
- **Interface** — a set of method signatures; satisfied automatically (structurally) by any type implementing them. See [[Interfaces]].
- **iota** — an auto-incrementing identifier used to build enumerated constants. See [[Variables and Types]].
- **internal package** — a package under an `internal/` directory, importable only from its parent module tree. See [[Packages and Visibility]].

### M
- **Method** — a function with a receiver, attaching behavior to a type. See [[Methods]].
- **Minimal Version Selection (MVS)** — Go's dependency resolution algorithm, picking the minimum version satisfying every requirement. See [[Go Modules]].

### P
- **panic** — stops normal execution and begins unwinding the stack, running deferred calls along the way. See [[Panic and Recover]].
- **Pointer** — a value holding another value's address; Go has no pointer arithmetic. See [[Pointers]].

### R
- **race detector** — a `go test -race`/`go run -race` instrumentation catching concurrent unsynchronized memory access. See [[Testing in Go]].
- **recover** — stops an in-progress panic when called directly inside a deferred function. See [[Panic and Recover]].

### S
- **select** — waits on multiple channel operations at once, choosing pseudo-randomly among ready cases. See [[Channels and Select]].
- **Sentinel error** — a package-level `error` value compared against with `errors.Is`. See [[Errors as Values]].
- **Slice** — a three-word header (pointer, length, capacity) over an underlying array. See [[Arrays and Slices]].
- **Struct tag** — string metadata on a struct field, read via reflection (e.g., by `encoding/json`). See [[Structs]].
- **sync.Mutex / sync.WaitGroup** — core synchronization primitives for mutual exclusion and waiting on goroutine completion. See [[The sync Package]].

### T
- **Type assertion** — recovers the concrete type stored in an interface value. See [[Interfaces]].
- **Type parameter** — the placeholder type (e.g., `T` in `func Max[T ...]`) in a generic function or type. See [[Generics]].

### U
- **Unexported identifier** — a name starting with a lowercase letter, visible only within its declaring package. See [[Packages and Visibility]].
- **Unbuffered channel** — a channel with capacity 0; send and receive rendezvous directly. See [[Channels and Select]].

### W
- **Wrapping (errors)** — embedding one error inside another via `%w`, preserving it for `errors.Is`/`errors.As`. See [[Error Wrapping]].
- **Work stealing** — an idle scheduling context (P) taking goroutines from a busy one's local queue. See [[The Go Scheduler]].

### Z
- **Zero value** — the automatic default value every declared variable gets if not explicitly initialized. See [[Variables and Types]].

## See also
- [[Go MOC]]

#glossary #reference

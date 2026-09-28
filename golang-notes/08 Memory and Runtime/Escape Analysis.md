---
tags: [go, runtime, escape-analysis]
---

# Escape Analysis

> [!summary] Summary
> Escape analysis is a compile-time process that decides whether each value can be safely allocated on the stack or must be allocated on the heap. The programmer doesn't choose this directly — the compiler infers it from how a value's address is used.

![[escape-analysis-stack-heap.svg]]

## 1. Stack vs heap, and why the distinction matters

- **Stack allocation** is essentially free: it's just moving a stack pointer, and the memory is automatically reclaimed the instant the function returns — no [[Garbage Collection|garbage collector]] involvement at all.
- **Heap allocation** costs more upfront (a real allocator call) and adds work for the garbage collector to eventually reclaim it.

Because stack allocation is so much cheaper, the compiler's default preference is to keep a value on the stack whenever it can **prove** doing so is safe — "escape analysis" is the name for that proof process.

## 2. The core question: does the address outlive the frame?

```go
func sum() int {
    x := 5          // x's address is never taken or exposed anywhere
    return x * 2      // safe to keep x on the stack
}

func makeX() *int {
    x := 5          // x's address IS taken...
    return &x         // ...and returned to the caller, who might use it long after sum() returns
}   // x must escape to the heap — the stack frame it would have lived in is gone once makeX() returns
```

If the compiler can prove a value's lifetime is entirely confined to the current function call (its address is never stored anywhere that outlives the call), it stays on the stack. If the address could plausibly be used after the function returns — returned directly, stored in a longer-lived struct or global, sent on a channel, or captured by a [[Functions|closure]] that itself escapes — the value **escapes** to the heap.

## 3. Common causes of escaping

| Pattern | Why it escapes |
|---|---|
| Returning a pointer to a local variable | The pointer must remain valid after the function returns |
| Storing a pointer in a struct field, global, or map | The reference could be used arbitrarily far in the future |
| Passing a pointer to a function the compiler can't fully analyze (e.g., across a genuinely dynamic interface call) | The compiler can't prove what the callee does with the address |
| A closure capturing a variable by reference, where the closure itself escapes (e.g., is returned or stored) | See [[Functions]] §4 — the captured variable must outlive the enclosing function |
| A value passed to `fmt.Println`/other `...any` variadic functions | Boxing into an interface value can force heap allocation, since the concrete type's size isn't statically known at the call site |
| A slice/map that grows beyond what the compiler can size upfront | Backing storage is inherently heap-allocated once its size isn't knowable at compile time |

## 4. Checking the compiler's actual decisions

```bash
go build -gcflags="-m" ./...
# ./main.go:8:6: moved to heap: x
# ./main.go:12:9: x does not escape
```

This is the authoritative way to find out what actually escapes in real code — guessing from source alone is unreliable, since some cases (like passing a value to certain standard library functions) are non-obvious. `-m -m` (repeated) gives more detail on the compiler's reasoning for borderline decisions.

## 5. Escape analysis and interfaces

```go
func printIt(v any) {
    fmt.Println(v)
}
printIt(42)   // the int 42 is typically boxed into the interface value, often causing an allocation
```

Storing a concrete value inside an interface value (see [[Interfaces]]) can force a heap allocation even for a small value like an `int`, because the interface's data pointer needs something stable to point at, and the compiler often can't prove the boxed copy is short-lived enough to stack-allocate. This is part of why hot, allocation-sensitive code sometimes avoids passing values through `any`-typed APIs (like heavy use of `fmt.Sprintf` in a tight loop) in favor of more specific, non-interface signatures or [[Generics|generics]].

## 6. Don't over-optimize prematurely

> [!tip] Profile before chasing escapes
> Escape analysis is a compiler implementation detail worth understanding, but most Go code should not be hand-tuned around it preemptively — write clear code first, then use `go build -gcflags="-m"` and [[Go Tooling|pprof]]'s allocation profiling to find *actual* hot allocation sites in code that's shown to matter, rather than guessing which of dozens of functions might benefit from restructuring to avoid an escape.

## See also
- [[Garbage Collection]]
- [[Pointers]]
- [[Functions]]

#go #runtime #escape-analysis

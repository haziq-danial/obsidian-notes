---
tags: [go, fundamentals]
---

# What is Go

> [!summary] Summary
> Go (often "Golang") is a compiled, statically-typed, garbage-collected language designed at Google (2007, released 2009) by Robert Griesemer, Rob Pike, and Ken Thompson. Its priorities — fast compilation, simplicity, and built-in concurrency — shape nearly every design decision covered in this vault.

## 1. Why Go was created

Go's designers were reacting directly to pain points in large-scale C++ development at Google: slow builds (minutes to hours for huge codebases), complex dependency management, and language features (deep inheritance hierarchies, templates) that made large teams' code harder to read than to write. Go's answer was deliberate **simplicity**: a small language spec, few keywords (25 total), one way to format code ([[Go Tooling|gofmt]]), and fast compilation as a first-class design goal, not an afterthought.

## 2. Defining characteristics

- **Compiled to a single static binary**: no separate runtime/VM to install on the target machine, no dependency hell at deploy time (see [[Go Toolchain]]).
- **Statically typed with type inference**: `x := 5` infers `int` — types are checked at compile time, but short variable declarations avoid the verbosity of explicit annotations everywhere.
- **Garbage collected**: no manual memory management, but tuned for low, predictable pause times rather than raw throughput (see [[Garbage Collection]]).
- **Built-in concurrency primitives**: [[Goroutines]] and [[Channels and Select|channels]] are part of the language itself, not a bolted-on library — concurrent code looks like ordinary sequential code with `go` and `<-` sprinkled in.
- **Structural typing for interfaces**: a type satisfies an [[Interfaces|interface]] automatically by implementing its methods — no `implements` keyword, no explicit declaration of intent.
- **No classes or inheritance**: behavior composition happens via [[Embedding]] and interfaces instead of class hierarchies.
- **Explicit error handling**: [[Errors as Values|errors are ordinary return values]], checked with `if err != nil`, not thrown/caught exceptions.

## 3. What Go is commonly used for

- **Networked services and APIs**: Go's standard library `net/http` is production-capable out of the box, and goroutines make handling many concurrent connections natural (see [[Goroutines]]).
- **Cloud infrastructure tooling**: Docker, Kubernetes, Terraform, Prometheus, and etcd are all written in Go — its static binaries and low operational overhead suit infrastructure software especially well.
- **CLI tools**: fast startup, a single dependency-free binary, and easy cross-compilation (`GOOS`/`GOARCH`, see [[Go Toolchain]]) make Go a popular choice for command-line tools distributed to end users.
- **Distributed systems and microservices**: lightweight goroutines and channels are a natural fit for services juggling many concurrent requests and background tasks.

## 4. Design trade-offs worth knowing up front

> [!note] What Go deliberately leaves out
> Go has no generics until 1.18 (2022, added carefully and later than most statically-typed languages — see [[Generics]]), no exceptions (by design — see [[Panic and Recover]] for the narrower mechanism it uses instead), no operator overloading, and no implicit type conversions. These aren't oversights; they're the same "make it simple and readable at scale" philosophy that shaped the rest of the language, occasionally at the cost of some expressiveness other languages offer.

## 5. Versioning and stability

Go maintains an unusually strong **backward compatibility promise** (the Go 1 compatibility guarantee): code written for Go 1.0 in 2012 still compiles and runs correctly on modern Go releases. New releases ship roughly every six months, each adding features without breaking existing programs — a deliberate contrast to languages with more frequent breaking changes.

## See also
- [[Go Toolchain]]
- [[Goroutines]]
- [[Errors as Values]]

#go #fundamentals

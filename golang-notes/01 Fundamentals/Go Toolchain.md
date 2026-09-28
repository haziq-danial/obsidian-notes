---
tags: [go, toolchain]
---

# Go Toolchain

> [!summary] Summary
> The `go` command is a single, batteries-included tool that builds, tests, formats, and manages dependencies for Go code — there's no separate build system, package manager, or formatter to install and configure.

![[go-build-pipeline.svg]]

## 1. The compilation pipeline

Running `go build` takes source files straight to a native, statically-linked binary:

1. **Parsing**: source files become an Abstract Syntax Tree (AST).
2. **Type checking**: every expression's type is verified; this stage also does most of what [[Escape Analysis|escape analysis]] needs.
3. **SSA generation and optimization**: code is lowered to Static Single Assignment form for optimization (inlining, dead code elimination, bounds-check elimination).
4. **Code generation**: machine code for the target architecture.
5. **Linking**: all package object code plus the Go runtime (scheduler, garbage collector — see [[The Go Scheduler]] and [[Garbage Collection]]) are combined into one self-contained binary.

> [!note] Why a single static binary matters operationally
> A Go binary has no runtime dependency to install on the deployment target — no JVM, no interpreter, no shared libraries to match versions against (by default, on Linux, with cgo disabled). This is a major reason Go became the default choice for container images and CLI tools: `COPY` one binary into a minimal container image and run it.

## 2. Core go commands

| Command | Effect |
|---|---|
| `go build` | compile packages into a binary (or verify a library compiles) without installing it |
| `go run main.go` | compile and immediately execute, without leaving a binary behind |
| `go test` | compile and run tests (see [[Testing in Go]]) |
| `go vet` | static analysis catching common mistakes (suspicious `Printf` format strings, unreachable code, etc.) |
| `go fmt` | reformat source to the one canonical Go style — see below |
| `go mod init/tidy` | initialize/maintain module dependencies (see [[Go Modules]]) |
| `go get` | add or upgrade a dependency |
| `go install` | build and install a binary into `$GOBIN` |
| `go doc` | view documentation for a package/symbol from the command line |

## 3. gofmt: one style, no debates

> [!tip] There is exactly one "correct" Go formatting
> `gofmt` (run via `go fmt`) is not configurable — indentation, brace placement, alignment are all fixed. This eliminates an entire category of team debate and code-review noise that plagues languages with many acceptable styles; every Go codebase in the world is formatted identically, which is why diffs stay focused on logic rather than style nits, and why tools can safely auto-format on save without surprising anyone.

## 4. Cross-compilation

```bash
GOOS=linux GOARCH=amd64 go build -o myapp-linux ./cmd/myapp
GOOS=windows GOARCH=amd64 go build -o myapp.exe ./cmd/myapp
GOOS=darwin GOARCH=arm64 go build -o myapp-mac-arm ./cmd/myapp
```

Because the toolchain includes a full cross-compiling backend for every supported target out of the box, building a Windows binary from a Linux machine (or any other OS/arch combination) requires nothing more than setting two environment variables — no separate cross-compiler toolchain to install.

## 5. Build tags and constraints

```go
//go:build linux && amd64

package mypackage
```

**Build constraints** conditionally include/exclude a file from compilation based on OS, architecture, or custom tags — commonly used for platform-specific implementations of the same function signature (e.g., `file_linux.go` vs `file_windows.go`, disambiguated automatically by filename suffix or an explicit `//go:build` line).

## 6. The go.dev ecosystem

- **pkg.go.dev**: the canonical documentation site, auto-generated from doc comments in published modules.
- **go.dev/play (the Playground)**: run small Go programs in-browser without any local install — commonly used for sharing runnable examples.

## See also
- [[Go Modules]]
- [[Testing in Go]]
- [[Go Tooling]]

#go #toolchain

---
tags: [go, packages, visibility]
---

# Packages and Visibility

> [!summary] Summary
> Every Go file belongs to a package — the unit of compilation and namespacing. Visibility outside a package is controlled by a single, simple rule: capitalization, with no `public`/`private` keywords needed.

## 1. Package basics

```go
// file: mathutil/add.go
package mathutil

func Add(a, b int) int {
    return a + b
}
```

```go
// file: main.go
package main

import "example.com/myapp/mathutil"

func main() {
    fmt.Println(mathutil.Add(2, 3))
}
```

Every file starts with a `package` declaration; all files in the same directory must declare the same package name (with the historical exception of `_test.go` files, which may use a `_test` suffixed package for black-box testing — see [[Testing in Go]]). `package main` is special: it marks a package as producing an executable (must contain a `func main()`), while any other package name produces a library package importable by others.

## 2. Exported vs unexported — capitalization is the visibility rule

```go
package mathutil

func Add(a, b int) int { return a + b }     // exported — capital A, visible outside the package
func subtract(a, b int) int { return a - b } // unexported — lowercase s, package-private
```

> [!note] No public/private/protected keywords
> A single, purely syntactic rule replaces the visibility keywords found in most other languages: an identifier (function, type, variable, struct field, constant) starting with an **uppercase** letter is exported (visible to importing packages); starting with **lowercase**, it's unexported (visible only within the declaring package). This applies uniformly to everything — functions, types, struct fields, even package-level variables — with no separate syntax needed per kind of declaration.

```go
type Config struct {
    Host string   // exported field — visible and settable from other packages
    port int       // unexported field — only this package can read/set it directly
}
```

## 3. Package-level initialization

```go
package db

var pool *sql.DB

func init() {
    pool = openPool()   // runs automatically before main(), after all package-level var initializers
}
```

An `init()` function (any package may have multiple) runs automatically at program startup, after all package-level variable initializers in that package have run, and before `main()` executes — used for one-time setup that must happen before any of the package's exported functions are called. Multiple `init()` functions (even across multiple files in the same package) all run, in the order their files are presented to the compiler; a package's own dependencies' `init()` functions run first, ensuring init order respects the import graph.

## 4. The internal package convention

```
myproject/
├── internal/
│   └── auth/
│       └── auth.go     // importable only from within myproject/...
├── cmd/
│   └── server/
│       └── main.go     // CAN import myproject/internal/auth
└── ...
```

Any package under a directory named `internal/` can only be imported by code rooted at the parent of that `internal/` directory — enforced by the compiler itself, not just convention. This lets a module expose a public API while keeping true implementation details genuinely unimportable by external consumers, closing a gap that plain exported/unexported capitalization alone doesn't cover (that rule only controls visibility *within* an already-imported package, not whether the package itself can be imported at all).

## 5. Package naming conventions

> [!tip] Short, lowercase, no underscores or stutter
> Idiomatic Go package names are short, all-lowercase, single words (`http`, `json`, `bytes`) — never `snake_case` or `camelCase`. Because callers always prefix usages with the package name (`http.Client`, `json.Marshal`), avoid "stuttering" names inside the package that repeat it (`http.HTTPClient` is redundant; `http.Client` reads cleanly at the call site).

## 6. Import paths and organization

An import path (`example.com/myapp/mathutil`) is both a unique identifier and (via a module's declared path, see [[Go Modules]]) a way for tools to locate the source — by convention rooted at a domain the author controls, avoiding global name collisions across the entire Go ecosystem without a centralized package-name registry.

## See also
- [[Go Modules]]
- [[Go Toolchain]]
- [[Testing in Go]]

#go #packages #visibility

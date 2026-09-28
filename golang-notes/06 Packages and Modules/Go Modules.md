---
tags: [go, modules, dependencies]
---

# Go Modules

> [!summary] Summary
> Go Modules (the default since Go 1.16, replacing the older GOPATH-based workflow) is Go's dependency management system: a `go.mod` file declares a module's identity and requirements, `go.sum` pins exact checksums, and a **Minimal Version Selection** algorithm resolves the whole dependency graph deterministically.

![[go-module-workflow.svg]]

## 1. Creating a module

```bash
go mod init example.com/myapp
```

```
module example.com/myapp

go 1.22

require (
    github.com/gin-gonic/gin v1.9.1
    golang.org/x/sync v0.5.0
)
```

The **module path** (`example.com/myapp`) is both the module's unique identity and the prefix every package inside it is imported by (`example.com/myapp/internal/auth`, see [[Packages and Visibility]] §4) — by convention, rooted at a domain the author controls, so it can also double as the location tooling fetches the source from.

## 2. go.mod vs go.sum

| File | Contents | Purpose |
|---|---|---|
| `go.mod` | module path, Go version, **direct** dependency requirements | human-edited (via `go get`/`go mod tidy`), defines the module |
| `go.sum` | cryptographic checksums for every module version in the **entire** resolved dependency graph | machine-generated, verifies integrity — never hand-edited |

Both files should be committed to version control. `go.sum` existing alongside `go.mod` is what lets `go build`/`go test` verify that every downloaded dependency's content exactly matches what was originally recorded — protecting against a compromised proxy or registry silently swapping out a dependency's contents.

## 3. Adding and updating dependencies

```bash
go get github.com/gin-gonic/gin@v1.9.1   # add or set an exact version
go get github.com/gin-gonic/gin@latest    # upgrade to the latest release
go get -u ./...                            # upgrade all dependencies to their latest minor/patch versions
go mod tidy                                 # add missing requirements, remove unused ones — keeps go.mod accurate
```

`go mod tidy` is commonly run after adding/removing imports in source code — it reconciles `go.mod`/`go.sum` with what the code actually imports, rather than requiring every dependency change to be manually reflected in the manifest.

## 4. Minimal Version Selection (MVS)

> [!note] Go picks the minimum sufficient version, not automatically the newest
> If your module requires `libfoo v1.2.0` and one of your dependencies requires `libfoo v1.4.0`, Go's build resolves to `v1.4.0` — the **minimum version that satisfies every stated requirement** in the whole graph, not necessarily the latest version that exists. This differs from many other ecosystems' resolvers, which often prefer the newest compatible version by default. MVS's guarantee is reproducibility: adding a new, unrelated dependency to your project cannot silently upgrade an existing one out from under you unless something in the graph actually now requires a newer version.

## 5. Semantic Import Versioning

```go
import "example.com/pkg/v2"    // a v2+ module's import path embeds its major version
```

For major version 2 and above, a module's import path must include a `/vN` suffix matching its `go.mod`'s declared module path — this means `example.com/pkg` (implicitly v0 or v1) and `example.com/pkg/v2` are treated as **entirely separate** packages by the toolchain, and a program can even depend on both simultaneously (e.g., during an incremental migration) without conflict, since they're distinguishable import paths, not just different resolved versions of "the same" package.

## 6. Module proxies and GOPROXY

By default, `go get` fetches modules through `proxy.golang.org` (Google's public module mirror/cache) rather than hitting each source repository directly — this provides availability (a source repo going offline doesn't break existing builds that already resolved a version through the proxy) and speed (cached, geographically distributed). `GOPROXY`/`GONOSUMCHECK`/`GOPRIVATE` environment variables configure this behavior, including routing internal/private modules around the public proxy entirely.

## 7. Workspaces (go.work)

```
go 1.22

use (
    ./service-a
    ./service-b
    ./shared-lib
)
```

A `go.work` file lets multiple modules in a local checkout be developed and built together — e.g., testing a change to `shared-lib` against `service-a` without needing to publish a new version and bump `service-a`'s `go.mod` requirement first. It's a local development convenience layered on top of modules, not a replacement for them; `go.work` is typically not committed alongside the modules it references (or is explicitly excluded via `.gitignore`).

## See also
- [[Packages and Visibility]]
- [[Go Toolchain]]

#go #modules

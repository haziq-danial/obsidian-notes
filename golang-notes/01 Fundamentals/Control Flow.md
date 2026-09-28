---
tags: [go, fundamentals, control-flow]
---

# Control Flow

> [!summary] Summary
> Go trims control-flow syntax to a minimum: one loop keyword (`for`, covering while/for/infinite loops), an `if` with an optional initializer, and a `switch` that doesn't fall through by default — each a deliberate simplification versus C-family conventions.

## 1. for — the only loop keyword

```go
for i := 0; i < 10; i++ { }        // classic three-part for

for count < 10 { count++ }          // while-loop equivalent — just drop two of the three parts

for { break }                        // infinite loop — drop all three parts

for i, v := range slice { }          // range form — iterate a slice, array, string, map, or channel
```

> [!note] Why one keyword instead of for/while/do-while
> Go's designers observed that `while` is just a `for` loop missing its init/post clauses — keeping only `for` removes a redundant keyword without losing any expressiveness, consistent with the language's minimalism.

### range's two values

```go
for i, v := range []int{10, 20, 30} {
    fmt.Println(i, v)   // i = index, v = value (a COPY, not a reference into the slice)
}
for k, v := range map[string]int{"a": 1} {
    fmt.Println(k, v)   // k = key, v = value — iteration order is intentionally randomized
}
```

> [!warning] Map iteration order is randomized on purpose
> Go's runtime deliberately randomizes the starting point of map iteration on every run specifically to prevent programs from accidentally depending on an iteration order that was never guaranteed — code that "worked" by relying on incidental map ordering will fail unpredictably, by design, forcing that bug to surface early rather than lurking until a runtime/version change silently reorders things.

## 2. if with an optional initializer

```go
if err := doSomething(); err != nil {
    return err
}
// err is scoped to the if/else, not visible after it
```

This pattern — initialize a variable, immediately check it, scope it to just the branch — is idiomatic throughout Go, especially for the `if err != nil` error-checking convention covered in [[Errors as Values]].

## 3. switch — no fallthrough by default

```go
switch day {
case "Sat", "Sun":
    fmt.Println("weekend")
case "Mon", "Tue", "Wed", "Thu", "Fri":
    fmt.Println("weekday")
default:
    fmt.Println("unknown")
}
```

> [!note] The opposite default from C
> Unlike C/Java/JavaScript, a Go `switch` case does **not** fall through to the next case automatically — each case implicitly `break`s. Explicit `fallthrough` is available when the C-style behavior is genuinely wanted, but it's rare in idiomatic Go precisely because implicit fallthrough is a common source of bugs in C-family languages.

### Type switch

```go
switch v := x.(type) {
case int:
    fmt.Println("int:", v)
case string:
    fmt.Println("string:", v)
case nil:
    fmt.Println("nil")
default:
    fmt.Printf("unknown type: %T\n", v)
}
```

A **type switch** branches on the dynamic type stored in an [[Interfaces|interface]] value — the standard way to handle "one of several possible concrete types" without a chain of type assertions.

## 4. Labeled break/continue

```go
outer:
for i := 0; i < 5; i++ {
    for j := 0; j < 5; j++ {
        if j == 2 {
            continue outer
        }
        if i == 3 {
            break outer
        }
    }
}
```

Labels let `break`/`continue` target an outer loop directly, avoiding the flag-variable workarounds needed in languages without this feature.

## 5. goto

```go
if earlyExit {
    goto done
}
// ... more work ...
done:
    cleanup()
```

`goto` exists but is rare in idiomatic Go — nearly always [[Panic and Recover|defer]] or restructured control flow is preferred; Go's `goto` has restrictions (can't jump into a block or over a variable declaration) that keep it from being used to build genuinely tangled control flow.

## See also
- [[Functions]]
- [[Errors as Values]]
- [[Interfaces]]

#go #fundamentals #control-flow

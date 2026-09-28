---
tags: [go, fundamentals, functions]
---

# Functions

> [!summary] Summary
> Go functions support multiple return values (used pervasively for the `(result, error)` idiom), named return values, variadic parameters, and first-class treatment as values — closures included — without needing a separate lambda syntax.

## 1. Basic syntax and multiple return values

```go
func add(a, b int) int {
    return a + b
}

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

result, err := divide(10, 2)
if err != nil {
    log.Fatal(err)
}
```

Multiple return values are the backbone of Go's error-handling idiom (see [[Errors as Values]]) — a function that can fail returns its result alongside an `error`, and callers are expected to check it immediately rather than letting a failure propagate silently.

## 2. Named return values

```go
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return   // "naked" return — returns the current values of x and y
}
```

Named returns document intent directly in the signature and enable a bare `return` (a "naked return"), but are generally used sparingly in idiomatic Go — mostly for short functions or when a deferred function needs to modify the return value (see the recover pattern in [[Panic and Recover]]).

## 3. Variadic parameters

```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

sum(1, 2, 3)          // nums = []int{1, 2, 3}
nums := []int{1, 2, 3}
sum(nums...)            // spread a slice into variadic args
```

`...T` collects any number of trailing arguments into a `[]T` inside the function — this is how `fmt.Println(a, b, c)` accepts an arbitrary number of arguments of any type (`...any`/`...interface{}`).

## 4. Functions as values and closures

```go
add := func(a, b int) int { return a + b }   // anonymous function literal, assigned to a variable

func makeCounter() func() int {
    count := 0
    return func() int {         // closes over `count`
        count++
        return count
    }
}

counter := makeCounter()
counter()   // 1
counter()   // 2
```

Functions are values with their own type (`func(int, int) int`), can be passed as arguments, returned from other functions, and stored in variables/struct fields — this is what makes closures, middleware patterns (`func(http.Handler) http.Handler`), and functional-style options all natural in Go despite the language having no dedicated "lambda" keyword — function literals fill that role directly.

> [!example] Why the closure captures by reference
> Each call to `makeCounter()` creates a fresh `count` variable that [[Escape Analysis|escapes to the heap]] because the returned closure retains a reference to it — this is exactly the "address outlives the stack frame" scenario escape analysis exists to detect, and it's why each `counter := makeCounter()` gets its own independent, persistent counter rather than all closures secretly sharing one.

## 5. Deferred calls

```go
func readFile(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return err
    }
    defer f.Close()   // guaranteed to run when readFile returns, however it returns
    // ... use f ...
    return nil
}
```

`defer` schedules a call to run when the surrounding function returns — regardless of whether it returns normally, via an early `return`, or while [[Panic and Recover|panicking]]. This is Go's primary mechanism for guaranteed cleanup (closing files, unlocking mutexes, closing channels), covered in depth in [[Panic and Recover]].

## 6. Methods vs functions — a preview

A function with a **receiver** argument (`func (r Receiver) Name(...)`) becomes a method on that type, callable as `r.Name(...)` — see [[Methods]] for the full treatment, including the value-vs-pointer receiver distinction.

## See also
- [[Control Flow]]
- [[Errors as Values]]
- [[Methods]]
- [[Escape Analysis]]

#go #fundamentals #functions

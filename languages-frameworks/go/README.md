# Go — Deep Dive

> Tags: #dx #scalability

Production-focused notes on the Go programming language: patterns, idioms, pitfalls, and optimization.

---

## Contents

- [Concurrency Patterns](#concurrency-patterns)
- [Error Handling](#error-handling)
- [Context & Cancellation](#context--cancellation)
- [Memory & Performance](#memory--performance)
- [Testing Patterns](#testing-patterns)
- [Common Pitfalls](#common-pitfalls)

---

## Concurrency Patterns

### Goroutine Lifecycle Management

Always give goroutines a way to exit:

```go
func worker(ctx context.Context, jobs <-chan Job) {
    for {
        select {
        case <-ctx.Done():
            return
        case job, ok := <-jobs:
            if !ok {
                return // channel closed
            }
            process(job)
        }
    }
}
```

### Worker Pool

Bound concurrency with a fixed pool:

```go
func ProcessItems(ctx context.Context, items []Item, workers int) error {
    jobs := make(chan Item, len(items))
    errs := make(chan error, workers)
    var wg sync.WaitGroup

    for i := 0; i < workers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for item := range jobs {
                if err := process(item); err != nil {
                    errs <- err
                }
            }
        }()
    }

    for _, item := range items {
        jobs <- item
    }
    close(jobs)

    wg.Wait()
    close(errs)

    for err := range errs {
        if err != nil {
            return err // return first error
        }
    }
    return nil
}
```

### Fan-Out / Fan-In

```go
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c {
            out <- n
        }
    }

    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

### sync.Once for Singletons

```go
var (
    instance *Service
    once     sync.Once
)

func GetService() *Service {
    once.Do(func() {
        instance = &Service{/* init */}
    })
    return instance
}
```

---

## Error Handling

### Wrapping Errors (Go 1.13+)

```go
// Wrap with context
if err != nil {
    return fmt.Errorf("creating order for customer %s: %w", customerID, err)
}

// Unwrap
var notFound *NotFoundError
if errors.As(err, &notFound) {
    // handle specifically
}

// Check sentinel
if errors.Is(err, ErrNotFound) {
    // handle
}
```

### Sentinel Errors vs Error Types

```go
// Sentinel (simple, good for control flow)
var ErrNotFound = errors.New("not found")

// Error type (carries context, good for rich inspection)
type ValidationError struct {
    Field   string
    Message string
}
func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}
```

### Avoid Panic for Normal Error Flows

Use `panic` only for programmer errors (invariant violations), not for expected runtime failures.

---

## Context & Cancellation

```go
// Pass context as first argument, always
func FetchUser(ctx context.Context, id string) (*User, error) {
    // ctx carries deadline, cancellation, and request-scoped values
    req, _ := http.NewRequestWithContext(ctx, "GET", "/users/"+id, nil)
    resp, err := http.DefaultClient.Do(req)
    // ...
}

// Timeout
ctx, cancel := context.WithTimeout(parentCtx, 5*time.Second)
defer cancel() // ALWAYS defer cancel to release resources

// Value (use sparingly — only for request-scoped data like trace IDs)
type contextKey string
const traceIDKey contextKey = "traceID"

ctx = context.WithValue(ctx, traceIDKey, traceID)
traceID := ctx.Value(traceIDKey).(string)
```

**Rules:**
- Always pass `ctx` as the first parameter
- Never store `ctx` in a struct
- Always `defer cancel()` immediately after `WithTimeout`/`WithCancel`

---

## Memory & Performance

### Avoid Allocations in Hot Paths

```go
// ❌ Creates a new slice each call
func hotPath() []byte {
    return make([]byte, 1024)
}

// ✅ Use sync.Pool for reuse
var bufPool = sync.Pool{
    New: func() any { return make([]byte, 1024) },
}

func hotPath() []byte {
    return bufPool.Get().([]byte)
}
func returnBuf(b []byte) {
    bufPool.Put(b[:0]) // reset before returning
}
```

### Pre-allocate Slices and Maps

```go
// ❌ Grows many times
var result []Item
for _, v := range data {
    result = append(result, transform(v))
}

// ✅ Single allocation
result := make([]Item, 0, len(data))
for _, v := range data {
    result = append(result, transform(v))
}
```

### Profiling

```go
// CPU profile
f, _ := os.Create("cpu.prof")
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()

// Heap profile
f, _ = os.Create("mem.prof")
pprof.WriteHeapProfile(f)
```

```bash
go tool pprof -http=:6060 cpu.prof
```

### Benchmarks

```go
func BenchmarkMyFunc(b *testing.B) {
    for b.Loop() { // Go 1.24+; use b.N in older versions
        MyFunc()
    }
}
```

---

## Testing Patterns

### Table-Driven Tests

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name    string
        a, b    int
        want    int
    }{
        {"positive", 1, 2, 3},
        {"negative", -1, -2, -3},
        {"zero", 0, 0, 0},
    }
    for _, tc := range tests {
        t.Run(tc.name, func(t *testing.T) {
            got := Add(tc.a, tc.b)
            if got != tc.want {
                t.Errorf("Add(%d, %d) = %d; want %d", tc.a, tc.b, got, tc.want)
            }
        })
    }
}
```

### Dependency Injection for Testability

Define interfaces close to where they're consumed, not where they're defined:

```go
// In your package, define only what you need
type UserFetcher interface {
    FetchUser(ctx context.Context, id string) (*User, error)
}

type OrderService struct {
    users UserFetcher
}
```

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| Loop variable capture in goroutine | All goroutines share same variable | Use `v := v` or pass as arg (fixed in Go 1.22) |
| Nil interface vs nil pointer | Interface with nil concrete value is not nil | Be explicit about interface nil checks |
| Goroutine leak | Goroutine blocked forever | Use `context.Context` for cancellation |
| Closing a nil channel | Panic | Initialize channels before use |
| Shadowing `err` with `:=` | Outer `err` not updated | Use `=` for existing vars |
| Copying a mutex | Data race risk | Use pointer receiver `*sync.Mutex` |

---

## Useful Standard Library Packages

| Package | Use Case |
|---------|----------|
| `sync` | Mutex, RWMutex, WaitGroup, Once, Pool |
| `context` | Cancellation, deadlines, request-scoped values |
| `errors` | Wrapping, unwrapping, Is/As |
| `testing` | Unit tests, benchmarks, fuzzing |
| `net/http` | HTTP client and server |
| `encoding/json` | JSON marshal/unmarshal |
| `database/sql` | SQL database abstraction |
| `io` / `bufio` | Streaming I/O |
| `time` | Timers, tickers, duration |

---

## References

- [Effective Go](https://go.dev/doc/effective_go)
- [Go Proverbs](https://go-proverbs.github.io/)
- [Go Blog](https://go.dev/blog/)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- Book: *The Go Programming Language* — Donovan & Kernighan
- Book: *100 Go Mistakes and How to Avoid Them* — Teiva Harsanyi

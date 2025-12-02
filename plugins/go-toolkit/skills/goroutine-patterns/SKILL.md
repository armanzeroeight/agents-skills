---
name: goroutine-patterns
description: Implement Go concurrency patterns using goroutines, channels, and synchronization primitives. Use when building concurrent systems, implementing parallelism, or managing goroutine lifecycles. Trigger words include "goroutine", "channel", "concurrent", "parallel", "sync", "context".
---

# Goroutine Patterns

Implement Go concurrency patterns for efficient parallel processing.

## Quick Start

**Basic goroutine:**
```go
go func() {
    // Runs concurrently
}()
```

**With channel:**
```go
ch := make(chan int)
go func() {
    ch <- 42
}()
result := <-ch
```

## Instructions

### Step 1: Choose Concurrency Pattern

**Simple parallel execution:**
```go
var wg sync.WaitGroup

for i := 0; i < 10; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        process(id)
    }(i)
}

wg.Wait()
```

**Worker pool:**
```go
jobs := make(chan Job, 100)
results := make(chan Result, 100)

// Start workers
for w := 0; w < numWorkers; w++ {
    go worker(jobs, results)
}

// Send jobs
for _, job := range allJobs {
    jobs <- job
}
close(jobs)

// Collect results
for range allJobs {
    result := <-results
    handleResult(result)
}
```

**Pipeline:**
```go
// Stage 1: Generate
gen := func() <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for i := 0; i < 10; i++ {
            out <- i
        }
    }()
    return out
}

// Stage 2: Process
process := func(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * 2
        }
    }()
    return out
}

// Use pipeline
for result := range process(gen()) {
    fmt.Println(result)
}
```

### Step 2: Implement Channel Communication

**Unbuffered channel (synchronous):**
```go
ch := make(chan int)

go func() {
    ch <- 42 // Blocks until received
}()

value := <-ch // Blocks until sent
```

**Buffered channel (asynchronous):**
```go
ch := make(chan int, 10) // Buffer of 10

ch <- 1 // Doesn't block until buffer full
ch <- 2

value := <-ch // Doesn't block if buffer has data
```

**Select for multiple channels:**
```go
select {
case msg := <-ch1:
    fmt.Println("Received from ch1:", msg)
case msg := <-ch2:
    fmt.Println("Received from ch2:", msg)
case <-time.After(time.Second):
    fmt.Println("Timeout")
}
```

### Step 3: Use Synchronization Primitives

**Mutex for shared state:**
```go
type SafeCounter struct {
    mu    sync.Mutex
    count int
}

func (c *SafeCounter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

**RWMutex for read-heavy workloads:**
```go
type Cache struct {
    mu    sync.RWMutex
    items map[string]string
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    val, ok := c.items[key]
    return val, ok
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.items[key] = value
}
```

**WaitGroup for goroutine coordination:**
```go
var wg sync.WaitGroup

for i := 0; i < 5; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        doWork(id)
    }(i)
}

wg.Wait() // Wait for all goroutines
```

### Step 4: Implement Context for Cancellation

**Basic context usage:**
```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

go func() {
    for {
        select {
        case <-ctx.Done():
            return // Exit when cancelled
        default:
            doWork()
        }
    }
}()

// Cancel when done
cancel()
```

**With timeout:**
```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

result, err := doWorkWithContext(ctx)
if err == context.DeadlineExceeded {
    fmt.Println("Operation timed out")
}
```

**Propagate context:**
```go
func ProcessRequest(ctx context.Context, req Request) error {
    // Pass context to all operations
    data, err := fetchData(ctx, req.ID)
    if err != nil {
        return err
    }
    
    return saveData(ctx, data)
}
```

## Common Patterns

### Fan-Out/Fan-In

```go
func fanOut(in <-chan int, n int) []<-chan int {
    outs := make([]<-chan int, n)
    for i := 0; i < n; i++ {
        outs[i] = process(in)
    }
    return outs
}

func fanIn(channels ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    
    for _, ch := range channels {
        wg.Add(1)
        go func(c <-chan int) {
            defer wg.Done()
            for n := range c {
                out <- n
            }
        }(ch)
    }
    
    go func() {
        wg.Wait()
        close(out)
    }()
    
    return out
}
```

### Rate Limiting

```go
func rateLimiter(rate time.Duration) <-chan time.Time {
    return time.Tick(rate)
}

limiter := rateLimiter(time.Second / 10) // 10 per second

for range limiter {
    go processItem()
}
```

### Timeout Pattern

```go
func doWithTimeout(timeout time.Duration) error {
    done := make(chan error, 1)
    
    go func() {
        done <- doWork()
    }()
    
    select {
    case err := <-done:
        return err
    case <-time.After(timeout):
        return errors.New("timeout")
    }
}
```

### Error Group

```go
import "golang.org/x/sync/errgroup"

func processAll(items []Item) error {
    g := new(errgroup.Group)
    
    for _, item := range items {
        item := item // Capture for goroutine
        g.Go(func() error {
            return process(item)
        })
    }
    
    return g.Wait() // Returns first error
}
```

## Advanced

For detailed patterns:
- [Channels](reference/channels.md) - Channel patterns and best practices
- [Sync](reference/sync.md) - Synchronization primitives
- [Context](reference/context.md) - Context usage and propagation

## Troubleshooting

**Goroutine leaks:**
- Always ensure goroutines can exit
- Use context for cancellation
- Close channels when done

**Deadlocks:**
- Avoid circular channel dependencies
- Don't send on unbuffered channel without receiver
- Use select with timeout

**Race conditions:**
- Use `go run -race` to detect
- Protect shared state with mutexes
- Prefer channels for communication

**Performance issues:**
- Don't create too many goroutines
- Use worker pools for bounded concurrency
- Profile with pprof

## Best Practices

1. **Don't communicate by sharing memory; share memory by communicating**
2. **Always handle goroutine cleanup**: Use context, defer, or done channels
3. **Avoid goroutine leaks**: Ensure all goroutines can exit
4. **Use buffered channels carefully**: Can hide synchronization issues
5. **Prefer sync.WaitGroup**: For waiting on multiple goroutines
6. **Pass context**: Propagate cancellation through call stack
7. **Close channels from sender**: Never close from receiver
8. **Check channel closure**: Use `val, ok := <-ch`

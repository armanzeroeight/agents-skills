# Context

Guide to using context.Context for cancellation, deadlines, and request-scoped values.

## Context Basics

Context carries deadlines, cancellation signals, and request-scoped values across API boundaries.

### Creating Contexts

```go
// Background context (root)
ctx := context.Background()

// TODO context (placeholder)
ctx := context.TODO()

// With cancellation
ctx, cancel := context.WithCancel(parent)
defer cancel()

// With timeout
ctx, cancel := context.WithTimeout(parent, 5*time.Second)
defer cancel()

// With deadline
ctx, cancel := context.WithDeadline(parent, time.Now().Add(5*time.Second))
defer cancel()
```

## Cancellation

### Basic Cancellation

```go
ctx, cancel := context.WithCancel(context.Background())

go func() {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Cancelled:", ctx.Err())
            return
        default:
            doWork()
        }
    }
}()

// Cancel after some time
time.Sleep(time.Second)
cancel()
```

### Timeout Pattern

```go
func doWorkWithTimeout() error {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    return doWork(ctx)
}

func doWork(ctx context.Context) error {
    select {
    case <-ctx.Done():
        return ctx.Err()  // context.DeadlineExceeded
    case result := <-process():
        return nil
    }
}
```

## Context Values

### Storing Values

```go
type key string

const userKey key = "user"

func WithUser(ctx context.Context, user *User) context.Context {
    return context.WithValue(ctx, userKey, user)
}

func GetUser(ctx context.Context) (*User, bool) {
    user, ok := ctx.Value(userKey).(*User)
    return user, ok
}
```

### Best Practices for Values

- Use for request-scoped data only
- Don't use for optional parameters
- Use unexported types for keys
- Document what values are expected

## Propagation

### HTTP Server

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()  // Get request context
    
    // Pass to all operations
    data, err := fetchData(ctx)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    
    json.NewEncoder(w).Encode(data)
}

func fetchData(ctx context.Context) (*Data, error) {
    // Check cancellation
    select {
    case <-ctx.Done():
        return nil, ctx.Err()
    default:
    }
    
    // Fetch data...
    return data, nil
}
```

### Database Queries

```go
func GetUser(ctx context.Context, db *sql.DB, id int) (*User, error) {
    var user User
    err := db.QueryRowContext(ctx, "SELECT * FROM users WHERE id = $1", id).Scan(&user)
    if err != nil {
        return nil, err
    }
    return &user, nil
}
```

### HTTP Client

```go
func fetchURL(ctx context.Context, url string) ([]byte, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    return io.ReadAll(resp.Body)
}
```

## Patterns

### Graceful Shutdown

```go
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    
    // Start workers
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go worker(ctx, &wg, i)
    }
    
    // Wait for interrupt
    sigCh := make(chan os.Signal, 1)
    signal.Notify(sigCh, os.Interrupt)
    <-sigCh
    
    // Cancel context
    cancel()
    
    // Wait for workers to finish
    wg.Wait()
}

func worker(ctx context.Context, wg *sync.WaitGroup, id int) {
    defer wg.Done()
    
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d stopping\n", id)
            return
        default:
            doWork(id)
        }
    }
}
```

### Cascading Cancellation

```go
func parent(ctx context.Context) error {
    // Create child context
    childCtx, cancel := context.WithCancel(ctx)
    defer cancel()
    
    // Start child operation
    errCh := make(chan error, 1)
    go func() {
        errCh <- child(childCtx)
    }()
    
    // Wait for completion or parent cancellation
    select {
    case err := <-errCh:
        return err
    case <-ctx.Done():
        cancel()  // Cancel child
        return ctx.Err()
    }
}
```

## Best Practices

1. **Always pass context as first parameter**: `func Do(ctx context.Context, ...)`
2. **Don't store context in structs**: Pass explicitly
3. **Always call cancel**: Use defer to ensure cleanup
4. **Check Done() in loops**: Allow cancellation
5. **Use context.Background() at top level**: For main, init, tests
6. **Use context.TODO() as placeholder**: When context should be added later
7. **Don't pass nil context**: Use context.Background() instead
8. **Context values for request-scoped only**: Not for optional parameters

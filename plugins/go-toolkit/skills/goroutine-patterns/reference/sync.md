# Sync Primitives

Guide to Go synchronization primitives from the sync package.

## Mutex

Mutual exclusion lock for protecting shared state.

```go
type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}
```

## RWMutex

Read-write mutex for read-heavy workloads.

```go
type Cache struct {
    mu    sync.RWMutex
    items map[string]string
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock()  // Multiple readers allowed
    defer c.mu.RUnlock()
    val, ok := c.items[key]
    return val, ok
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock()  // Exclusive write access
    defer c.mu.Unlock()
    c.items[key] = value
}
```

## WaitGroup

Wait for collection of goroutines to finish.

```go
var wg sync.WaitGroup

for i := 0; i < 10; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        process(id)
    }(i)
}

wg.Wait()  // Block until all Done() called
```

## Once

Execute function exactly once.

```go
var once sync.Once
var instance *Singleton

func GetInstance() *Singleton {
    once.Do(func() {
        instance = &Singleton{}
    })
    return instance
}
```

## Pool

Reusable object pool to reduce allocations.

```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func process() {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer bufferPool.Put(buf)
    buf.Reset()
    
    // Use buffer
}
```

## Cond

Condition variable for waiting on events.

```go
type Queue struct {
    mu    sync.Mutex
    cond  *sync.Cond
    items []int
}

func NewQueue() *Queue {
    q := &Queue{}
    q.cond = sync.NewCond(&q.mu)
    return q
}

func (q *Queue) Enqueue(item int) {
    q.mu.Lock()
    defer q.mu.Unlock()
    q.items = append(q.items, item)
    q.cond.Signal()  // Wake one waiter
}

func (q *Queue) Dequeue() int {
    q.mu.Lock()
    defer q.mu.Unlock()
    
    for len(q.items) == 0 {
        q.cond.Wait()  // Wait for signal
    }
    
    item := q.items[0]
    q.items = q.items[1:]
    return item
}
```

## Best Practices

1. **Always defer Unlock**: Ensure mutex is released
2. **Keep critical sections small**: Minimize time holding lock
3. **Use RWMutex for reads**: When reads >> writes
4. **Avoid nested locks**: Can cause deadlocks
5. **Use sync.Pool carefully**: Objects must be stateless

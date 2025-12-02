# Channels

Comprehensive guide to Go channels for goroutine communication.

## Channel Basics

Channels are typed conduits for sending and receiving values between goroutines.

### Creating Channels

```go
// Unbuffered channel
ch := make(chan int)

// Buffered channel
ch := make(chan int, 10)

// Receive-only channel
var recv <-chan int = ch

// Send-only channel
var send chan<- int = ch
```

## Channel Operations

### Send and Receive

```go
ch <- value    // Send
value := <-ch  // Receive
value, ok := <-ch  // Receive with closed check
```

### Closing Channels

```go
close(ch)  // Close channel

// Check if closed
value, ok := <-ch
if !ok {
    // Channel closed
}

// Range over channel (stops when closed)
for value := range ch {
    process(value)
}
```

## Channel Patterns

### Generator Pattern

```go
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}
```

### Pipeline Pattern

```go
func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

// Use: for n := range square(generate(1, 2, 3)) { ... }
```

### Fan-Out Pattern

```go
func fanOut(in <-chan int, workers int) []<-chan int {
    channels := make([]<-chan int, workers)
    for i := 0; i < workers; i++ {
        channels[i] = worker(in)
    }
    return channels
}
```

### Fan-In Pattern

```go
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

## Select Statement

### Basic Select

```go
select {
case v := <-ch1:
    fmt.Println("Received from ch1:", v)
case v := <-ch2:
    fmt.Println("Received from ch2:", v)
default:
    fmt.Println("No data available")
}
```

### Timeout Pattern

```go
select {
case result := <-ch:
    return result
case <-time.After(5 * time.Second):
    return errors.New("timeout")
}
```

### Done Channel Pattern

```go
done := make(chan struct{})

go func() {
    for {
        select {
        case <-done:
            return
        default:
            doWork()
        }
    }
}()

// Signal done
close(done)
```

## Best Practices

1. **Close from sender**: Only sender should close channels
2. **Check closure**: Use two-value receive to check if closed
3. **Buffered vs unbuffered**: Choose based on synchronization needs
4. **Nil channels**: Receiving from nil channel blocks forever
5. **Direction**: Use directional channels in function signatures

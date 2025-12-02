---
name: error-handling
description: Implement Go error handling patterns including error wrapping, sentinel errors, custom error types, and error handling conventions. Use when handling errors, creating error types, or implementing error propagation. Trigger words include "error", "panic", "recover", "error handling", "error wrapping".
---

# Error Handling

Implement idiomatic Go error handling patterns.

## Quick Start

**Basic error handling:**
```go
result, err := doSomething()
if err != nil {
    return fmt.Errorf("do something: %w", err)
}
```

**Sentinel error:**
```go
var ErrNotFound = errors.New("not found")

if errors.Is(err, ErrNotFound) {
    // Handle not found
}
```

## Instructions

### Step 1: Return Errors

**Basic error return:**
```go
func ReadFile(path string) ([]byte, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("read file %s: %w", path, err)
    }
    return data, nil
}
```

**Multiple return values:**
```go
func Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

### Step 2: Wrap Errors with Context

**Using fmt.Errorf with %w:**
```go
func ProcessFile(path string) error {
    data, err := ReadFile(path)
    if err != nil {
        return fmt.Errorf("process file: %w", err)
    }
    
    if err := Validate(data); err != nil {
        return fmt.Errorf("validate data: %w", err)
    }
    
    return nil
}
```

**Error chain:**
```go
// Original error
err := os.Open("file.txt")

// Wrapped once
err = fmt.Errorf("open config: %w", err)

// Wrapped again
err = fmt.Errorf("initialize app: %w", err)

// Unwrap to check original
if errors.Is(err, os.ErrNotExist) {
    // Handle file not found
}
```

### Step 3: Use Sentinel Errors

**Define sentinel errors:**
```go
var (
    ErrNotFound     = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrInvalidInput = errors.New("invalid input")
)

func GetUser(id string) (*User, error) {
    user, ok := cache[id]
    if !ok {
        return nil, ErrNotFound
    }
    return user, nil
}
```

**Check with errors.Is:**
```go
user, err := GetUser("123")
if errors.Is(err, ErrNotFound) {
    // Handle not found case
    return nil
}
if err != nil {
    // Handle other errors
    return err
}
```

### Step 4: Create Custom Error Types

**Error type with fields:**
```go
type ValidationError struct {
    Field string
    Value interface{}
    Msg   string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %s: %s", e.Field, e.Msg)
}

func Validate(user *User) error {
    if user.Email == "" {
        return &ValidationError{
            Field: "email",
            Value: user.Email,
            Msg:   "email is required",
        }
    }
    return nil
}
```

**Check with errors.As:**
```go
if err := Validate(user); err != nil {
    var valErr *ValidationError
    if errors.As(err, &valErr) {
        fmt.Printf("Field %s failed: %s\n", valErr.Field, valErr.Msg)
    }
    return err
}
```

### Step 5: Handle Errors Appropriately

**Check immediately:**
```go
// Good
result, err := doSomething()
if err != nil {
    return err
}

// Bad - don't defer error checking
result, err := doSomething()
// ... more code ...
if err != nil {
    return err
}
```

**Don't ignore errors:**
```go
// Bad
doSomething()

// Good
if err := doSomething(); err != nil {
    log.Printf("warning: %v", err)
}

// Or explicitly ignore
_ = doSomething()
```

## Common Patterns

### Error Wrapping Chain

```go
func LoadConfig() (*Config, error) {
    data, err := readConfigFile()
    if err != nil {
        return nil, fmt.Errorf("load config: %w", err)
    }
    
    config, err := parseConfig(data)
    if err != nil {
        return nil, fmt.Errorf("load config: %w", err)
    }
    
    return config, nil
}
```

### Multiple Error Returns

```go
func ProcessBatch(items []Item) ([]Result, error) {
    results := make([]Result, 0, len(items))
    
    for _, item := range items {
        result, err := process(item)
        if err != nil {
            return nil, fmt.Errorf("process item %s: %w", item.ID, err)
        }
        results = append(results, result)
    }
    
    return results, nil
}
```

### Collecting Multiple Errors

```go
type MultiError []error

func (m MultiError) Error() string {
    var msgs []string
    for _, err := range m {
        msgs = append(msgs, err.Error())
    }
    return strings.Join(msgs, "; ")
}

func ValidateAll(users []*User) error {
    var errs MultiError
    
    for _, user := range users {
        if err := Validate(user); err != nil {
            errs = append(errs, err)
        }
    }
    
    if len(errs) > 0 {
        return errs
    }
    return nil
}
```

### Panic and Recover

**Use panic for programmer errors:**
```go
func MustCompile(pattern string) *regexp.Regexp {
    re, err := regexp.Compile(pattern)
    if err != nil {
        panic(err)  // Invalid pattern is programmer error
    }
    return re
}
```

**Recover from panics:**
```go
func SafeExecute(fn func()) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic: %v", r)
        }
    }()
    
    fn()
    return nil
}
```

### Error Context

```go
type contextError struct {
    op  string
    err error
}

func (e *contextError) Error() string {
    return fmt.Sprintf("%s: %v", e.op, e.err)
}

func (e *contextError) Unwrap() error {
    return e.err
}

func withContext(op string, err error) error {
    if err == nil {
        return nil
    }
    return &contextError{op: op, err: err}
}
```

## Error Handling in HTTP

```go
func handler(w http.ResponseWriter, r *http.Request) {
    user, err := getUser(r.Context(), r.URL.Query().Get("id"))
    if errors.Is(err, ErrNotFound) {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    if errors.Is(err, ErrUnauthorized) {
        http.Error(w, "Unauthorized", http.StatusUnauthorized)
        return
    }
    if err != nil {
        log.Printf("get user: %v", err)
        http.Error(w, "Internal error", http.StatusInternalServerError)
        return
    }
    
    json.NewEncoder(w).Encode(user)
}
```

## Error Handling in Goroutines

```go
func processAsync(items []Item) error {
    errCh := make(chan error, len(items))
    
    for _, item := range items {
        go func(it Item) {
            errCh <- process(it)
        }(item)
    }
    
    // Collect errors
    for range items {
        if err := <-errCh; err != nil {
            return fmt.Errorf("process failed: %w", err)
        }
    }
    
    return nil
}
```

## Best Practices

1. **Return errors, don't panic**: Except for programmer errors
2. **Check errors immediately**: Don't defer checking
3. **Wrap errors with context**: Use fmt.Errorf with %w
4. **Use sentinel errors**: For expected error cases
5. **Use custom types**: For errors needing additional data
6. **errors.Is for sentinel**: Check wrapped sentinel errors
7. **errors.As for types**: Extract custom error types
8. **Don't ignore errors**: Handle or explicitly ignore with _
9. **Error messages lowercase**: Start with lowercase, no punctuation
10. **Add context**: Include operation and relevant data

## Anti-Patterns

**Don't:**
- Ignore errors silently
- Use panic for normal errors
- Return error strings (use error types)
- Check error strings (use errors.Is/As)
- Create errors with fmt.Sprintf (use fmt.Errorf)
- Wrap errors without %w (loses error chain)

## Troubleshooting

**Error not matching with errors.Is:**
- Ensure using %w when wrapping
- Check sentinel error is same instance

**Can't extract error type:**
- Use errors.As, not type assertion
- Ensure error type is pointer

**Lost error context:**
- Wrap errors at each level
- Include operation name in wrap

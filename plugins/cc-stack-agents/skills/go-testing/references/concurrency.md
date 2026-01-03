# Concurrency Testing

Testing concurrent Go code.

## Race Detector

```bash
# Run tests with race detector
go test -race ./...

# Run specific test with race detector
go test -race -run TestConcurrentFunction
```

## Testing Concurrent Code

```go
func TestCounter(t *testing.T) {
    counter := NewCounter()

    const goroutines = 100
    const incrementsPerGoroutine = 100

    var wg sync.WaitGroup
    wg.Add(goroutines)

    for i := 0; i < goroutines; i++ {
        go func() {
            defer wg.Done()
            for j := 0; j < incrementsPerGoroutine; j++ {
                counter.Increment()
            }
        }()
    }

    wg.Wait()

    expected := goroutines * incrementsPerGoroutine
    if counter.Value() != expected {
        t.Errorf("counter = %d, want %d", counter.Value(), expected)
    }
}
```

## Parallel Tests

```go
func TestParallel(t *testing.T) {
    tests := []struct{name string}{...}

    for _, tt := range tests {
        tt := tt  // Capture range variable
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel()  // Run in parallel

            // Test code...
        })
    }
}
```

## t.Cleanup for Resource Cleanup

```go
func TestWithCleanup(t *testing.T) {
    db := openTestDB(t)
    t.Cleanup(func() {
        db.Close()
    })

    // Test code... cleanup runs even if test fails
}
```

# Benchmarking

Measuring Go code performance.

## Writing Benchmarks

```go
func BenchmarkFunctionName(b *testing.B) {
    for i := 0; i < b.N; i++ {
        FunctionName()
    }
}

// Benchmark with input
func BenchmarkFunctionNameWithInput(b *testing.B) {
    input := prepareInput()
    b.ResetTimer()  // Reset timer to exclude setup
    for i := 0; i < b.N; i++ {
        FunctionName(input)
    }
}
```

## Running Benchmarks

```bash
# Run all benchmarks
go test -bench=.

# Run specific benchmark
go test -bench=BenchmarkFunctionName

# Run with memory allocation stats
go test -bench=. -benchmem

# Run multiple times for stability
go test -bench=. -count=5
```

## Reading Benchmark Output

```
BenchmarkFunctionName-8    1000000    1023 ns/op    512 B/op    8 allocs/op
```

- `BenchmarkFunctionName-8`: Benchmark name, GOMAXPROCS=8
- `1000000`: Number of iterations (b.N)
- `1023 ns/op`: Nanoseconds per operation
- `512 B/op`: Bytes allocated per operation
- `8 allocs/op`: Number of allocations per operation

## Benchmark Comparison

```bash
# Compare before/after
go test -bench=. -benchmem > old.txt
# Make changes...
go test -bench=. -benchmem > new.txt
benchstat old.txt new.txt
```

## pprof for CPU profiling

```bash
# Run benchmark with CPU profiling
go test -bench=. -cpuprofile=cpu.prof

# Analyze profile
go tool pprof cpu.prof

# Generate flame graph
go tool pprof -http=:8080 cpu.prof
```

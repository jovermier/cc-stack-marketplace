---
name: go-performance
description: Go performance optimizations including memory allocation reduction, efficient string building, I/O operations, and resource pooling. Use when optimizing Go code for speed or memory efficiency.
---

# Go Performance

Expert guidance for writing efficient, high-performance Go code.

## Quick Reference

| Concern | Solution | Impact |
|---------|----------|--------|
| String concatenation in loops | strings.Builder | O(n) vs O(n²) |
| Repeated allocations | sync.Pool | Reduces GC pressure |
| Large byte slice to string conversion | unsafe package (carefully) | Avoids allocation |
| I/O operations | bufio.Scanner/buffered writers | Reduces syscalls |
| Defer in tight loops | Move defer outside function | Prevents memory buildup |
| Premature optimization | Don't do it | Measure first |

## What Do You Need?

1. **String operations** - Building, concatenating, converting
2. **Memory allocations** - Reducing allocations, object reuse
3. **I/O efficiency** - Buffered operations, batch processing
4. **Resource pooling** - sync.Pool for frequently allocated objects
5. **Benchmarking** - How to measure improvements

Specify a number or describe your performance concern.

## Routing

| Response | Reference to Read |
|----------|-------------------|
| 1, "string", "concat", "build" | [strings.md](./references/strings.md) |
| 2, "allocation", "memory", "pool" | [allocations.md](./references/allocations.md) |
| 3, "io", "file", "reader", "writer" | [io.md](./references/io.md) |
| 4, "benchmark", "measure", "profile" | [benchmarking.md](./references/benchmarking.md) |
| 5, general performance | Read relevant references |

## Critical Rules

- **Measure before optimizing**: Use benchmarks to identify real bottlenecks
- **strings.Builder for concatenation**: Never use + in loops
- **Avoid defer in tight loops**: Defers accumulate until function return
- **Use bufio for I/O**: Reduces system call overhead
- **sync.Pool for reuse**: Pool frequently allocated objects
- **Prefer fmt.Errorf over errors.New**: For error wrapping context

## Performance Impact Table

| Issue | Performance Cost | Fix |
|-------|------------------|-----|
| String + in loop | O(n²) allocations | strings.Builder |
| Defer in loop | Unbounded memory | Move to separate function |
| Byte to string conversion | Allocates new string | unsafe.String (careful) |
| Unbuffered I/O | System call per byte | bufio.Reader/Writer |
| Repeated allocations | High GC pressure | sync.Pool |
| Substring allocations | O(n) per operation | Use []byte views |

## Common Optimizations

### Strings
```go
// Bad: O(n²) allocations
func build(items []string) string {
    var s string
    for _, item := range items {
        s += item + ","  // New string each iteration
    }
    return s
}

// Good: O(n) with pre-allocation
func build(items []string) string {
    var b strings.Builder
    b.Grow(len(items) * 10)  // Pre-allocate if you can estimate
    for _, item := range items {
        b.WriteString(item)
        b.WriteByte(',')
    }
    return b.String()
}
```

### Defer in Loops
```go
// Bad: Defers accumulate until function returns
func process(files []string) error {
    for _, f := range files {
        file, err := os.Open(f)
        if err != nil {
            return err
        }
        defer file.Close()  // BAD: All deferred until return!
    }
    return nil
}

// Good: Each defer released when function returns
func process(files []string) error {
    for _, f := range files {
        if err := processFile(f); err != nil {
            return err
        }
    }
    return nil
}

func processFile(path string) error {
    file, err := os.Open(path)
    if err != nil {
        return err
    }
    defer file.Close()  // GOOD: Released when processFile returns
    // ...
    return nil
}
```

### sync.Pool
```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func getBuffer() *bytes.Buffer {
    return bufferPool.Get().(*bytes.Buffer)
}

func putBuffer(b *bytes.Buffer) {
    b.Reset()
    bufferPool.Put(b)
}
```

## Reference Index

| File | Topics |
|------|--------|
| [strings.md](./references/strings.md) | Builder, Reader, efficient concatenation |
| [allocations.md](./references/allocations.md) | sync.Pool, escape analysis, reduction strategies |
| [io.md](./references/io.md) | bufio, Scanner, buffered writers, chunking |
| [benchmarking.md](./references/benchmarking.md) | Writing benchmarks, pprof, flame graphs |

## Success Criteria

Code is performant when:
- benchmarks show improvement (measure, don't guess)
- strings.Builder used for string building in loops
- no defer in tight loops
- sync.Pool used for frequently allocated objects
- bufio used for I/O operations
- allocations are minimized (check with -benchmem)

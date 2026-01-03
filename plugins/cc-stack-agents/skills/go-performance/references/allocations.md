# Memory Allocations

Reducing allocations in Go.

## sync.Pool for frequently allocated objects

```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func processData() string {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer func() {
        buf.Reset()
        bufferPool.Put(buf)
    }()

    buf.WriteString("processed data")
    return buf.String()
}
```

## Avoid defer in tight loops

```go
// ❌ Bad: Defers accumulate until function returns
func processFiles(files []string) error {
    for _, file := range files {
        f, err := os.Open(file)
        if err != nil {
            return err
        }
        defer f.Close()  // All defers run at end!
    }
    return nil
}

// ✅ Good: Move defer to separate function
func processFiles(files []string) error {
    for _, file := range files {
        if err := processFile(file); err != nil {
            return err
        }
    }
    return nil
}

func processFile(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return err
    }
    defer f.Close()  // Released when processFile returns
    // Process file...
    return nil
}
```

## Pre-allocate slices when size is known

```go
// ❌ Bad: Grows slice multiple times
var result []string
for i := 0; i < 1000; i++ {
    result = append(result, stringify(i))
}

// ✅ Good: Pre-allocate
result := make([]string, 0, 1000)
for i := 0; i < 1000; i++ {
    result = append(result, stringify(i))
}
```

## Use value receivers for small structs

```go
// ✅ Good: Value receiver for small struct (no allocation)
type Point struct {
    X, Y int
}

func (p Point) String() string {
    return fmt.Sprintf("(%d,%d)", p.X, p.Y)
}

// For large structs, use pointer receiver
type LargeStruct struct {
    data [1024]int
}

func (p *LargeStruct) Process() {
    // ...
}
```

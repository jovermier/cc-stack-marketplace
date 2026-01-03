# String Building

Efficient string operations in Go.

## Use strings.Builder for concatenation in loops

```go
// ❌ Bad: O(n²) time and space
func buildString(items []string) string {
    var result string
    for _, item := range items {
        result += item + ","  // Allocates new string each iteration
    }
    return result
}

// ✅ Good: O(n) time
func buildString(items []string) string {
    var b strings.Builder
    b.Grow(len(items) * 10)  // Pre-allocate if you can estimate
    for _, item := range items {
        b.WriteString(item)
        b.WriteByte(',')
    }
    return b.String()
}
```

## strings.Builder vs fmt.Sprintf

For simple concatenation, Builder is faster:

```go
// ✅ Good: Builder for simple concatenation
var b strings.Builder
b.WriteString("Hello, ")
b.WriteString(name)
b.WriteString("!")

// Use fmt.Sprintf for formatting
msg := fmt.Sprintf("Hello, %s!", name)
```

## Avoid byte slice to string conversion

```go
// ❌ Bad: Allocates new string
data := []byte("hello")
str := string(data)

// ✅ Good: Use unsafe (carefully!) or keep as []byte
str := unsafe.String(unsafe.SliceData(data), len(data))

// Or just keep as []byte if possible
```

## strings.Reader for reading strings

```go
r := strings.NewReader("hello world")
buf := make([]byte, 5)
r.Read(buf)  // Reads "hello"
```

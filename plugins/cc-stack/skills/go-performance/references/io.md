# I/O Operations

Efficient I/O in Go.

## Use bufio for buffered I/O

```go
// ❌ Bad: Unbuffered (many syscalls)
scanner := bufio.NewScanner(file)
for scanner.Scan() {
    processLine(scanner.Text())
}

// ✅ Good: Buffered (fewer syscalls)
scanner := bufio.NewScanner(file)
scanner.Buffer(nil, 64*1024)  // 64KB buffer
for scanner.Scan() {
    processLine(scanner.Text())
}
```

## bufio.Scanner for line-by-line reading

```go
file, err := os.Open("largefile.txt")
if err != nil {
    return err
}
defer file.Close()

scanner := bufio.NewScanner(file)
for scanner.Scan() {
    line := scanner.Text()
    processLine(line)
}

if err := scanner.Err(); err != nil {
    return err
}
```

## io.Copy for efficient copying

```go
// Copy from reader to writer efficiently
dest, err := os.Create("output.txt")
if err != nil {
    return err
}
defer dest.Close()

src, err := os.Open("input.txt")
if err != nil {
    return err
}
defer src.Close()

if _, err := io.Copy(dest, src); err != nil {
    return err
}
```

## io.ReadAll vs streaming

```go
// For small files: io.ReadAll is fine
data, err := io.ReadAll(reader)

// For large files: Stream instead
for {
    buf := make([]byte, 4096)
    n, err := reader.Read(buf)
    if err != nil && err != io.EOF {
        return err
    }
    if n == 0 {
        break
    }
    processChunk(buf[:n])
}
```

## LimitReader for size limits

```go
limitedReader := io.LimitReader(reader, 1024*1024)  // Max 1MB
data, err := io.ReadAll(limitedReader)
if err != nil {
    return err
}
```

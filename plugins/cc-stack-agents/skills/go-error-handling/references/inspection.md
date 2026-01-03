# Error Inspection

Checking and extracting errors with errors.Is and errors.As.

## errors.Is for Sentinel Errors

```go
var (
    ErrNotFound = errors.New("not found")
    ErrDenied   = errors.New("access denied")
)

// Check if error is sentinel
if errors.Is(err, ErrNotFound) {
    // Handle not found
}

// Can check chain
if errors.Is(err, ErrNotFound) || errors.Is(err, ErrDenied) {
    // Handle both cases
}
```

## errors.As for Custom Types

```go
var validationErr *ValidationError
if errors.As(err, &validationErr) {
    fmt.Printf("Field: %s, Message: %s\n", validationErr.Field, validationErr.Message)
}

// Check multiple types
var (
    validationErr *ValidationError
    notFoundErr   *NotFoundError
)
switch {
case errors.As(err, &validationErr):
    // Handle validation error
case errors.As(err, &notFoundErr):
    // Handle not found
default:
    // Handle other errors
}
```

## Unwrapping Errors

```go
// Get the underlying error
unwrapped := errors.Unwrap(err)

// Unwrap multiple levels
for {
    err = errors.Unwrap(err)
    if err == nil {
        break
    }
}
```

## Combining Is and As

```go
// First check sentinel
if errors.Is(err, ErrValidation) {
    // Then extract details
    var validationErr *ValidationError
    if errors.As(err, &validationErr) {
        // Handle with details
    }
}
```

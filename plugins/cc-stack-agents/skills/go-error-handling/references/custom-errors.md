# Custom Error Types

Creating structured error types in Go.

## Basic Custom Error

```go
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed for field %s: %s", e.Field, e.Message)
}

// Usage
func (s *Service) Validate(input Input) error {
    if input.Email == "" {
        return &ValidationError{
            Field:   "email",
            Message: "is required",
        }
    }
    return nil
}
```

## Error with Methods

```go
type NotFoundError struct {
    Resource string
    ID       string
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s with id %s not found", e.Resource, e.ID)
}

func (e *NotFoundError) Is(target error) bool {
    _, ok := target.(*NotFoundError)
    return ok
}
```

## Checking Custom Errors

```go
// Use errors.As
var validationErr *ValidationError
if errors.As(err, &validationErr) {
    fmt.Printf("Field %s: %s\n", validationErr.Field, validationErr.Message)
}
```

## Temporary Errors (for retries)

```go
type Temporary interface {
    Temporary() bool
}

type transientError struct {
    err error
}

func (e *transientError) Error() string { return e.err.Error() }
func (e *transientError) Temporary() bool { return true }
func (e *transientError) Unwrap() error { return e.err }
```

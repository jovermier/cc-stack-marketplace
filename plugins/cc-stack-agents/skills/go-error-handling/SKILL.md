---
name: go-error-handling
description: Go error handling patterns including wrapping, custom error types, errors.Is/As, and error conventions. Use when handling, creating, or checking errors in Go.
---

# Go Error Handling

Expert guidance for proper error handling in Go.

## Quick Reference

| Operation | Pattern | Example |
|-----------|---------|---------|
| Wrap with context | fmt.Errorf with %w | `fmt.Errorf("opening file: %w", err)` |
| Create custom error | struct with Error() | type ValidationError struct {...} |
| Check error type | errors.Is | `errors.Is(err, ErrNotFound)` |
| Extract error | errors.As | `errors.As(err, &validationErr)` |
| Sentinel errors | var at package level | `var ErrNotFound = errors.New("not found")` |
| Ignore errors | Never | Always check err != nil |

## What Do You Need?

1. **Error wrapping** - Adding context to errors
2. **Custom error types** - Creating structured errors
3. **Error inspection** - errors.Is, errors.As
4. **Sentinel errors** - Package-level error values
5. **Error conventions** - When to wrap, return, or create

Specify a number or describe your error handling scenario.

## Routing

| Response | Reference to Read |
|----------|-------------------|
| 1, "wrap", "context", "fmt.Errorf" | [wrapping.md](./references/wrapping.md) |
| 2, "custom", "type", "struct" | [custom-errors.md](./references/custom-errors.md) |
| 3, "check", "errors.Is", "errors.As" | [inspection.md](./references/inspection.md) |
| 4, "sentinel", "package", "global" | [sentinel.md](./references/sentinel.md) |
| 5, general error handling | Read relevant references |

## Critical Rules

- **Never ignore errors**: Always check err != nil
- **Wrap with %w**: Use %w to preserve error type for errors.Is
- **Wrap at boundaries**: Wrap when crossing package boundaries
- **Don't wrap twice**: Avoid double-wrapping the same error
- **Use errors.Is for sentinel**: Check if error is a specific value
- **Use errors.As for types**: Extract and inspect custom error types

## Error Handling Patterns

### Wrapping Errors
```go
// Good: Wrap with context using %w
func (s *Service) Process(id string) error {
    item, err := s.repo.Find(id)
    if err != nil {
        return fmt.Errorf("finding item %s: %w", id, err)
    }
    // ...
}

// Bad: Wrapping with %v loses error type
return fmt.Errorf("finding item %s: %v", id, err)  // Can't use errors.Is()

// Bad: Double wrapping
return fmt.Errorf("processing: %w", fmt.Errorf("finding: %w", err))
```

### Custom Error Types
```go
// Define custom error type
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed for field %s: %s", e.Field, e.Message)
}

// Return custom error
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

### Error Inspection
```go
// Check for sentinel error
if errors.Is(err, ErrNotFound) {
    // Handle not found
}

// Extract and check custom error type
var validationErr *ValidationError
if errors.As(err, &validationErr) {
    // Access validationErr.Field, validationErr.Message
}

// Check multiple possibilities
if errors.Is(err, ErrNotFound) || errors.Is(err, ErrAccessDenied) {
    // Handle both cases
}
```

### Sentinel Errors
```go
// Package-level sentinel errors
var (
    ErrNotFound    = errors.New("not found")
    ErrAccessDenied = errors.New("access denied")
    ErrInvalidInput = errors.New("invalid input")
)

// Use in returns
func (r *Repository) Find(id string) (*Item, error) {
    // ...
    return nil, ErrNotFound
}

// Check in callers
if err != nil {
    if errors.Is(err, ErrNotFound) {
        return nil, nil  // Not found is not an error here
    }
    return nil, err  // Other errors are still errors
}
```

## When to Wrap vs Return

| Scenario | Action |
|----------|--------|
| Crossing package boundary | Wrap with context |
| Internal function | Return as-is |
| Adding retry logic | Don't wrap (check with errors.Is) |
| Adding logging | Log then wrap or return |
| API layer | Wrap for user-friendly messages |

## Common Mistakes

| Mistake | Severity | Fix |
|---------|----------|-----|
| Ignoring errors | Critical | Always check err != nil |
| Using %v instead of %w | High | Use %w to preserve error type |
| Double wrapping | Medium | Wrap only at boundary |
| Panicking on errors | Critical | Return errors, don't panic |
| Creating strings for errors | Low | Use errors.New() or sentinel |
| Wrapping nil error | Medium | Check err != nil before wrapping |

## Reference Index

| File | Topics |
|------|--------|
| [wrapping.md](./references/wrapping.md) | fmt.Errorf with %w, when to wrap |
| [custom-errors.md](./references/custom-errors.md) | Error types, methods, best practices |
| [inspection.md](./references/inspection.md) | errors.Is, errors.As, type switches |
| [sentinel.md](./references/sentinel.md) | Package-level errors, comparison |

## Success Criteria

Error handling is correct when:
- No errors are ignored (all checked)
- Errors wrapped at package boundaries with %w
- Custom error types for domain-specific errors
- Sentinel errors for expected conditions
- errors.Is used for sentinel checking
- errors.As used for type inspection
- No panic on errors (except in package init/main)

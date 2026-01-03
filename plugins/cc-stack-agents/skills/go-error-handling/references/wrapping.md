# Error Wrapping

Adding context to errors with fmt.Errorf.

## Wrap with %w

```go
// ✅ Good: Wrap with %w preserves error type
func (s *Service) Process(id string) error {
    item, err := s.repo.Find(id)
    if err != nil {
        return fmt.Errorf("finding item %s: %w", id, err)
    }
    // ...
}

// ❌ Bad: Using %v loses error type
return fmt.Errorf("finding item %s: %v", id, err)
```

## Why %w Matters

```go
// With %w, errors.Is works
if errors.Is(err, ErrNotFound) {
    // Handle not found
}

// With %v, this check fails!
```

## Wrap at Boundaries

```go
// Repository layer (internal)
func (r *Repo) Find(id string) (*Item, error) {
    item, err := r.db.Query("SELECT * FROM items WHERE id = ?", id)
    if err != nil {
        return nil, err  // Don't wrap internal errors
    }
    return item, nil
}

// Service layer (exposes to other packages)
func (s *Service) GetItem(id string) (*Item, error) {
    item, err := s.repo.Find(id)
    if err != nil {
        return nil, fmt.Errorf("getting item %s: %w", id, err)  // Wrap at boundary
    }
    return item, nil
}
```

## When NOT to Wrap

- Internal functions in same package
- When error is already well-contextualized
- When wrapping would duplicate context

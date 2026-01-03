# Sentinel Errors

Package-level error values for expected conditions.

## Defining Sentinel Errors

```go
package repo

var (
    ErrNotFound    = errors.New("not found")
    ErrAlreadyExists = errors.New("already exists")
    ErrClosed      = errors.New("closed")
)
```

## Using Sentinel Errors

```go
func (r *Repository) Find(id string) (*Item, error) {
    item, err := r.db.QueryOne("SELECT * FROM items WHERE id = ?", id)
    if errors.Is(err, sql.ErrNoRows) {
        return nil, ErrNotFound  // Return sentinel
    }
    if err != nil {
        return nil, err
    }
    return item, nil
}
```

## Checking Sentinel Errors

```go
item, err := repo.Find(id)
if err != nil {
    if errors.Is(err, repo.ErrNotFound) {
        return nil, nil  // Not found is expected here
    }
    return nil, err  // Other errors are still errors
}
```

## When to Use Sentinel Errors

- **Expected conditions**: Not found, already exists, closed
- **Control flow**: When caller needs to handle specifically
- **Public APIs**: Expose stable error values

## When NOT to Use

- **Unexpected errors**: Use regular errors with context
- **Internal details**: Don't expose internal sentinels
- **Rich error data**: Use custom error types instead

## Prefix for Uniqueness

```go
// Good: Prefixed to avoid collisions
var ErrNotFound = errors.New("myapp: not found")

// Or use in package name scope
package repo
var ErrNotFound = errors.New("not found")  // Check as repo.ErrNotFound
```

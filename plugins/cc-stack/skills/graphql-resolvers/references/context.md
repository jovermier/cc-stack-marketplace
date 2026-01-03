# Context

Using context.Context in GraphQL resolvers.

## Passing Context Through Resolvers

```go
func (r *queryResolver) Users(ctx context.Context) ([]*User, error) {
    // Context available here

    // Get authenticated user
    user := auth.FromContext(ctx)

    // Get dataloader
    loaders := dataloader.For(ctx)

    // Use context for external calls
    resp, err := r.httpClient.Do(req.WithContext(ctx))

    // ...
}
```

## Setting Context Values

```go
// Middleware
func Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Add values to context
        ctx := r.Context()

        // Add authenticated user
        user := getUserFromToken(r.Header.Get("Authorization"))
        ctx = context.WithValue(ctx, "user", user)

        // Add request ID
        requestID := uuid.New().String()
        ctx = context.WithValue(ctx, "requestID", requestID)

        // Add dataloader
        loaders := NewLoaders(db)
        ctx = context.WithValue(ctx, "loaders", loaders)

        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

## Context Key Types

```go
// Define context key type to avoid collisions
type contextKey string

const (
    userKey      contextKey = "user"
    requestIDKey contextKey = "requestID"
    loadersKey   contextKey = "loaders"
)

// Helper functions
func WithUser(ctx context.Context, user *User) context.Context {
    return context.WithValue(ctx, userKey, user)
}

func UserFromContext(ctx context.Context) (*User, bool) {
    user, ok := ctx.Value(userKey).(*User)
    return user, ok
}
```

## Context in Nested Resolvers

```go
// Query resolver
func (r *queryResolver) User(ctx context.Context, id string) (*User, error) {
    return r.db.FindUser(id)
}

// Field resolver (context passed through)
func (r *userResolver) Email(ctx context.Context, obj *User) (string, error) {
    // Can access context here
    viewer, ok := UserFromContext(ctx)
    if !ok {
        return "", nil
    }

    // Authorization check
    if obj.ID != viewer.ID && !viewer.IsAdmin {
        return "", nil
    }

    return obj.Email, nil
}
```

## Context Cancellation

```go
func (r *queryResolver) ExpensiveQuery(ctx context.Context) (*Result, error) {
    // Check for cancellation
    select {
    case <-ctx.Done():
        return nil, ctx.Err()
    default:
    }

    // Do expensive work...
    for i := 0; i < 1000000; i++ {
        // Check periodically
        if i%10000 == 0 {
            select {
            case <-ctx.Done():
                return nil, ctx.Err()
            default:
            }
        }
        // ... work ...
    }

    return result, nil
}
```

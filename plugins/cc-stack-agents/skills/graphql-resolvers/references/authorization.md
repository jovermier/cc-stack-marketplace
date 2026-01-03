# Authorization

Authorization checks in GraphQL resolvers.

## Authentication vs Authorization

- **Authentication**: Who is this? (verify identity)
- **Authorization**: What can they do? (check permissions)

## Pattern: Auth in Context

```go
// Set auth middleware
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        user, err := validateToken(token)
        if err != nil {
            http.Error(w, "Unauthorized", 401)
            return
        }

        ctx := context.WithValue(r.Context(), "user", user)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// Get user from context
func FromContext(ctx context.Context) *User {
    user, ok := ctx.Value("user").(*User)
    if !ok {
        return nil
    }
    return user
}
```

## Resolver Authorization

```go
func (r *queryResolver) User(ctx context.Context, id string) (*User, error) {
    // Check authentication
    viewer := FromContext(ctx)
    if viewer == nil {
        return nil, fmt.Errorf("authentication required")
    }

    // Fetch user
    user, err := r.db.FindUser(id)
    if err != nil {
        return nil, err
    }

    // Authorization: users can view own profile, admins can view any
    if user.ID != viewer.ID && !viewer.IsAdmin {
        return nil, fmt.Errorf("access denied")
    }

    return user, nil
}
```

## Field-Level Authorization

```go
func (r *userResolver) Email(ctx context.Context, obj *User) (string, error) {
    viewer := FromContext(ctx)
    if viewer == nil {
        return "", nil  // Not authenticated, return empty
    }

    // Only show email to user themselves or admins
    if obj.ID != viewer.ID && !viewer.IsAdmin {
        return "", nil
    }

    return obj.Email, nil
}
```

## Role-Based Authorization

```go
type Role string

const (
    RoleAdmin Role = "ADMIN"
    RoleUser  Role = "USER"
)

func (u *User) HasRole(role Role) bool {
    for _, r := range u.Roles {
        if r == role {
            return true
        }
    }
    return false
}

// Usage
if !viewer.HasRole(RoleAdmin) {
    return nil, fmt.Errorf("admin access required")
}
```

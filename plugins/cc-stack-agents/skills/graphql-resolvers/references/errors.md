# Error Handling

Proper error handling in GraphQL resolvers.

## Never Expose Internal Errors

```go
// ❌ Bad: Exposes internal details
func (r *mutationResolver) CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserPayload, error) {
    if err := r.db.CreateUser(input); err != nil {
        return nil, fmt.Errorf("database error: %v", err)  // Leaks DB details!
    }
    // ...
}

// ✅ Good: Wrapped errors
func (r *mutationResolver) CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserPayload, error) {
    if err := r.db.CreateUser(input); err != nil {
        if errors.Is(err, db.ErrDuplicate) {
            return &CreateUserPayload{
                Errors: []UserError{{
                    Field:   []string{"email"},
                    Message: "Email already exists",
                }},
            }, nil
        }
        return nil, fmt.Errorf("failed to create user")  // Generic message
    }
    // ...
}
```

## Field-Specific Errors in Payload

```go
type UserError struct {
    Field   []string `json:"field"`    // e.g., ["email"] or ["user", "emails", 0]
    Message string   `json:"message"`
}

type CreateUserPayload struct {
    User   *User       `json:"user"`
    Errors []UserError `json:"errors"`
}

func (r *mutationResolver) CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserPayload, error) {
    var errs []UserError

    // Validate input
    if input.Name == "" {
        errs = append(errs, UserError{
            Field:   []string{"name"},
            Message: "Name is required",
        })
    }
    if !isValidEmail(input.Email) {
        errs = append(errs, UserError{
            Field:   []string{"email"},
            Message: "Invalid email format",
        })
    }

    if len(errs) > 0 {
        return &CreateUserPayload{Errors: errs}, nil
    }

    // Create user...
    return &CreateUserPayload{User: user, Errors: []UserError{}}, nil
}
```

## Error Extensions

```go
import "github.com/vektah/gqlparser/v2/gqlerror"

// Add extensions to errors
func (r *queryResolver) User(ctx context.Context, id string) (*User, error) {
    user, err := r.db.FindUser(id)
    if err != nil {
        if errors.Is(err, db.ErrNotFound) {
            return nil, gqlerror.Errorf("user not found").
                Extend(map[string]interface{}{
                    "code": "NOT_FOUND",
                    "retryable": false,
                })
        }
        return nil, err
    }
    return user, nil
}
```

## Common Error Codes

| Code | Description | Retryable |
|------|-------------|-----------|
| NOT_FOUND | Resource doesn't exist | No |
| ALREADY_EXISTS | Duplicate unique key | No |
| VALIDATION_ERROR | Input validation failed | No |
| UNAUTHORIZED | Not authenticated | No (after login) |
| FORBIDDEN | Not authorized | No |
| INTERNAL | Server error | Maybe |
| RATE_LIMITED | Too many requests | Yes (after delay) |
| SERVICE_UNAVAILABLE | Temporary issue | Yes |

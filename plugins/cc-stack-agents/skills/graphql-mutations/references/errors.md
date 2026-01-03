# Mutation Errors

Error handling in GraphQL mutations.

## Validation Errors (in Payload)

```go
// Validation errors go in payload, not thrown
func (r *mutationResolver) CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserPayload, error) {
    var errs []UserError

    if input.Name == "" {
        errs = append(errs, UserError{
            Field:   []string{"name"},
            Message: "Name is required",
        })
    }

    if len(errs) > 0 {
        return &CreateUserPayload{Errors: errs}, nil  // Return in payload
    }

    // ... create user
    return &CreateUserPayload{User: user}, nil
}
```

## Expected Errors (in Payload)

```go
func (r *mutationResolver) DeleteUser(ctx context.Context, id string) (*DeleteUserPayload, error) {
    user, err := r.db.FindUser(id)
    if err != nil {
        if errors.Is(err, db.ErrNotFound) {
            return &DeleteUserPayload{
                Errors: []UserError{{
                    Message: "User not found",
                }},
            }, nil
        }
    }

    // ... delete
    return &DeleteUserPayload{DeletedUserId: id}, nil
}
```

## Unexpected Errors (Thrown)

```go
// Only unexpected errors are thrown
func (r *mutationResolver) CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserPayload, error) {
    // ... validate and handle expected errors

    // If database connection fails entirely
    if err := r.db.CreateUser(input); err != nil {
        // Unexpected error - throw it (but don't expose internals)
        return nil, fmt.Errorf("failed to create user")
    }

    return &CreateUserPayload{User: user}, nil
}
```

## Error Type Hierarchy

```
Errors
├── Expected (in payload)
│   ├── Validation errors
│   ├── Not found
│   ├── Already exists
│   └── Permission denied
└── Unexpected (thrown)
    ├── Database connection failed
    ├── External service unavailable
    └── Internal server error
```

## Client Handling

```graphql
mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
        user {
            id
            name
        }
        errors {
            field
            message
        }
    }
}
```

```typescript
// Client handles errors in payload
const result = await createUser({ variables: { input } })
if (result.data?.createUser?.errors?.length > 0) {
    // Display validation errors
    showErrors(result.data.createUser.errors)
} else if (result.data?.createUser?.user) {
    // Success
    navigate(`/users/${result.data.createUser.user.id}`)
}
```

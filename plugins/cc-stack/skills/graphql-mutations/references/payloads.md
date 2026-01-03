# Mutation Payloads

Designing mutation return types.

## Standard Payload Structure

```graphql
type CreateUserPayload {
    user: User
    errors: [UserError!]!
}

type UserError {
    field: [String!]!  # Path to field
    message: String!
}
```

## Create Mutation Payload

```graphql
type Mutation {
    createUser(input: CreateUserInput!): CreateUserPayload!
}

type CreateUserPayload {
    """
    The created user, or null if validation failed
    """
    user: User

    """
    Validation errors, empty if successful
    """
    errors: [UserError!]!
}
```

## Update Mutation Payload

```graphql
type Mutation {
    updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
}

type UpdateUserPayload {
    """
    The updated user
    """
    user: User

    """
    Validation errors
    """
    errors: [UserError!]!
}
```

## Delete Mutation Payload

```graphql
type Mutation {
    deleteUser(id: ID!): DeleteUserPayload!
}

type DeleteUserPayload {
    """
    The ID of the deleted user
    """
    deletedUserId: ID

    """
    Errors (e.g., not found, permission denied)
    """
    errors: [UserError!]!
}
```

## Field Path Convention

```go
// Single field
UserError{Field: []string{"email"}, Message: "Invalid format"}

// Nested field
UserError{Field: []string{"user", "emails", 0}, Message: "Invalid"}

// Multiple fields
UserError{Field: []string{"password", "confirmation"}, Message: "Must match"}
```

## Implementation

```go
func (r *mutationResolver) CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserPayload, error) {
    // Validate
    if errs := validate(input); len(errs) > 0 {
        return &CreateUserPayload{Errors: errs}, nil
    }

    // Create
    user, err := r.db.CreateUser(input)
    if err != nil {
        if errors.Is(err, db.ErrDuplicate) {
            return &CreateUserPayload{
                Errors: []UserError{{
                    Field:   []string{"email"},
                    Message: "Email already exists",
                }},
            }, nil
        }
        return nil, err
    }

    return &CreateUserPayload{User: user, Errors: []UserError{}}, nil
}
```

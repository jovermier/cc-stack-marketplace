# Validation

Schema-level validation in GraphQL.

## Validate at Schema Layer

Use GraphQL types and directives for validation:

```graphql
input CreateUserInput {
    """
    User's display name (2-100 characters)
    """
    name: String!

    """
    Email address (must be valid format)
    """
    email: String!

    """
    Password (min 8 characters)
    """
    password: String!
}
```

## Custom Validation in Resolvers

```go
func validateCreateUserInput(input CreateUserInput) []UserError {
    var errs []UserError

    // Name validation
    if input.Name == "" {
        errs = append(errs, UserError{
            Field:   []string{"name"},
            Message: "Name is required",
        })
    } else if len(input.Name) < 2 {
        errs = append(errs, UserError{
            Field:   []string{"name"},
            Message: "Name must be at least 2 characters",
        })
    } else if len(input.Name) > 100 {
        errs = append(errs, UserError{
            Field:   []string{"name"},
            Message: "Name must be less than 100 characters",
        })
    }

    // Email validation
    if input.Email == "" {
        errs = append(errs, UserError{
            Field:   []string{"email"},
            Message: "Email is required",
        })
    } else if !isValidEmail(input.Email) {
        errs = append(errs, UserError{
            Field:   []string{"email"},
            Message: "Invalid email format",
        })
    }

    // Password validation
    if input.Password == "" {
        errs = append(errs, UserError{
            Field:   []string{"password"},
            Message: "Password is required",
        })
    } else if len(input.Password) < 8 {
        errs = append(errs, UserError{
            Field:   []string{"password"},
            Message: "Password must be at least 8 characters",
        })
    }

    return errs
}
```

## Using Validation in Mutations

```go
func (r *mutationResolver) CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserPayload, error) {
    // Validate first
    if errs := validateCreateUserInput(input); len(errs) > 0 {
        return &CreateUserPayload{Errors: errs}, nil
    }

    // Create user...
    return &CreateUserPayload{User: user}, nil
}
```

## Validation Helper Functions

```go
func isValidEmail(email string) bool {
    rx := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    return rx.MatchString(email)
}

func isValidURL(url string) bool {
    u, err := url.Parse(url)
    return err == nil && u.Scheme != "" && u.Host != ""
}

func validateLength(field, value string, min, max int) *UserError {
    if value == "" {
        return &UserError{
            Field:   []string{field},
            Message: fmt.Sprintf("%s is required", field),
        }
    }
    if len(value) < min {
        return &UserError{
            Field:   []string{field},
            Message: fmt.Sprintf("%s must be at least %d characters", field, min),
        }
    }
    if max > 0 && len(value) > max {
        return &UserError{
            Field:   []string{field},
            Message: fmt.Sprintf("%s must be less than %d characters", field, max),
        }
    }
    return nil
}
```

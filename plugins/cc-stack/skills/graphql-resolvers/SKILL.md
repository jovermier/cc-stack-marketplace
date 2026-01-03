---
name: graphql-resolvers
description: GraphQL resolver patterns including dataloader for N+1 prevention, context propagation, authorization, error handling, and validation. Use when implementing GraphQL resolvers.
---

# GraphQL Resolvers

Expert guidance for implementing efficient, secure GraphQL resolvers.

## Quick Reference

| Concern | Solution | Pattern |
|---------|----------|---------|
| N+1 queries | Dataloader | Batch load relations |
| Authentication | Context middleware | Check before resolving |
| Authorization | Field-level checks | User can access this data |
| Validation | Schema layer | Input validation before resolvers |
| Error handling | Wrapped errors | Don't expose internal details |
| Context propagation | Pass through all levels | context.Context to nested resolvers |

## What Do You Need?

1. **Dataloader** - Batching relations to prevent N+1 queries
2. **Authorization** - Checking access at field level
3. **Error handling** - Proper GraphQL errors, no internal exposure
4. **Context** - Propagating user, request-scoped data
5. **Validation** - Schema-level validation approach

Specify a number or describe your resolver scenario.

## Routing

| Response | Reference to Read |
|----------|-------------------|
| 1, "dataloader", "n+1", "batch", "relation" | [dataloader.md](./references/dataloader.md) |
| 2, "auth", "authorization", "access", "permission" | [authorization.md](./references/authorization.md) |
| 3, "error", "wrapped", "internal" | [errors.md](./references/errors.md) |
| 4, "context", "user", "request" | [context.md](./references/context.md) |
| 5, "validation", "input", "schema" | [validation.md](./references/validation.md) |

## Critical Rules

- **Always use dataloader for relations**: Prevents N+1 queries
- **Authorize at resolver level**: Check user can access the data
- **Never expose internal errors**: Wrap before returning
- **Propagate context through resolver chain**: All nested resolvers need it
- **Validate at schema layer**: Use input validation, not in resolvers
- **No circular dependencies**: Be aware of resolver chains

## Dataloader Pattern

```go
// Bad: N+1 query pattern
func (r *queryResolver) Users(ctx context.Context) ([]*User, error) {
    users, _ := r.db.Users()  // 1 query
    for _, user := range users {
        posts, _ := r.db.PostsByUser(user.ID)  // N queries!
        user.Posts = posts
    }
    return users, nil
}

// Good: Using dataloader
func (r *queryResolver) Users(ctx context.Context) ([]*User, error) {
    users, err := r.db.Users()
    if err != nil {
        return nil, err
    }

    // Batch load posts using dataloader
    loaders := dataloader.For(ctx)
    for _, user := range users {
        user.Posts, err = loaders.PostsByUser.Load(user.ID)
        if err != nil {
            return nil, err
        }
    }

    return users, nil
}
```

## Authorization Pattern

```go
// Good: Authorization check in resolver
func (r *queryResolver) User(ctx context.Context, id string) (*User, error) {
    // Check authentication
    viewer := auth.FromContext(ctx)
    if viewer == nil {
        return nil, fmt.Errorf("authentication required")
    }

    // Fetch user
    user, err := r.db.FindUser(id)
    if err != nil {
        return nil, err
    }

    // Check authorization (users can view own profile, admins can view any)
    if user.ID != viewer.ID && !viewer.IsAdmin {
        return nil, fmt.Errorf("access denied")
    }

    return user, nil
}
```

## Error Handling Pattern

```go
// Bad: Exposing internal errors
func (r *mutationResolver) CreateUser(ctx context.Context, input CreateUserInput) (*CreateUserPayload, error) {
    if err := r.db.CreateUser(input); err != nil {
        return nil, fmt.Errorf("database error: %v", err)  // Leaks DB details!
    }
    // ...
}

// Good: Wrapped errors
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
        return nil, fmt.Errorf("failed to create user")
    }
    // ...
}
```

## Common Resolver Issues

| Issue | Severity | Impact | Fix |
|-------|----------|--------|-----|
| N+1 queries | Critical | Database overload, slow | Use dataloader |
| Missing authorization | Critical | Data exposure | Add auth checks |
| Exposing internal errors | High | Information disclosure | Wrap errors |
| Not propagating context | High | Breaks auth, timeout | Pass ctx through |
| No validation | Medium | Bad data in DB | Validate at schema |
| Circular resolver dependencies | High | Infinite loops | Restructure schema |

## Reference Index

| File | Topics |
|------|--------|
| [dataloader.md](./references/dataloader.md) | Batching, caching, implementation |
| [authorization.md](./references/authorization.md) | Auth checks, role-based access |
| [errors.md](./references/errors.md) | Error wrapping, field errors |
| [context.md](./references/context.md) | Propagation, request-scoped data |
| [validation.md](./references/validation.md) | Schema validation, input types |

## Success Criteria

Resolvers are correct when:
- Dataloader used for all relations (no N+1 queries)
- Authorization checked before data access
- Internal errors wrapped, not exposed
- Context propagated through resolver chain
- Validation happens at schema layer
- No circular dependencies in resolver chains
- Field-level authorization for sensitive data

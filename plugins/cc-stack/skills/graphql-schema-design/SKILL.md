---
name: graphql-schema-design
description: GraphQL schema design including types, fields, pagination, nullability, naming conventions, and descriptions. Use when designing or modifying GraphQL schemas.
---

# GraphQL Schema Design

Expert guidance for designing well-structured GraphQL schemas.

## Quick Reference

| Concept | Best Practice | Example |
|---------|---------------|---------|
| Nullability | Default nullable, required only when necessary | `email: String` not `email: String!` |
| Pagination | Relay connections (edges/nodes) | `users(first: Int, after: String): UserConnection!` |
| Naming | PascalCase types, camelCase fields | `type UserProfile { firstName: String }` |
| Descriptions | All types, fields, arguments | `"""User account"""` |
| Mutations | Noun + Verb pattern | `createUser`, `deletePost` |
| Deprecations | @deprecated with reason | `@deprecated(reason: "Use newField")` |

## What Do You Need?

1. **Type design** - Structs, interfaces, unions, enums
2. **Field design** - Nullability, arguments, defaults
3. **Pagination** - Relay-style connections
4. **Naming** - Conventions for consistency
5. **Documentation** - Descriptions, deprecations

Specify a number or describe your schema concern.

## Routing

| Response | Reference to Read |
|----------|-------------------|
| 1, "type", "interface", "union", "enum" | [types.md](./references/types.md) |
| 2, "field", "argument", "nullable", "default" | [fields.md](./references/fields.md) |
| 3, "pagination", "connection", "relay" | [pagination.md](./references/pagination.md) |
| 4, "naming", "convention", "consistency" | [naming.md](./references/naming.md) |
| 5, "description", "deprecation", "document" | [documentation.md](./references/documentation.md) |

## Critical Rules

- **Default to nullable**: Easier to make required later than vice versa
- **Use Relay pagination**: Connections with edges/nodes, not lists
- **Document everything**: Schema is the API documentation
- **Deprecate before removing**: @deprecated with reason, wait for clients to migrate
- **Noun mutations for state changes**: createUser, deletePost, closeCard
- **Avoid business logic in schema**: Schema describes shape, not behavior

## Schema Template

```graphql
"""
A user in the system
"""
type User {
    """
    The unique identifier of the user
    """
    id: ID!

    """
    The user's display name
    """
    name: String!

    """
    The user's email address (optional if not public)
    """
    email: String

    """
    Posts created by this user, paginated
    """
    posts(
        """
        Number of posts to return
        """
        first: Int
        """
        Cursor for pagination
        """
        after: String
    ): PostConnection!
}

"""
Paginated connection of posts
"""
type PostConnection {
    edges: [PostEdge!]!
    pageInfo: PageInfo!
    totalCount: Int!
}

type PostEdge {
    node: Post!
    cursor: String!
}

type PageInfo {
    hasNextPage: Boolean!
    hasPreviousPage: Boolean!
    startCursor: String
    endCursor: String
}
```

## Common Schema Issues

| Issue | Severity | Fix |
|-------|----------|-----|
| Unbounded lists | High | Use pagination connections |
| Missing descriptions | Medium | Add doc comments |
| Inconsistent nullability | Medium | Be intentional about ! |
| Breaking changes without deprecation | High | Use @deprecated first |
| CRUD-style mutations | Low | Use noun+verb (createUser) |
| No pagination on collections | High | Add Relay connections |

## Nullability Guidelines

```graphql
# Good: Nullable by default
type User {
    id: ID!
    name: String!          # Required for user
    email: String          # Optional (not all users have email)
    bio: String            # Optional (not all users filled it out)
    posts(first: Int): PostConnection!  # Connection required, edges may be empty
}

# Avoid: Too many required fields
type User {
    id: ID!
    name: String!
    email: String!         # Required may block mutations
    phone: String!         # Required may block mutations
    bio: String!           # Required may block mutations
}
```

## Reference Index

| File | Topics |
|------|--------|
| [types.md](./references/types.md) | Objects, interfaces, unions, enums, scalars |
| [fields.md](./references/fields.md) | Nullability, arguments, defaults, lists |
| [pagination.md](./references/pagination.md) | Relay connections, edges, nodes, cursors |
| [naming.md](./references/naming.md) | Conventions for types, fields, mutations |
| [documentation.md](./references/documentation.md) | Descriptions, deprecations, comments |

## Success Criteria

Schema is well-designed when:
- All types and fields have descriptions
- Collections use Relay pagination (not unbounded lists)
- Nullability is intentional (not default required)
- Naming follows conventions (PascalCase, camelCase)
- Deprecated fields have @deprecated with reason
- No breaking changes without deprecation period
- Schema reads as documentation for clients

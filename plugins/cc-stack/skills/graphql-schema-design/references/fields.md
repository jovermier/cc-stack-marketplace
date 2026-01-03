# GraphQL Fields

Designing fields, arguments, and nullability.

## Nullability Guidelines

```graphql
# Default to nullable
type User {
    id: ID!
    name: String!
    email: String      # Optional
    bio: String        # May not be set
    posts: [Post!]!    # Connection required, edges may be empty
}

# Avoid too many required fields
type BadExample {
    id: ID!
    name: String!
    email: String!     # Required may block mutations
    phone: String!     # Required may block mutations
}
```

## Field Arguments

```graphql
type Query {
    """
    Get users with pagination
    """
    users(
        """
        Number of results to return (max: 100)
        """
        first: Int = 10

        """
        Cursor for pagination
        """
        after: String

        """
        Filter by role
        """
        role: UserRole
    ): UserConnection!
}
```

## Lists vs Connections

```graphql
# ❌ Bad: Unbounded list
type Query {
    allUsers: [User!]!  # Will return millions eventually
}

# ✅ Good: Paginated connection
type Query {
    users(first: Int, after: String): UserConnection!
}
```

## Default Values

```graphql
type Query {
    posts(
        limit: Int = 10    # Default to 10
        sort: SortOrder = DESC  # Default sort
    ): PostConnection!
}
```

## Field Descriptions

```graphql
type User {
    """
    The user's unique identifier (UUID format)
    """
    id: ID!

    """
    Display name shown in UI
    """
    name: String!
}
```

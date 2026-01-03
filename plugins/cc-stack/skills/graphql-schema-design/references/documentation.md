# Schema Documentation

Documenting GraphQL schemas.

## Type Descriptions

```graphql
"""
A user account in the system.

Users can create posts, comment, and interact with content.
"""
type User {
    """
    Unique identifier (UUID v4)

    This ID is stable and should be used for caching keys.
    """
    id: ID!

    """
    Display name shown in UI

    Max 100 characters. Can be changed by user.
    """
    name: String!
}
```

## Field Descriptions

```graphql
type Query {
    """
    Get the currently authenticated user

    Returns null if not authenticated. Use this for
    personalized views and profile management.

    Example: `query { viewer { id name } }`
    """
    viewer: User

    """
    Get a user by ID

    Requires authentication. Users can view their own profile
    and profiles of users in same organization.

    Example: `query { user(id: "123") { id name } }`
    """
    user(id: ID!): User
}
```

## Argument Descriptions

```graphql
type Query {
    users(
        """
        Number of users to return (max: 100)

        Default: 10. Use pagination for large datasets.
        """
        first: Int

        """
        Cursor for pagination

        Get this from the `pageInfo.endCursor` of previous query.
        """
        after: String
    ): UserConnection!
}
```

## Enum Value Descriptions

```graphql
enum UserRole {
    """
    Standard user with basic permissions

    Can view and create own content.
    """
    USER

    """
    Administrator with full system access

    Can manage users, settings, and all content.
    """
    ADMIN
}
```

## Deprecation

```graphql
type User {
    """
    User's full name

    @deprecated(reason: "Use firstName and lastName instead")
    """
    name: String! @deprecated(reason: "Use firstName and lastName instead")

    """
    First name
    """
    firstName: String!

    """
    Last name
    """
    lastName: String!
}
```

## Breaking Changes Documentation

When making breaking changes:

1. Add `@deprecated` with reason to old field
2. Keep old field for at least one version
3. Document migration guide in description
4. Remove only after deprecation period

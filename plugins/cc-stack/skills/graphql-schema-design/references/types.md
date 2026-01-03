# GraphQL Types

Designing GraphQL types and type system.

## Object Types

```graphql
"""
A user in the system
"""
type User {
    """
    Unique identifier
    """
    id: ID!

    """
    Display name (always available)
    """
    name: String!

    """
    Optional email (not all users have email)
    """
    email: String
}
```

## Interfaces

```graphql
"""
A node in the system (for Relay pagination)
"""
interface Node {
    """
    Unique identifier for Relay
    """
    id: ID!
}

type User implements Node {
    id: ID!
    name: String!
}

type Post implements Node {
    id: ID!
    title: String!
}
```

## Unions

```graphql
"""
Search result can be different types
"""
union SearchResult = User | Post | Comment

type Query {
    search(query: String!): [SearchResult!]!
}
```

## Enums

```graphql
"""
User role in the system
"""
enum UserRole {
    """
    Standard user with basic permissions
    """
    USER

    """
    Administrator with full access
    """
    ADMIN
}
```

## Scalars

```graphql
# Built-in scalars
ID        # Unique identifier
String
Int
Float
Boolean

# Custom scalars (define in code)
scalar Date
scalar JSON
scalar Upload
```

## Input Types

```graphql
# For complex inputs
input CreateUserInput {
    name: String!
    email: String
    password: String!
}
```

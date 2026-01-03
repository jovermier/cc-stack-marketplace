# Naming Conventions

Consistent naming for GraphQL schemas.

## Type Names (PascalCase)

```graphql
# ✅ Good: PascalCase
type UserProfile { }
type BlogPost { }
type HTTPResponse { }

# ❌ Bad: Other casing
type user_profile { }
type blog_post { }
```

## Field Names (camelCase)

```graphql
# ✅ Good: camelCase
type User {
    firstName: String!
    lastName: String!
    emailAddress: String
    createdAt: String!
}

# ❌ Bad: snake_case
type User {
    first_name: String!
    last_name: String!
}
```

## Mutation Names (Noun + Verb)

```graphql
# ✅ Good: noun + verb pattern
type Mutation {
    createUser: CreateUserPayload!
    deleteUser: DeleteUserPayload!
    updateUser: UpdateUserPayload!
    closeCard: CloseCardPayload!
}

# ❌ Bad: verb + noun (less consistent)
type Mutation {
    userCreate: UserCreatePayload!
    deleteUser: DeleteUserPayload!
}
```

## Query Names (Noun, plural for lists)

```graphql
# ✅ Good
type Query {
    user: User
    users: [User!]!
    post: Post
    posts: [Post!]!
}

# ✅ Good: Specific prefixes
type Query {
    viewer: User         # Current user
    node: Node           # By ID
    search: [SearchResult!]
}
```

## Argument Names (camelCase)

```graphql
type Query {
    users(
        first: Int
        after: String
        sortBy: SortOrder
    ): UserConnection!
}
```

## Input Types (PascalCase with "Input" suffix)

```graphql
# ✅ Good
input CreateUserInput {
    name: String!
    email: String!
}

input UpdateUserProfileInput {
    firstName: String
    lastName: String
}
```

## Enum Values (UPPER_SNAKE_CASE)

```graphql
enum UserRole {
    STANDARD_USER
    ADMINISTRATOR
    GUEST
}
```

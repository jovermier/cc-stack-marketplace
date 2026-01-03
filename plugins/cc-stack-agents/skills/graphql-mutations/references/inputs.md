# Mutation Inputs

Structuring mutation input objects.

## Input Object Pattern

```graphql
# ✅ Good: Single input object
type Mutation {
    createUser(input: CreateUserInput!): CreateUserPayload!
}

input CreateUserInput {
    name: String!
    email: String!
    password: String!
}

# ❌ Bad: Multiple scalar arguments
type Mutation {
    createUser(name: String!, email: String!, password: String!): User!
}
```

## Nested Input Objects

```graphql
type Mutation {
    updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!
}

input UpdateUserInput {
    """
    Personal information
    """
    profile: UserProfileInput

    """
    Email preferences
    """
    emails: EmailPreferencesInput
}

input UserProfileInput {
    firstName: String
    lastName: String
    bio: String
}

input EmailPreferencesInput {
    newsletter: Boolean
    notifications: Boolean
}
```

## Array Input

```graphql
type Mutation {
    addTagsToPost(postId: ID!, tags: [String!]!): AddTagsPayload!
}

input AddTagsInput {
    postId: ID!
    tags: [String!]!
}
```

## Complex Input with Validation

```graphql
input CreatePostInput {
    """
    Post title (required, 5-200 characters)
    """
    title: String!

    """
    Post content (markdown supported)
    """
    content: String!

    """
    Post tags (max 10)
    """
    tags: [String!]

    """
    Publishing options
    """
    publish: PublishOptionsInput
}

input PublishOptionsInput {
    """
    Publish immediately or schedule
    """
    mode: PublishMode = IMMEDIATE

    """
    Scheduled publish date (required if mode is SCHEDULED)
    """
    scheduledAt: DateTime
}

enum PublishMode {
    DRAFT
    IMMEDIATE
    SCHEDULED
}
```

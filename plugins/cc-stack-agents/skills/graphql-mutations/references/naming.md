# Mutation Naming

Conventions for naming GraphQL mutations.

## Noun + Verb Pattern

```graphql
# ✅ Good: noun + verb
type Mutation {
    createUser: CreateUserPayload!
    deleteUser: DeleteUserPayload!
    updateUser: UpdateUserPayload!
    closeCard: CloseCardPayload!
    publishPost: PublishPostPayload!
}

# ❌ Less consistent: verb + noun
type Mutation {
    userCreate: UserCreatePayload!
    userDelete: UserDeletePayload!
}
```

## Naming Guidelines

| Action | Naming | Example |
|--------|--------|---------|
| Create | `create{Resource}` | `createUser`, `createPost` |
| Update | `update{Resource}` | `updateUser`, `updatePost` |
| Delete | `delete{Resource}` | `deleteUser`, `deletePost` |
| State change | `{verb}{Resource}` | `closeCard`, `publishPost`, `archiveUser` |
| Add/Remove | `add{Item}To{Collection}` | `addTagToPost`, `removeMemberFromTeam` |

## State Change Mutations

```graphql
# State change: use descriptive verb + noun
type Mutation {
    """
    Close a card (marks as closed, not deleted)
    """
    closeCard(id: ID!): CloseCardPayload!

    """
    Archive a user (soft delete, inactive)
    """
    archiveUser(id: ID!): ArchiveUserPayload!

    """
    Publish a post (make visible)
    """
    publishPost(id: ID!): PublishPostPayload!

    """
    Unpublish a post (make private)
    """
    unpublishPost(id: ID!): UnpublishPostPayload!
}
```

## Add/Remove Pattern

```graphql
type Mutation {
    """
    Add a tag to a post
    """
    addTagToPost(postId: ID!, tagId: ID!): AddTagToPostPayload!

    """
    Remove a tag from a post
    """
    removeTagFromPost(postId: ID!, tagId: ID!): RemoveTagFromPostPayload!

    """
    Add a member to a team
    """
    addMemberToTeam(teamId: ID!, userId: ID!): AddMemberToTeamPayload!
}
```

## Bulk Operations

```graphql
# Singular for single, plural for bulk
type Mutation {
    createUser(input: CreateUserInput!): CreateUserPayload!
    createUsers(inputs: [CreateUserInput!]!): CreateUsersPayload!

    deleteUser(id: ID!): DeleteUserPayload!
    deleteUsers(ids: [ID!]!): DeleteUsersPayload!
}
```

## Boolean Returns (Avoid)

```graphql
# ❌ Avoid: Boolean returns don't give details
type Mutation {
    deleteUser(id: ID!): Boolean
}

# ✅ Good: Payload gives details
type Mutation {
    deleteUser(id: ID!): DeleteUserPayload!
}

type DeleteUserPayload {
    deletedUserId: ID
    errors: [UserError!]!
}
```

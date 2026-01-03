# Pagination

Relay-style connection pagination.

## Connection Pattern

```graphql
type Query {
    users(first: Int, after: String): UserConnection!
}

type UserConnection {
    edges: [UserEdge!]!
    pageInfo: PageInfo!
    totalCount: Int!
}

type UserEdge {
    node: User!
    cursor: String!
}

type PageInfo {
    hasNextPage: Boolean!
    hasPreviousPage: Boolean!
    startCursor: String
    endCursor: String
}
```

## Cursor Design

```go
// Encode cursor (typically base64 of ID)
func EncodeCursor(id string) string {
    return base64.StdEncoding.EncodeToString([]byte(id))
}

// Decode cursor
func DecodeCursor(cursor string) (string, error) {
    decoded, err := base64.StdEncoding.DecodeString(cursor)
    return string(decoded), err
}
```

## Resolver Implementation

```go
func (r *queryResolver) Users(ctx context.Context, first *int, after *string) (*UserConnection, error) {
    limit := 10
    if first != nil {
        limit = *first
    }

    var lastID string
    if after != nil {
        lastID = decodeCursor(*after)
    }

    users, err := r.db.UsersAfter(lastID, limit+1) // Fetch one extra
    if err != nil {
        return nil, err
    }

    edges := make([]*UserEdge, 0, limit)
    for i, user := range users {
        if i >= limit {
            break  // Don't include extra in edges
        }
        edges = append(edges, &UserEdge{
            Node:   user,
            Cursor: encodeCursor(user.ID),
        })
    }

    hasNextPage := len(users) > limit
    var startCursor, endCursor *string
    if len(edges) > 0 {
        startCursor = &edges[0].Cursor
        endCursor = &edges[len(edges)-1].Cursor
    }

    return &UserConnection{
        Edges:      edges,
        PageInfo: &PageInfo{
            HasNextPage: hasNextPage,
            StartCursor: startCursor,
            EndCursor:   endCursor,
        },
    }, nil
}
```

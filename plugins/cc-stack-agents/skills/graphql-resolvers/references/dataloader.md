# Dataloader

Batching and caching to prevent N+1 queries.

## N+1 Problem

```go
// ❌ Bad: N+1 queries
func (r *queryResolver) Users(ctx context.Context) ([]*User, error) {
    users, _ := r.db.Users()  // 1 query
    for _, user := range users {
        posts, _ := r.db.PostsByUser(user.ID)  // N queries!
        user.Posts = posts
    }
    return users, nil
}
```

## Dataloader Solution

```go
// Create dataloader
func NewPostLoader(db *DB) *dataloader.Loader {
    return dataloader.NewBatchedLoader(
        func(ctx context.Context, keys dataloader.Keys) []*dataloader.Result {
            var results []*dataloader.Result

            // Single batch query
            posts, err := db.PostsByUserIDs(keys.Keys())
            if err != nil {
                results = make([]*dataloader.Result, len(keys))
                for i := range results {
                    results[i] = &dataloader.Result{Error: err}
                }
                return results
            }

            // Map results
            postMap := make(map[string][]*Post)
            for _, post := range posts {
                postMap[post.UserID] = append(postMap[post.UserID], post)
            }

            // Return in same order as keys
            results = make([]*dataloader.Result, len(keys))
            for i, key := range keys.Keys() {
                results[i] = &dataloader.Result{Data: postMap[key.String()]}
            }

            return results
        },
        dataloader.WithBatch(100),
        dataloader.WithCache(dataloader.NewCache()),
    )
}
```

## Using Dataloader

```go
// ✅ Good: Using dataloader
func (r *queryResolver) Users(ctx context.Context) ([]*User, error) {
    users, err := r.db.Users()
    if err != nil {
        return nil, err
    }

    // Get dataloader from context
    loaders := dataloader.For(ctx)
    for _, user := range users {
        // Batch loaded automatically
        user.Posts, err = loaders.PostsByUser.Load(ctx, user.ID)()
        if err != nil {
            return nil, err
        }
    }

    return users, nil
}
```

## Context Setup

```go
type Loaders struct {
    PostsByUser *dataloader.Loader
}

func Middleware(db *DB) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            loaders := &Loaders{
                PostsByUser: NewPostLoader(db),
            }
            ctx := context.WithValue(r.Context(), "loaders", loaders)
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}

func For(ctx context.Context) *Loaders {
    return ctx.Value("loaders").(*Loaders)
}
```

# Mocking

Strategies for testing with external dependencies.

## Prefer Fakes Over Mocks

```go
// Fake: In-memory implementation
type FakeRepository struct {
    users map[string]*User
    mu    sync.Mutex
}

func (f *FakeRepository) FindUser(id string) (*User, error) {
    f.mu.Lock()
    defer f.mu.Unlock()
    return f.users[id], nil
}

func (f *FakeRepository) CreateUser(user *User) error {
    f.mu.Lock()
    defer f.mu.Unlock()
    f.users[user.ID] = user
    return nil
}

// Use in tests
func TestService(t *testing.T) {
    repo := &FakeRepository{users: make(map[string]*User)}
    service := NewService(repo)

    user, _ := service.CreateUser("test@example.com")
    // ...
}
```

## Interface-Based Design

```go
// Define interface for dependency
type UserRepository interface {
    FindUser(id string) (*User, error)
    CreateUser(user *User) error
}

// Service depends on interface
type Service struct {
    repo UserRepository
}

// In tests, use fake
type FakeRepo struct { ... }
func (f *FakeRepo) FindUser(id string) (*User, error) { ... }

func TestService(t *testing.T) {
    fake := &FakeRepo{...}
    service := NewService(fake)
    // ...
}
```

## httptest for HTTP Handlers

```go
func TestHandler(t *testing.T) {
    tests := []struct {
        method       string
        body         string
        wantStatus   int
    }{
        {"POST", `{"name":"test"}`, 200},
        {"POST", `{`, 400},  // Invalid JSON
    }

    for _, tt := range tests {
        t.Run(tt.method, func(t *testing.T) {
            req := httptest.NewRequest(tt.method, "/test", strings.NewReader(tt.body))
            rec := httptest.NewRecorder()

            Handler(rec, req)

            if rec.Code != tt.wantStatus {
                t.Errorf("status = %d, want %d", rec.Code, tt.wantStatus)
            }
        })
    }
}
```

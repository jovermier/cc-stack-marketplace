# Table-Driven Tests

Go's standard pattern for testing multiple cases.

## Basic Template

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 1, 2, 3},
        {"negative", -1, 1, 0},
        {"zeros", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := Add(tt.a, tt.b); got != tt.expected {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.expected)
            }
        })
    }
}
```

## With Error Cases

```go
func TestValidate(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        wantErr bool
        errIs   error
    }{
        {"valid email", "user@example.com", false, nil},
        {"empty", "", true, ErrEmpty},
        {"invalid", "not-an-email", true, ErrInvalidFormat},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := Validate(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("Validate() error = %v, wantErr %v", err, tt.wantErr)
            }
            if tt.errIs != nil && !errors.Is(err, tt.errIs) {
                t.Errorf("Validate() error = %v, wantIs %v", err, tt.errIs)
            }
        })
    }
}
```

## Key Points

- Use struct to define test case fields
- Name each test case for clear output
- Use t.Run() for subtests (can run individual cases)
- Test both success and failure paths

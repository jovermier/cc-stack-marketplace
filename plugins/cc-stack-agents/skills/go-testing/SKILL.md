---
name: go-testing
description: Go testing best practices including table-driven tests, race detection, test coverage, and mocking strategies. Use when writing or reviewing Go tests.
---

# Go Testing

Expert guidance for writing maintainable, effective Go tests.

## Quick Reference

| Pattern | When to Use | Structure |
|---------|-------------|-----------|
| Table-driven tests | Multiple inputs/outputs | []struct with test cases |
| Subtests | Related test variants | t.Run() for each case |
| TestMain | Global setup/teardown | func TestMain(m *testing.M) |
| t.Cleanup | Per-test cleanup | Deferred cleanup function |
| fakes/fuzzing | Random input testing | testing.F, f.Fuzz() |
| Race detector | Concurrent code | go test -race |
| Coverage | Ensuring thoroughness | go test -cover |

## What Do You Need?

1. **Test structure** - Table-driven, subtests, organization
2. **Mocking** - Fakes, interfaces, test doubles
3. **Concurrency testing** - Race detector, parallel tests
4. **Coverage** - Measuring and improving test coverage
5. **Test data** - Fixtures, golden files, test helpers

Specify a number or describe your testing scenario.

## Routing

| Response | Reference to Read |
|----------|-------------------|
| 1, "table", "driven", "multiple cases" | [table-driven.md](./references/table-driven.md) |
| 2, "mock", "fake", "interface" | [mocking.md](./references/mocking.md) |
| 3, "race", "concurrent", "parallel" | [concurrency.md](./references/concurrency.md) |
| 4, "coverage", "measure", "thorough" | [coverage.md](./references/coverage.md) |
| 5, general testing | Read relevant references |

## Critical Rules

- **Table-driven for variations**: Use for multiple inputs/outputs
- **Descriptive test names**: TestFunctionName_State format
- **t.Cleanup for cleanup**: Prefer over defer in tests
- **Run with -race**: Must pass for concurrent code
- **Avoid mocking when possible**: Use real implementations or fakes
- **Tests should fail for the right reason**: Not due to flakiness

## Test Template

```go
func TestFunctionName(t *testing.T) {
    tests := []struct {
        name    string
        input   InputType
        want    WantType
        wantErr bool
        errIs   error
    }{
        {
            name:    "successful case",
            input:   InputType{...},
            want:    WantType{...},
            wantErr: false,
        },
        {
            name:    "validation error",
            input:   InputType{...},
            wantErr: true,
            errIs:   ErrValidation,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := FunctionName(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("FunctionName() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if tt.errIs != nil && !errors.Is(err, tt.errIs) {
                t.Errorf("FunctionName() error = %v, wantIs %v", err, tt.errIs)
            }
            if !reflect.DeepEqual(got, tt.want) {
                t.Errorf("FunctionName() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

## Test Organization

```
project/
├── internal/
│   ├── service/
│   │   ├── service.go
│   │   ├── service_test.go
│   │   └── service_golden_test.go
│   └── service/
│       ├── mocks/          # Generated mocks (if needed)
│       └── testdata/       # Golden files, fixtures
└── testutil/
    ├── setup.go           # Test helpers
    └── fixtures.go        # Shared test data
```

## Common Testing Patterns

### HTTP Handlers
```go
func TestHandler(t *testing.T) {
    tests := []struct {
        name       string
        method     string
        body       string
        wantStatus int
        wantBody   string
    }{
        {"valid POST", "POST", `{"foo":"bar"}`, 200, `{"result":"ok"}`},
        {"invalid JSON", "POST", `{`, 400, ""},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
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

### Using t.Cleanup
```go
func TestWithCleanup(t *testing.T) {
    // Setup
    db := openTestDB(t)
    t.Cleanup(func() {
        db.Close()  // Runs even if test fails
    })

    // Test code...
}
```

### Race Detection
```bash
# Run tests with race detector
go test -race ./...

# Run specific test with race detector
go test -race -run TestConcurrentFunction
```

## Reference Index

| File | Topics |
|------|--------|
| [table-driven.md](./references/table-driven.md) | Table structure, subtests, naming |
| [mocking.md](./references/mocking.md) | Interfaces, fakes, mocking libraries |
| [concurrency.md](./references/concurrency.md) | Race detector, parallel tests, sync |
| [coverage.md](./references/coverage.md) | -cover, -coverprofile, thresholds |

## Success Criteria

Tests are good when:
- Table-driven tests cover variations
- Race detector passes (-race)
- Coverage is meaningful (not just high numbers)
- Tests are readable and maintainable
- t.Cleanup used for resource cleanup
- Test failures are clear about what went wrong

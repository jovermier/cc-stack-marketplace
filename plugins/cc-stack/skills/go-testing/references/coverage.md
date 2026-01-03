# Test Coverage

Measuring and improving test coverage in Go.

## Running Coverage

```bash
# Show coverage percentage
go test -cover ./...

# Generate coverage profile
go test -coverprofile=coverage.out ./...

# View coverage in browser
go tool cover -html=coverage.out

# Show which lines are not covered
go tool cover -func=coverage.out
```

## Coverage Thresholds

```bash
# Check if coverage meets threshold
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out | grep total

# Example output: total: statements 85.7%
```

## Coverage by Package

```bash
# Coverage for specific package
go test -coverprofile=coverage.out ./path/to/package

# Combine multiple packages
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
```

## What Coverage Doesn't Measure

- **Edge cases**: Coverage doesn't mean all scenarios tested
- **Integration**: Unit coverage doesn't guarantee system works
- **Race conditions**: Race detector needed (-race)
- **Error paths**: Explicitly test error cases

## Best Practices

- Aim for meaningful coverage, not just high numbers
- Focus on critical paths and error cases
- Use table-driven tests for variations
- Test both success and failure paths
- Use -race for concurrent code

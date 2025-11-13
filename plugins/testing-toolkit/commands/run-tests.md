---
description: Execute tests with appropriate configuration and flags based on project type
allowed-tools: Bash(npm:*), Bash(yarn:*), Bash(pnpm:*), Bash(pytest:*), Bash(go:*), Read
argument-hint: [test-type] [--flags]
---

# Run Tests

## Context

- Project type: !`[ -f "package.json" ] && echo "Node.js" || [ -f "pyproject.toml" ] && echo "Python" || [ -f "go.mod" ] && echo "Go" || echo "Unknown"`
- Test framework: !`[ -f "package.json" ] && (grep -q "jest" package.json && echo "Jest" || grep -q "vitest" package.json && echo "Vitest" || echo "Unknown") || [ -f "pyproject.toml" ] && echo "pytest" || echo "Unknown"`

## Your Task

Execute tests with appropriate configuration based on the project type and test framework.

### Arguments

- `$1` (optional): Test type - `unit`, `integration`, `e2e`, or `all` (default: `all`)
- `$2` (optional): Additional flags (e.g., `--watch`, `--coverage`, `--verbose`)

### Steps

1. **Detect project type and test framework** from context above

2. **Determine test command** based on framework:

   **Jest**:
   ```bash
   npm test
   # or with flags
   npm test -- --coverage --verbose
   ```

   **Vitest**:
   ```bash
   npm run test
   # or
   vitest run
   ```

   **pytest**:
   ```bash
   pytest
   # or with flags
   pytest -v --cov=src
   ```

   **Go**:
   ```bash
   go test ./...
   # or with flags
   go test -v -cover ./...
   ```

3. **Apply test type filter** if specified:

   **Jest/Vitest** (unit tests):
   ```bash
   npm test -- --testPathPattern=unit
   ```

   **Jest/Vitest** (integration tests):
   ```bash
   npm test -- --testPathPattern=integration
   ```

   **pytest** (unit tests):
   ```bash
   pytest tests/unit/
   ```

   **pytest** (integration tests):
   ```bash
   pytest tests/integration/
   ```

   **Go** (specific package):
   ```bash
   go test ./pkg/...
   ```

4. **Execute the test command** with appropriate flags

5. **Report results**:
   - Number of tests passed/failed
   - Execution time
   - Coverage percentage (if --coverage flag used)
   - Any failing tests with error messages

### Examples

**Run all tests**:
```
/run-tests
```

**Run unit tests only**:
```
/run-tests unit
```

**Run tests with coverage**:
```
/run-tests all --coverage
```

**Run tests in watch mode**:
```
/run-tests --watch
```

**Run specific test file**:
```
/run-tests integration --testPathPattern=api
```

# Coverage Tools Reference

## Contents
- Jest Configuration
- Vitest Configuration
- pytest Configuration
- Go Coverage
- Coverage Report Formats
- CI/CD Integration

## Jest Configuration

### Basic Setup

```json
// package.json
{
  "jest": {
    "collectCoverage": true,
    "coverageDirectory": "coverage",
    "coverageReporters": ["text", "lcov", "html"],
    "collectCoverageFrom": [
      "src/**/*.{js,ts,jsx,tsx}",
      "!src/**/*.d.ts",
      "!src/**/*.test.{js,ts,jsx,tsx}",
      "!src/**/__tests__/**"
    ]
  }
}
```

### Coverage Thresholds

```json
{
  "jest": {
    "coverageThreshold": {
      "global": {
        "branches": 70,
        "functions": 70,
        "lines": 70,
        "statements": 70
      },
      "./src/core/**/*.js": {
        "branches": 90,
        "functions": 90,
        "lines": 90,
        "statements": 90
      }
    }
  }
}
```

### Ignore Patterns

```json
{
  "jest": {
    "coveragePathIgnorePatterns": [
      "/node_modules/",
      "/dist/",
      "/build/",
      "/.next/",
      "/coverage/"
    ]
  }
}
```

## Vitest Configuration

### Basic Setup

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8', // or 'istanbul'
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'dist/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/mockData/**'
      ]
    }
  }
})
```

### Coverage Thresholds

```typescript
export default defineConfig({
  test: {
    coverage: {
      thresholds: {
        lines: 70,
        functions: 70,
        branches: 70,
        statements: 70
      }
    }
  }
})
```

## pytest Configuration

### Basic Setup

```ini
# pytest.ini or setup.cfg
[tool:pytest]
addopts = 
    --cov=src
    --cov-report=html
    --cov-report=term-missing
    --cov-report=xml
    --cov-fail-under=70
```

### pyproject.toml Configuration

```toml
[tool.pytest.ini_options]
addopts = [
    "--cov=src",
    "--cov-report=html",
    "--cov-report=term-missing",
    "--cov-fail-under=70"
]

[tool.coverage.run]
source = ["src"]
omit = [
    "*/tests/*",
    "*/test_*.py",
    "*/__pycache__/*",
    "*/venv/*"
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
    "if TYPE_CHECKING:",
    "@abstractmethod"
]
```

### Branch Coverage

```bash
pytest --cov=src --cov-branch --cov-report=term-missing
```

## Go Coverage

### Basic Usage

```bash
# Run tests with coverage
go test -cover ./...

# Generate coverage profile
go test -coverprofile=coverage.out ./...

# View coverage in browser
go tool cover -html=coverage.out

# View coverage by function
go tool cover -func=coverage.out
```

### Coverage Modes

```bash
# Set coverage mode (default: set)
go test -covermode=set ./...    # Was line executed?
go test -covermode=count ./...  # How many times?
go test -covermode=atomic ./... # Like count, but atomic
```

### Package-specific Coverage

```bash
# Single package
go test -cover ./pkg/mypackage

# Multiple packages
go test -cover ./pkg/... ./internal/...

# Exclude vendor
go test -cover $(go list ./... | grep -v /vendor/)
```

## Coverage Report Formats

### HTML Reports

**Jest/Vitest**: Open `coverage/index.html`
**pytest**: Open `htmlcov/index.html`
**Go**: `go tool cover -html=coverage.out`

### Terminal Reports

**Jest**:
```bash
npm test -- --coverage --coverageReporters=text
```

**pytest**:
```bash
pytest --cov=src --cov-report=term-missing
```

**Go**:
```bash
go tool cover -func=coverage.out
```

### LCOV Format

**Jest**:
```json
{
  "jest": {
    "coverageReporters": ["lcov"]
  }
}
```

**pytest**:
```bash
pytest --cov=src --cov-report=lcov
```

### JSON Format

**Jest**:
```json
{
  "jest": {
    "coverageReporters": ["json"]
  }
}
```

**pytest**:
```bash
pytest --cov=src --cov-report=json
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Test Coverage

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tests with coverage
        run: npm test -- --coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true
```

### GitLab CI

```yaml
test:
  script:
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
```

### Coverage Badges

**Codecov**:
```markdown
[![codecov](https://codecov.io/gh/username/repo/branch/main/graph/badge.svg)](https://codecov.io/gh/username/repo)
```

**Coveralls**:
```markdown
[![Coverage Status](https://coveralls.io/repos/github/username/repo/badge.svg?branch=main)](https://coveralls.io/github/username/repo?branch=main)
```

## Troubleshooting

### Low Coverage Despite Tests

**Issue**: Tests run but coverage is low
**Solution**: 
- Check `collectCoverageFrom` patterns
- Verify test files are being executed
- Ensure imports are correct

### Missing Files in Report

**Issue**: Some files not in coverage report
**Solution**:
- Check ignore patterns
- Verify file extensions match
- Ensure files are imported somewhere

### Slow Coverage Generation

**Issue**: Coverage slows down tests significantly
**Solution**:
- Use `--coverage` only in CI
- Use faster coverage provider (v8 vs istanbul)
- Reduce `collectCoverageFrom` scope

### Branch Coverage Lower Than Line Coverage

**Issue**: Branch coverage significantly lower
**Solution**:
- Add tests for else branches
- Test error paths
- Test early returns
- Test switch/case alternatives

---
name: test-coverage-analyzer
description: Analyzes test coverage reports, identifies gaps, and recommends priority areas for testing. Use when reviewing coverage, finding untested code, or planning test improvements.
---

# Test Coverage Analyzer

Analyze test coverage and identify testing gaps.

## Quick Start

Run coverage and analyze results:
```bash
# JavaScript/TypeScript
npm test -- --coverage

# Python
pytest --cov=src --cov-report=term-missing

# Go
go test -cover ./...
```

## Instructions

### Step 1: Generate Coverage Report

**JavaScript/TypeScript (Jest)**:
```bash
npm test -- --coverage --coverageReporters=text --coverageReporters=lcov
```

**JavaScript/TypeScript (Vitest)**:
```bash
vitest run --coverage
```

**Python (pytest)**:
```bash
pytest --cov=src --cov-report=html --cov-report=term-missing
```

**Go**:
```bash
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### Step 2: Analyze Coverage Metrics

Review these key metrics:
- **Line coverage**: Percentage of lines executed
- **Branch coverage**: Percentage of conditional branches tested
- **Function coverage**: Percentage of functions called
- **Statement coverage**: Percentage of statements executed

**Target thresholds**:
- Critical code: 80%+ coverage
- Standard code: 60%+ coverage
- UI/Config: 40%+ coverage

### Step 3: Identify Gaps

Look for:
- **Uncovered functions**: Functions with 0% coverage
- **Partial coverage**: Functions with <50% coverage
- **Missing branches**: Untested if/else paths
- **Error paths**: Untested catch blocks
- **Edge cases**: Boundary conditions not tested

### Step 4: Prioritize Testing

**High Priority** (test first):
- Business logic with 0% coverage
- Security-critical functions
- Payment/transaction code
- Data validation logic
- Error handling paths

**Medium Priority** (test next):
- API endpoints
- Database operations
- Utility functions
- Configuration logic

**Low Priority** (test if time permits):
- Simple getters/setters
- UI presentation logic
- Type definitions
- Generated code

### Step 5: Create Action Plan

For each gap:
1. Identify the untested code
2. Determine test type needed (unit/integration/e2e)
3. Estimate effort (small/medium/large)
4. Assign priority (high/medium/low)
5. Create test implementation tasks

## Common Patterns

**Pattern: Find untested files**
```bash
# Jest
npm test -- --coverage --collectCoverageFrom='src/**/*.{js,ts}' --coverageThreshold='{"global":{"lines":0}}'

# pytest
pytest --cov=src --cov-report=term-missing | grep "0%"
```

**Pattern: Check specific module**
```bash
# Jest
npm test -- path/to/module --coverage

# pytest
pytest tests/test_module.py --cov=src.module --cov-report=term-missing
```

**Pattern: Enforce coverage thresholds**
```json
// package.json (Jest)
{
  "jest": {
    "coverageThreshold": {
      "global": {
        "branches": 70,
        "functions": 70,
        "lines": 70,
        "statements": 70
      }
    }
  }
}
```

## Advanced

For detailed information, see:
- [Coverage Tools](reference/coverage-tools.md) - Tool-specific configuration and usage
- [Testing Patterns](reference/patterns.md) - Common testing patterns and best practices

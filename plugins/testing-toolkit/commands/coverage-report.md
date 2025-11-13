---
description: Generate and display test coverage report with analysis and recommendations
allowed-tools: Bash(npm:*), Bash(yarn:*), Bash(pytest:*), Bash(go:*), Read, Write
argument-hint: [format]
---

# Coverage Report

## Context

- Project type: !`[ -f "package.json" ] && echo "Node.js" || [ -f "pyproject.toml" ] && echo "Python" || [ -f "go.mod" ] && echo "Go" || echo "Unknown"`
- Test framework: !`[ -f "package.json" ] && (grep -q "jest" package.json && echo "Jest" || grep -q "vitest" package.json && echo "Vitest" || echo "Unknown") || [ -f "pyproject.toml" ] && echo "pytest" || echo "Unknown"`
- Coverage files: !`find . -name "coverage" -o -name "htmlcov" -o -name "coverage.out" 2>/dev/null | head -5`

## Your Task

Generate a comprehensive test coverage report and provide analysis with recommendations.

### Arguments

- `$1` (optional): Report format - `terminal`, `html`, `json`, or `all` (default: `terminal`)

### Steps

1. **Generate coverage report** based on framework:

   **Jest**:
   ```bash
   npm test -- --coverage --coverageReporters=text --coverageReporters=html --coverageReporters=json
   ```

   **Vitest**:
   ```bash
   vitest run --coverage
   ```

   **pytest**:
   ```bash
   pytest --cov=src --cov-report=term-missing --cov-report=html --cov-report=json
   ```

   **Go**:
   ```bash
   go test -coverprofile=coverage.out ./...
   go tool cover -func=coverage.out
   ```

2. **Display terminal report** with key metrics:
   - Overall coverage percentage
   - Line coverage
   - Branch coverage
   - Function coverage
   - Statement coverage

3. **Analyze coverage data** and identify:
   - Files with 0% coverage
   - Files with <50% coverage
   - Critical files with low coverage
   - Uncovered branches and error paths

4. **Provide recommendations**:

   **High Priority** (address first):
   - List files with 0% coverage
   - Identify critical business logic with low coverage
   - Highlight security-related code without tests

   **Medium Priority** (address next):
   - List files with <50% coverage
   - Identify API endpoints without tests
   - Highlight database operations without tests

   **Low Priority** (address if time permits):
   - List files with 50-70% coverage
   - Identify UI components without tests
   - Highlight utility functions with partial coverage

5. **Generate HTML report** (if requested):
   - Open coverage/index.html (Jest/Vitest)
   - Open htmlcov/index.html (pytest)
   - Generate HTML with `go tool cover -html=coverage.out` (Go)

6. **Create summary** with:
   - Current coverage: X%
   - Target coverage: 70-80%
   - Gap: Y%
   - Estimated effort to reach target
   - Top 5 files to test next

### Examples

**Generate terminal report**:
```
/coverage-report
```

**Generate HTML report**:
```
/coverage-report html
```

**Generate all formats**:
```
/coverage-report all
```

### Output Format

```
📊 Test Coverage Report
=======================

Overall Coverage: 65.4%
├─ Lines: 68.2%
├─ Branches: 58.9%
├─ Functions: 70.1%
└─ Statements: 67.8%

🔴 High Priority (0% coverage):
├─ src/utils/payment.js
├─ src/services/auth.js
└─ src/validators/user.js

🟡 Medium Priority (<50% coverage):
├─ src/api/orders.js (42%)
├─ src/models/product.js (38%)
└─ src/middleware/auth.js (45%)

🟢 Low Priority (50-70% coverage):
├─ src/utils/format.js (62%)
└─ src/helpers/date.js (58%)

📈 Recommendations:
1. Add tests for payment.js (critical business logic)
2. Add tests for auth.js (security-critical)
3. Improve coverage for orders.js API endpoints
4. Target: 75% overall coverage (+9.6%)

💡 Next Steps:
Run: npm test src/utils/payment.test.js --coverage
```

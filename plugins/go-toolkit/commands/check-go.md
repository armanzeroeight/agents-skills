---
description: Run Go code quality checks including formatting, linting, vetting, and tests
allowed-tools: Bash(go:*,gofmt:*,golint:*,staticcheck:*)
argument-hint: [--fix]
---

# Check Go

Run comprehensive Go code quality checks.

## Context

- Go version: !`go version`
- Current directory: !`pwd`
- Go files: !`find . -name "*.go" -not -path "*/vendor/*" | head -10`

## Your Task

Run Go code quality checks and provide actionable feedback.

### Arguments

- `$1`: Optional `--fix` flag to automatically fix issues

### Steps

1. **Check Go installation**
   - Verify go command is available
   - Check Go version

2. **Run gofmt**
   - Check code formatting
   - If --fix: Apply formatting
   - Report unformatted files

3. **Run go vet**
   - Check for suspicious constructs
   - Report issues found
   - Provide fix suggestions

4. **Run staticcheck (if available)**
   - Advanced static analysis
   - Check for bugs and inefficiencies
   - Report findings

5. **Run go test**
   - Execute all tests
   - Report test results
   - Show coverage if available

6. **Check for race conditions**
   - Run tests with -race flag
   - Report any race conditions

7. **Provide summary**
   - List all issues found
   - Categorize by severity
   - Suggest fixes
   - Show overall status

### Output Format

```markdown
# Go Code Quality Report

## Formatting
- Status: [PASS/FAIL]
- Unformatted files: [count]
- [List of files if any]

## Go Vet
- Status: [PASS/FAIL]
- Issues found: [count]
- [List of issues with file:line]

## Static Analysis
- Status: [PASS/FAIL/SKIPPED]
- Issues found: [count]
- [List of issues]

## Tests
- Status: [PASS/FAIL]
- Tests run: [count]
- Passed: [count]
- Failed: [count]
- Coverage: [percentage]

## Race Detection
- Status: [PASS/FAIL/SKIPPED]
- Races found: [count]

## Summary
- Overall: [PASS/FAIL]
- Total issues: [count]
- Action required: [yes/no]

## Recommendations
1. [Fix suggestion 1]
2. [Fix suggestion 2]
...
```

## Examples

**Check code quality:**
```
/check-go
```

**Check and auto-fix:**
```
/check-go --fix
```

**Expected output:**
- Formatting status
- Vet results
- Static analysis findings
- Test results
- Race detection results
- Actionable recommendations

---
description: Validate dbt project configuration, models, and tests
allowed-tools: Bash(dbt:*), Read
argument-hint: [model-name]
---

# Validate dbt Project

## Context

- dbt version: !`dbt --version 2>&1 || echo "dbt not installed"`
- Current directory: !`pwd`
- dbt project: !`ls dbt_project.yml 2>&1 && echo "Found" || echo "Not in dbt project"`
- Models: !`find models -name "*.sql" 2>&1 | head -10 || echo "No models found"`

## Your Task

Validate dbt project configuration, compile models, and run tests.

### Arguments

- `$1`: Model name to validate (optional, validates all if not provided)

### Steps

1. **Validate project configuration**
   ```bash
   dbt debug
   ```

2. **Compile models**
   ```bash
   dbt compile --select $1
   ```

3. **Run tests**
   ```bash
   dbt test --select $1
   ```

4. **Check documentation**
   - Verify all models have descriptions
   - Check column documentation
   - Ensure tests are defined

5. **Provide recommendations**
   - Missing tests
   - Undocumented models
   - Performance improvements
   - Best practice violations

### Output Format

```markdown
## dbt Validation Results

**Project:** [project-name]
**Models Checked:** [count]

### Configuration
✓ Passed / ✗ Failed

### Compilation
✓ Passed / ✗ Failed
[Compilation errors if any]

### Tests
✓ Passed / ✗ Failed
- Total: [count]
- Passed: [count]
- Failed: [count]

### Documentation Coverage
- Models documented: [count]/[total]
- Columns documented: [count]/[total]

### Recommendations
1. [Recommendation]
2. [Recommendation]
```

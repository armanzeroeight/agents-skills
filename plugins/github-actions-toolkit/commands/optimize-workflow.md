---
description: Analyze and optimize GitHub Actions workflow for performance, cost, and best practices
allowed-tools: Read, Bash(cat:*,ls:*,find:*,grep:*)
argument-hint: [workflow-file]
---

# Optimize Workflow

Analyze a GitHub Actions workflow file and provide optimization recommendations.

## Context

- Workflow file: !`[ -f "$1" ] && cat "$1" || find .github/workflows -name "*.yml" -o -name "*.yaml" | head -5`
- Repository structure: !`ls -la`

## Your Task

Analyze the GitHub Actions workflow and provide specific optimization recommendations.

### Arguments

- `$1`: Path to workflow file (optional, will search .github/workflows if not provided)

### Steps

1. **Identify the workflow file**
   - If argument provided, use that file
   - Otherwise, list workflows in .github/workflows and ask user which to analyze

2. **Analyze workflow structure**
   - Check for parallelization opportunities
   - Identify caching opportunities
   - Review job dependencies
   - Check for redundant steps

3. **Performance optimizations**
   - Suggest caching for dependencies (npm, pip, cargo, etc.)
   - Recommend artifact usage for inter-job data sharing
   - Identify steps that can run in parallel
   - Check checkout depth (shallow clones)
   - Review matrix strategy efficiency

4. **Cost optimizations**
   - Suggest concurrency controls to cancel redundant runs
   - Recommend appropriate runner types
   - Identify unnecessary jobs or steps
   - Check for over-testing (too many matrix combinations)

5. **Best practices**
   - Verify actions are pinned to specific versions (SHA)
   - Check for hardcoded secrets (should use GitHub Secrets)
   - Review permissions (use least-privilege)
   - Suggest reusable workflows for common patterns
   - Check for proper error handling

6. **Security recommendations**
   - Verify `permissions` are set appropriately
   - Check for OIDC usage with cloud providers
   - Review secret handling
   - Suggest environment protection rules if deploying

7. **Provide specific recommendations**
   - List issues found with severity (high/medium/low)
   - Provide code snippets for fixes
   - Estimate potential time/cost savings
   - Prioritize recommendations by impact

### Output Format

```markdown
# Workflow Optimization Report

## Summary
- Current jobs: X
- Estimated runtime: Y minutes
- Potential improvements: Z

## High Priority
1. [Issue description]
   - Impact: [time/cost savings]
   - Fix: [code snippet]

## Medium Priority
...

## Low Priority
...

## Optimized Workflow
[Provide optimized version of key sections]
```

## Examples

**Analyze specific workflow:**
```
/optimize-workflow .github/workflows/ci.yml
```

**Analyze workflows in repository:**
```
/optimize-workflow
```

**Expected output:**
- List of optimization opportunities
- Specific code changes to implement
- Estimated performance improvements
- Best practice recommendations

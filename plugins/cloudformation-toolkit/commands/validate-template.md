---
description: Validate CloudFormation template for syntax, security, and best practices
allowed-tools: Bash(aws:*), Bash(cfn-lint:*), Bash(cfn_nag:*), Read
argument-hint: [template-file]
---

# Validate CloudFormation Template

## Context

- Current directory: !`pwd`
- Available CloudFormation templates: !`find . -name "*.yaml" -o -name "*.yml" -o -name "*.json" | grep -E "(template|stack|cfn)" | head -20`
- AWS CLI version: !`aws --version 2>&1 || echo "AWS CLI not installed"`
- cfn-lint installed: !`cfn-lint --version 2>&1 || echo "cfn-lint not installed"`

## Your Task

Validate the specified CloudFormation template for syntax errors, security issues, and best practices violations.

### Arguments

- `$1`: Path to CloudFormation template file (required)

### Steps

1. **Verify template file exists**
   - Check if the specified file exists
   - If not provided, list available templates and ask user to specify

2. **Run AWS CloudFormation validation**
   ```bash
   aws cloudformation validate-template --template-body file://$1
   ```
   - Check for syntax errors
   - Verify template structure
   - Report any AWS validation errors

3. **Run cfn-lint validation (if available)**
   ```bash
   cfn-lint $1
   ```
   - Check for best practices violations
   - Identify resource property issues
   - Report warnings and errors
   - If cfn-lint not installed, suggest: `pip install cfn-lint`

4. **Run cfn-nag security scan (if available)**
   ```bash
   cfn_nag_scan --input-path $1
   ```
   - Check for security issues
   - Identify overly permissive IAM policies
   - Check for unencrypted resources
   - Report security warnings and failures
   - If cfn-nag not installed, suggest: `gem install cfn-nag`

5. **Analyze and summarize results**
   - Categorize issues by severity (errors, warnings, info)
   - Highlight critical security issues
   - Provide specific recommendations for fixes
   - Show line numbers for issues when available

6. **Provide actionable recommendations**
   - For each issue, explain what's wrong
   - Suggest specific fixes with code examples
   - Prioritize by impact (high/medium/low)
   - Reference best practices documentation

### Output Format

Provide validation results in this format:

```markdown
## CloudFormation Template Validation Results

**Template:** [filename]
**Size:** [file size]

### AWS Validation
✓ Passed / ✗ Failed
[Details of AWS validation]

### cfn-lint Analysis
✓ Passed / ✗ Failed
- Errors: [count]
- Warnings: [count]
- Info: [count]

[List of issues with line numbers]

### Security Scan (cfn-nag)
✓ Passed / ✗ Failed
- Failures: [count]
- Warnings: [count]

[List of security issues]

### Summary

**Critical Issues:** [count]
- [Issue 1 with recommendation]
- [Issue 2 with recommendation]

**Warnings:** [count]
- [Warning 1 with recommendation]

**Recommendations:**
1. [Priority 1 fix]
2. [Priority 2 fix]
3. [Priority 3 fix]
```

### Examples

```bash
# Validate specific template
/validate-template infrastructure/network.yaml

# Validate template in current directory
/validate-template template.yaml

# Validate nested stack template
/validate-template stacks/database-stack.yaml
```

### Common Issues to Check

**Syntax:**
- Valid YAML/JSON format
- Correct intrinsic function usage
- Valid resource types
- Required properties present

**Security:**
- No wildcard IAM permissions
- No unrestricted security group rules (0.0.0.0/0 for SSH/RDP)
- Encryption enabled for S3, RDS, EBS
- No hardcoded credentials
- Secrets Manager/SSM for sensitive values

**Best Practices:**
- Descriptive resource names
- Tags applied to resources
- Parameter validation
- Output descriptions
- DependsOn only when necessary

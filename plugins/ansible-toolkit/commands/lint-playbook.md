---
description: Lint and validate Ansible playbooks for syntax and best practices
allowed-tools: Bash(ansible:*), Bash(ansible-lint:*), Bash(yamllint:*), Read
argument-hint: [playbook-file]
---

# Lint Ansible Playbook

## Context

- Current directory: !`pwd`
- Available playbooks: !`find . -name "*.yml" -o -name "*.yaml" | grep -E "(playbook|site)" | head -20`
- Ansible version: !`ansible --version 2>&1 | head -1 || echo "Ansible not installed"`
- ansible-lint installed: !`ansible-lint --version 2>&1 || echo "ansible-lint not installed"`

## Your Task

Lint and validate the specified Ansible playbook for syntax errors, best practices violations, and potential issues.

### Arguments

- `$1`: Path to Ansible playbook file (required)

### Steps

1. **Verify playbook file exists**
   - Check if the specified file exists
   - If not provided, list available playbooks and ask user to specify

2. **Run syntax check**
   ```bash
   ansible-playbook --syntax-check $1
   ```
   - Verify YAML syntax
   - Check for undefined variables
   - Validate task structure
   - Report syntax errors

3. **Run ansible-lint (if available)**
   ```bash
   ansible-lint $1
   ```
   - Check best practices
   - Identify deprecated syntax
   - Check for security issues
   - Report warnings and errors
   - If not installed, suggest: `pip install ansible-lint`

4. **Run yamllint (if available)**
   ```bash
   yamllint $1
   ```
   - Check YAML formatting
   - Verify indentation
   - Check line length
   - If not installed, suggest: `pip install yamllint`

5. **Analyze and summarize results**
   - Categorize issues by severity
   - Highlight critical issues
   - Provide specific recommendations
   - Show line numbers for issues

6. **Provide actionable recommendations**
   - Explain each issue
   - Suggest specific fixes
   - Prioritize by impact
   - Reference best practices

### Output Format

```markdown
## Ansible Playbook Lint Results

**Playbook:** [filename]
**Size:** [file size]

### Syntax Check
✓ Passed / ✗ Failed
[Details of syntax check]

### ansible-lint Analysis
✓ Passed / ✗ Failed
- Errors: [count]
- Warnings: [count]

[List of issues with line numbers and rules]

### YAML Lint
✓ Passed / ✗ Failed
- Errors: [count]
- Warnings: [count]

### Summary

**Critical Issues:** [count]
- [Issue 1 with fix]
- [Issue 2 with fix]

**Warnings:** [count]
- [Warning 1 with recommendation]

**Recommendations:**
1. [Priority 1 fix with code example]
2. [Priority 2 fix with code example]
```

### Examples

```bash
# Lint specific playbook
/lint-playbook site.yml

# Lint playbook in subdirectory
/lint-playbook playbooks/deploy.yml

# Lint role tasks
/lint-playbook roles/webserver/tasks/main.yml
```

### Common Issues to Check

**Syntax:**
- Valid YAML format
- Proper indentation
- Correct module names
- Required parameters present

**Best Practices:**
- Use fully qualified collection names (ansible.builtin.*)
- Descriptive task names
- Proper use of handlers
- Idempotent tasks
- No command/shell when module exists

**Security:**
- No hardcoded passwords
- Use ansible-vault for secrets
- Proper file permissions
- No become without need

**Style:**
- Consistent indentation (2 spaces)
- Line length under 160 characters
- Proper YAML syntax
- Comments for complex logic

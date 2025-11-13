---
description: Generate and create a commit with a conventional commit message based on staged changes
allowed-tools: Bash(git:*), Read
argument-hint: [--amend] [--no-verify]
---

# Smart Commit

## Context

- Current branch: !`git branch --show-current`
- Staged changes: !`git diff --staged --stat`
- Recent commits: !`git log --oneline -3`

## Your Task

Analyze staged changes and generate a conventional commit message, then create the commit.

### Arguments

- `--amend` (optional): Amend the last commit instead of creating new one
- `--no-verify` (optional): Skip pre-commit hooks

### Steps

1. **Analyze staged changes**:
   ```bash
   git diff --staged
   ```

2. **Determine commit type** based on changes:
   - `feat`: New feature or functionality
   - `fix`: Bug fix
   - `docs`: Documentation only
   - `style`: Formatting, whitespace
   - `refactor`: Code restructuring
   - `test`: Adding or updating tests
   - `chore`: Maintenance, dependencies
   - `perf`: Performance improvement

3. **Identify scope** (optional):
   - Component name (e.g., `auth`, `api`, `ui`)
   - Module name (e.g., `payments`, `users`)
   - Feature area (e.g., `dashboard`, `settings`)

4. **Generate commit message** following format:
   ```
   <type>(<scope>): <description>
   
   [optional body]
   
   [optional footer]
   ```

5. **Create commit**:
   ```bash
   git commit -m "type(scope): description"
   ```
   
   Or for detailed message:
   ```bash
   git commit -m "type(scope): description" -m "Body text" -m "Footer text"
   ```

6. **Confirm commit**:
   ```bash
   git log -1 --pretty=format:"%h - %s"
   ```

### Examples

**Simple feature commit**:
```
/smart-commit
# Analyzes: Added new login form component
# Generates: feat(auth): add login form component
# Commits with generated message
```

**Bug fix with details**:
```
/smart-commit
# Analyzes: Fixed null pointer in user service
# Generates:
# fix(users): handle null response in getUserById
#
# Previously crashed when user data was null.
# Now returns 404 with appropriate error message.
#
# Fixes #123
```

**Amend last commit**:
```
/smart-commit --amend
# Updates the last commit message based on current staged changes
```

**Skip hooks**:
```
/smart-commit --no-verify
# Creates commit without running pre-commit hooks
```

### Commit Message Guidelines

**Subject line**:
- Use imperative mood ("add" not "added")
- No period at end
- Max 50-72 characters
- Lowercase after type
- Clear and concise

**Body** (if needed):
- Explain what and why, not how
- Wrap at 72 characters
- Separate from subject with blank line

**Footer** (if needed):
- Breaking changes: `BREAKING CHANGE: description`
- Issue references: `Closes #123`, `Fixes #456`

### Quality Checks

Before committing, verify:
- [ ] Changes are staged (`git diff --staged`)
- [ ] Commit is atomic (one logical change)
- [ ] Tests pass (if applicable)
- [ ] No debug code or console.logs
- [ ] No commented-out code
- [ ] Message follows conventions

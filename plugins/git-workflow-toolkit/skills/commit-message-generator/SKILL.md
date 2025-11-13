---
name: commit-message-generator
description: Generates conventional commit messages by analyzing git diffs and changes. Use when writing commit messages, following commit conventions, or documenting changes.
---

# Commit Message Generator

Generate clear, conventional commit messages from git diffs.

## Quick Start

Analyze staged changes and generate message:
```bash
git diff --staged
# Then generate message following conventional commits format
```

## Instructions

### Step 1: Analyze Changes

Review the git diff to understand:
- What files changed
- What functionality was added/modified/removed
- Scope of changes (component, module, feature)
- Breaking changes or deprecations

```bash
git diff --staged
# or for specific files
git diff --staged path/to/file
```

### Step 2: Determine Commit Type

**feat**: New feature
- Adding new functionality
- New user-facing capability
- New API endpoint

**fix**: Bug fix
- Fixing a bug
- Correcting behavior
- Resolving an issue

**docs**: Documentation
- README updates
- Code comments
- API documentation

**style**: Code style
- Formatting changes
- Missing semicolons
- Whitespace changes
- No code logic changes

**refactor**: Code refactoring
- Restructuring code
- No functionality change
- Performance improvements

**test**: Tests
- Adding tests
- Updating tests
- Test infrastructure

**chore**: Maintenance
- Dependency updates
- Build configuration
- CI/CD changes
- Tooling updates

**perf**: Performance
- Performance improvements
- Optimization changes

**ci**: CI/CD
- CI configuration
- Build scripts
- Deployment changes

**build**: Build system
- Build tool changes
- External dependencies

**revert**: Revert
- Reverting previous commit

### Step 3: Identify Scope

Scope indicates what part of codebase changed:
- Component name: `(button)`, `(navbar)`
- Module name: `(auth)`, `(api)`
- Feature area: `(payments)`, `(search)`
- Package name: `(core)`, `(utils)`

Optional but recommended for clarity.

### Step 4: Write Description

**Subject line** (first line):
- Use imperative mood ("add" not "added")
- No period at end
- Max 50-72 characters
- Lowercase after type
- Clear and concise

**Body** (optional, after blank line):
- Explain what and why, not how
- Wrap at 72 characters
- Use bullet points for multiple changes
- Reference issues/tickets

**Footer** (optional):
- Breaking changes: `BREAKING CHANGE: description`
- Issue references: `Closes #123`, `Fixes #456`
- Co-authors: `Co-authored-by: Name <email>`

### Step 5: Format Message

**Basic format**:
```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Examples**:

```
feat(auth): add JWT token refresh mechanism

Implement automatic token refresh when access token expires.
Tokens are refreshed 5 minutes before expiration.

Closes #234
```

```
fix(api): handle null response in user endpoint

Previously crashed when user data was null.
Now returns 404 with appropriate error message.

Fixes #567
```

```
docs(readme): update installation instructions

Add prerequisites section and troubleshooting guide.
```

```
refactor(utils): simplify date formatting logic

Extract common date operations into helper functions.
No functionality changes.
```

```
chore(deps): upgrade react to v18.2.0

Update react and react-dom dependencies.
Update tests to match new API.
```

## Common Patterns

**Multiple changes in one commit**:
```
feat(dashboard): add user analytics and export

- Add analytics charts for user activity
- Implement CSV export functionality
- Add date range filter

Closes #123, #124
```

**Breaking change**:
```
feat(api)!: change authentication endpoint structure

BREAKING CHANGE: Auth endpoints now use /v2/auth prefix.
Update client code to use new endpoints.

Migration guide: docs/migration-v2.md
```

**Revert commit**:
```
revert: feat(auth): add JWT token refresh

This reverts commit abc123def456.
Reason: Causing issues in production.
```

**Co-authored commit**:
```
feat(search): implement fuzzy search algorithm

Co-authored-by: Jane Doe <jane@example.com>
```

## Commit Message Quality

**Good commit messages**:
- Clear and descriptive
- Explain why, not just what
- Reference related issues
- Follow conventions consistently
- Atomic (one logical change)

**Bad commit messages**:
- "fix stuff"
- "WIP"
- "updates"
- "asdf"
- "fix fix fix"

## Workflow Integration

### Before committing:

1. Review changes: `git diff --staged`
2. Ensure changes are atomic
3. Generate appropriate message
4. Commit: `git commit -m "type(scope): description"`

### For detailed commits:

1. Stage changes: `git add files`
2. Open editor: `git commit`
3. Write multi-line message
4. Save and close

### Amending last commit:

```bash
git commit --amend
# Edit message in editor
```

### Interactive staging:

```bash
git add -p  # Stage hunks interactively
git commit  # Write message for staged changes
```

## Conventional Commits Spec

Format: `<type>[optional scope]: <description>`

**Required**:
- type: Must be one of the defined types
- description: Brief summary of change

**Optional**:
- scope: Component/module affected
- body: Detailed explanation
- footer: Breaking changes, issue references

**Rules**:
- Type must be lowercase
- Scope in parentheses
- Colon and space after scope
- Description starts lowercase
- No period at end of description
- Body separated by blank line
- Footer separated by blank line

## Advanced

For complex scenarios:
- Multi-commit features
- Monorepo commit conventions
- Automated commit message validation
- Commit message templates
- Semantic versioning integration

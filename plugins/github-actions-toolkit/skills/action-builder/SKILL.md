---
name: action-builder
description: Create custom GitHub Actions (composite, Docker, or JavaScript). Use when building reusable actions, creating custom workflow steps, or packaging logic for distribution. Trigger words include "create action", "custom action", "build action", "composite action", "Docker action".
---

# Action Builder

Build custom GitHub Actions for reusable workflow logic.

## Quick Start

Choose action type based on requirements:
- **Composite**: Combine multiple workflow steps (simplest, no code)
- **Docker**: Run containerized logic (any language, slower startup)
- **JavaScript**: Fast execution, Node.js required

## Instructions

### Step 1: Choose Action Type

**Use Composite when:**
- Combining existing actions and shell commands
- No custom code needed
- Fast execution required
- Example: Setup environment with multiple tools

**Use Docker when:**
- Need specific runtime environment
- Using non-JavaScript languages
- Complex dependencies
- Example: Custom linting tool, data processing

**Use JavaScript when:**
- Need programmatic control
- Fast startup time critical
- Interacting with GitHub API
- Example: Issue labeler, PR validator

### Step 2: Create Action Structure

**All action types need:**
```
action-name/
├── action.yml          # Action metadata
├── README.md           # Documentation
└── [implementation]    # Type-specific files
```

### Step 3: Define action.yml

**Required fields:**
```yaml
name: 'Action Name'
description: 'What the action does'
author: 'Your Name'

inputs:
  input-name:
    description: 'Input description'
    required: true
    default: 'default-value'

outputs:
  output-name:
    description: 'Output description'
    value: ${{ steps.step-id.outputs.output-name }}

runs:
  using: 'composite|docker|node16'
  # Type-specific configuration
```

### Step 4: Implement Action Logic

**For Composite Actions:**
```yaml
runs:
  using: 'composite'
  steps:
    - name: Step name
      run: echo "Command"
      shell: bash
    - uses: actions/checkout@v3
```

**For Docker Actions:**
```yaml
runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.input-name }}
```

**For JavaScript Actions:**
```yaml
runs:
  using: 'node16'
  main: 'dist/index.js'
```

### Step 5: Test Action Locally

**Test in workflow:**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: ./  # Test local action
        with:
          input-name: 'test-value'
```

### Step 6: Version and Publish

**Versioning:**
- Use semantic versioning (v1.0.0)
- Create Git tags for releases
- Maintain major version tags (v1, v2)

**Publishing:**
- Push to GitHub repository
- Create release with tag
- Add to GitHub Marketplace (optional)

## Common Patterns

### Input Validation

```yaml
inputs:
  environment:
    description: 'Deployment environment'
    required: true
  region:
    description: 'AWS region'
    required: false
    default: 'us-east-1'
```

### Output Setting

**Composite:**
```bash
echo "output-name=value" >> $GITHUB_OUTPUT
```

**JavaScript:**
```javascript
core.setOutput('output-name', 'value');
```

### Error Handling

**Composite:**
```bash
if [ $? -ne 0 ]; then
  echo "::error::Operation failed"
  exit 1
fi
```

**JavaScript:**
```javascript
try {
  // Action logic
} catch (error) {
  core.setFailed(error.message);
}
```

## Advanced

For detailed implementation guides:
- [Composite Actions](reference/composite-actions.md) - Multi-step workflows
- [Docker Actions](reference/docker-actions.md) - Containerized actions
- [Inputs and Outputs](reference/inputs-outputs.md) - Data handling patterns

## Troubleshooting

**Action not found:**
- Verify action.yml exists in repository root
- Check action path in workflow (uses: owner/repo@version)

**Inputs not working:**
- Confirm input names match action.yml
- Check required vs. optional inputs
- Verify default values

**Outputs not available:**
- Ensure output is set before job completes
- Check step ID matches output reference
- Verify output syntax (${{ steps.id.outputs.name }})

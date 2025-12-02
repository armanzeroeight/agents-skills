# Composite Actions

Composite actions combine multiple workflow steps into a single reusable action using only YAML configuration.

## Contents
- When to use composite actions
- Structure and syntax
- Working with inputs and outputs
- Shell selection and environment
- Advanced patterns

## When to Use Composite Actions

**Best for:**
- Combining existing actions and shell commands
- Standardizing setup procedures across workflows
- No custom code or dependencies needed
- Fast execution (no container overhead)

**Examples:**
- Environment setup (install tools, configure settings)
- Multi-step validation (lint, format, type-check)
- Deployment preparation (build, test, package)

## Structure and Syntax

### Basic Structure

```yaml
name: 'Setup Node Environment'
description: 'Install Node.js and dependencies with caching'
author: 'Your Name'

inputs:
  node-version:
    description: 'Node.js version to install'
    required: true
    default: '18'
  cache-dependency-path:
    description: 'Path to package-lock.json'
    required: false
    default: 'package-lock.json'

outputs:
  cache-hit:
    description: 'Whether cache was restored'
    value: ${{ steps.cache.outputs.cache-hit }}

runs:
  using: 'composite'
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'
        cache-dependency-path: ${{ inputs.cache-dependency-path }}
    
    - name: Install dependencies
      run: npm ci
      shell: bash
    
    - name: Cache node_modules
      id: cache
      uses: actions/cache@v3
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles(inputs.cache-dependency-path) }}
```

### Shell Specification

**Always specify shell in composite actions:**
```yaml
- run: echo "Hello"
  shell: bash  # Required in composite actions
```

**Available shells:**
- `bash`: Cross-platform (recommended)
- `pwsh`: PowerShell Core
- `python`: Python script
- `sh`: POSIX shell

## Working with Inputs

### Input Types and Validation

```yaml
inputs:
  # String input
  message:
    description: 'Message to display'
    required: true
  
  # Boolean input (passed as string 'true'/'false')
  verbose:
    description: 'Enable verbose output'
    required: false
    default: 'false'
  
  # Number input (passed as string)
  timeout:
    description: 'Timeout in seconds'
    required: false
    default: '300'
```

### Using Inputs in Steps

```yaml
steps:
  - name: Use inputs
    run: |
      echo "Message: ${{ inputs.message }}"
      if [ "${{ inputs.verbose }}" = "true" ]; then
        echo "Verbose mode enabled"
      fi
    shell: bash
```

### Input Validation

```yaml
steps:
  - name: Validate inputs
    run: |
      if [ -z "${{ inputs.required-input }}" ]; then
        echo "::error::required-input is empty"
        exit 1
      fi
      
      if [ "${{ inputs.environment }}" != "dev" ] && \
         [ "${{ inputs.environment }}" != "prod" ]; then
        echo "::error::Invalid environment: ${{ inputs.environment }}"
        exit 1
      fi
    shell: bash
```

## Working with Outputs

### Setting Outputs

```yaml
steps:
  - name: Generate output
    id: generate
    run: |
      RESULT="computed-value"
      echo "result=$RESULT" >> $GITHUB_OUTPUT
    shell: bash

outputs:
  result:
    description: 'Computed result'
    value: ${{ steps.generate.outputs.result }}
```

### Multiple Outputs

```yaml
steps:
  - name: Build info
    id: build
    run: |
      echo "version=$(cat package.json | jq -r .version)" >> $GITHUB_OUTPUT
      echo "commit=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT
      echo "timestamp=$(date -u +%Y%m%d%H%M%S)" >> $GITHUB_OUTPUT
    shell: bash

outputs:
  version:
    description: 'Package version'
    value: ${{ steps.build.outputs.version }}
  commit:
    description: 'Git commit SHA'
    value: ${{ steps.build.outputs.commit }}
  timestamp:
    description: 'Build timestamp'
    value: ${{ steps.build.outputs.timestamp }}
```

## Environment Variables

### Passing Environment Variables

```yaml
steps:
  - name: Set environment
    run: |
      echo "CUSTOM_VAR=value" >> $GITHUB_ENV
    shell: bash
  
  - name: Use environment
    run: echo "Value: $CUSTOM_VAR"
    shell: bash
```

### Using Secrets

```yaml
# In workflow calling the action
- uses: ./my-action
  with:
    api-key: ${{ secrets.API_KEY }}

# In action
steps:
  - name: Use secret
    run: |
      curl -H "Authorization: Bearer ${{ inputs.api-key }}" \
        https://api.example.com
    shell: bash
```

## Advanced Patterns

### Conditional Steps

```yaml
steps:
  - name: Optional step
    if: inputs.enable-feature == 'true'
    run: echo "Feature enabled"
    shell: bash
```

### Error Handling

```yaml
steps:
  - name: Risky operation
    id: risky
    continue-on-error: true
    run: |
      # Operation that might fail
      exit 1
    shell: bash
  
  - name: Handle failure
    if: steps.risky.outcome == 'failure'
    run: |
      echo "::warning::Risky operation failed, using fallback"
      # Fallback logic
    shell: bash
```

### Working with Files

```yaml
steps:
  - name: Create file
    run: |
      cat > config.json << EOF
      {
        "setting": "${{ inputs.setting }}",
        "enabled": ${{ inputs.enabled }}
      }
      EOF
    shell: bash
  
  - name: Process file
    run: |
      if [ -f config.json ]; then
        cat config.json
      fi
    shell: bash
```

### Matrix-like Behavior

```yaml
steps:
  - name: Process multiple items
    run: |
      for item in ${{ inputs.items }}; do
        echo "Processing: $item"
        # Process each item
      done
    shell: bash
```

## Testing Composite Actions

### Local Testing

```yaml
# .github/workflows/test-action.yml
name: Test Action

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Test action
        uses: ./  # Reference local action
        with:
          input1: 'test-value'
          input2: 'another-value'
      
      - name: Verify outputs
        run: |
          echo "Output: ${{ steps.test.outputs.result }}"
```

### Testing Different Inputs

```yaml
jobs:
  test:
    strategy:
      matrix:
        input: ['value1', 'value2', 'value3']
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: ./
        with:
          test-input: ${{ matrix.input }}
```

## Best Practices

1. **Always specify shell**: Required in composite actions
2. **Validate inputs**: Check required inputs and valid values
3. **Use step IDs**: For outputs and conditional logic
4. **Handle errors**: Use continue-on-error and check outcomes
5. **Document thoroughly**: Clear descriptions for inputs/outputs
6. **Keep focused**: One clear purpose per action
7. **Version properly**: Use semantic versioning
8. **Test locally**: Before publishing or tagging

## Common Pitfalls

**Missing shell specification:**
```yaml
# Wrong
- run: echo "Hello"

# Correct
- run: echo "Hello"
  shell: bash
```

**Incorrect output reference:**
```yaml
# Wrong
outputs:
  result:
    value: ${{ outputs.result }}

# Correct
outputs:
  result:
    value: ${{ steps.step-id.outputs.result }}
```

**Environment variable scope:**
```yaml
# Variables set in one step are not available in the next
# Use GITHUB_ENV to persist across steps
- run: |
    echo "VAR=value" >> $GITHUB_ENV
  shell: bash
```

## Example: Complete Composite Action

```yaml
name: 'Deploy Application'
description: 'Build, test, and deploy application'
author: 'DevOps Team'

inputs:
  environment:
    description: 'Deployment environment (dev/staging/prod)'
    required: true
  version:
    description: 'Version to deploy'
    required: false
    default: 'latest'
  skip-tests:
    description: 'Skip test execution'
    required: false
    default: 'false'

outputs:
  deployment-url:
    description: 'URL of deployed application'
    value: ${{ steps.deploy.outputs.url }}
  deployment-time:
    description: 'Deployment timestamp'
    value: ${{ steps.deploy.outputs.timestamp }}

runs:
  using: 'composite'
  steps:
    - name: Validate environment
      run: |
        if [ "${{ inputs.environment }}" != "dev" ] && \
           [ "${{ inputs.environment }}" != "staging" ] && \
           [ "${{ inputs.environment }}" != "prod" ]; then
          echo "::error::Invalid environment: ${{ inputs.environment }}"
          exit 1
        fi
      shell: bash
    
    - name: Build application
      run: |
        echo "Building version ${{ inputs.version }}"
        npm run build
      shell: bash
    
    - name: Run tests
      if: inputs.skip-tests != 'true'
      run: npm test
      shell: bash
    
    - name: Deploy
      id: deploy
      run: |
        URL="https://${{ inputs.environment }}.example.com"
        TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)
        
        echo "Deploying to $URL"
        # Deployment logic here
        
        echo "url=$URL" >> $GITHUB_OUTPUT
        echo "timestamp=$TIMESTAMP" >> $GITHUB_OUTPUT
      shell: bash
    
    - name: Verify deployment
      run: |
        curl -f ${{ steps.deploy.outputs.url }}/health || exit 1
      shell: bash
```

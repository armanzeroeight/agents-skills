# Inputs and Outputs

Comprehensive guide to handling inputs and outputs in GitHub Actions.

## Contents
- Input definition and types
- Output definition and setting
- Environment variables
- Secrets handling
- Advanced patterns

## Input Definition

### Basic Input Structure

```yaml
inputs:
  input-name:
    description: 'Description of the input'
    required: true|false
    default: 'default-value'
```

### Input Types

**GitHub Actions treats all inputs as strings.** Type conversion must be handled in your action code.

```yaml
inputs:
  # String input
  message:
    description: 'Message to display'
    required: true
  
  # Boolean (passed as string 'true' or 'false')
  enable-feature:
    description: 'Enable optional feature'
    required: false
    default: 'false'
  
  # Number (passed as string)
  timeout:
    description: 'Timeout in seconds'
    required: false
    default: '300'
  
  # Choice/Enum (validated in code)
  environment:
    description: 'Deployment environment'
    required: true
    # Note: No native enum support, validate in code
  
  # List (passed as string, parse in code)
  files:
    description: 'Comma-separated list of files'
    required: false
    default: 'file1.txt,file2.txt'
```

### Required vs Optional

```yaml
inputs:
  # Required input - workflow must provide
  api-key:
    description: 'API key for authentication'
    required: true
  
  # Optional with default
  region:
    description: 'AWS region'
    required: false
    default: 'us-east-1'
  
  # Optional without default (will be empty string)
  custom-config:
    description: 'Path to custom config'
    required: false
```

## Accessing Inputs

### In Composite Actions

```yaml
steps:
  - name: Use input
    run: |
      echo "Input value: ${{ inputs.input-name }}"
      
      # Boolean check
      if [ "${{ inputs.enable-feature }}" = "true" ]; then
        echo "Feature enabled"
      fi
      
      # Number comparison
      if [ "${{ inputs.timeout }}" -gt 300 ]; then
        echo "Long timeout"
      fi
    shell: bash
```

### In JavaScript Actions

```javascript
const core = require('@actions/core');

// Get inputs
const message = core.getInput('message', { required: true });
const enableFeature = core.getBooleanInput('enable-feature');
const timeout = parseInt(core.getInput('timeout'));
const files = core.getMultilineInput('files');

// With default value
const region = core.getInput('region') || 'us-east-1';
```

### In Docker Actions

**Via environment variables:**
```python
import os

# Environment variables are prefixed with INPUT_
# Hyphens become underscores, uppercase
message = os.environ.get('INPUT_MESSAGE')
enable_feature = os.environ.get('INPUT_ENABLE-FEATURE', 'false') == 'true'
timeout = int(os.environ.get('INPUT_TIMEOUT', '300'))
```

**Via command-line arguments:**
```yaml
# action.yml
runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.message }}
    - ${{ inputs.enable-feature }}
    - ${{ inputs.timeout }}
```

```python
# In Python
import sys

message = sys.argv[1]
enable_feature = sys.argv[2] == 'true'
timeout = int(sys.argv[3])
```

## Input Validation

### In Composite Actions

```yaml
steps:
  - name: Validate inputs
    run: |
      # Check required input
      if [ -z "${{ inputs.api-key }}" ]; then
        echo "::error::api-key is required"
        exit 1
      fi
      
      # Validate enum
      ENV="${{ inputs.environment }}"
      if [ "$ENV" != "dev" ] && [ "$ENV" != "staging" ] && [ "$ENV" != "prod" ]; then
        echo "::error::Invalid environment: $ENV. Must be dev, staging, or prod"
        exit 1
      fi
      
      # Validate number range
      TIMEOUT="${{ inputs.timeout }}"
      if [ "$TIMEOUT" -lt 0 ] || [ "$TIMEOUT" -gt 3600 ]; then
        echo "::error::Timeout must be between 0 and 3600"
        exit 1
      fi
      
      # Validate file exists
      CONFIG="${{ inputs.config-file }}"
      if [ -n "$CONFIG" ] && [ ! -f "$CONFIG" ]; then
        echo "::error::Config file not found: $CONFIG"
        exit 1
      fi
    shell: bash
```

### In JavaScript Actions

```javascript
const core = require('@actions/core');

try {
  // Get and validate
  const environment = core.getInput('environment', { required: true });
  const validEnvs = ['dev', 'staging', 'prod'];
  
  if (!validEnvs.includes(environment)) {
    throw new Error(`Invalid environment: ${environment}. Must be one of: ${validEnvs.join(', ')}`);
  }
  
  const timeout = parseInt(core.getInput('timeout'));
  if (isNaN(timeout) || timeout < 0 || timeout > 3600) {
    throw new Error('Timeout must be a number between 0 and 3600');
  }
  
  // Continue with validated inputs
} catch (error) {
  core.setFailed(error.message);
}
```

### In Python (Docker Actions)

```python
import os
import sys

def validate_environment(env):
    """Validate environment input."""
    valid = ['dev', 'staging', 'prod']
    if env not in valid:
        print(f"::error::Invalid environment: {env}. Must be one of: {', '.join(valid)}")
        sys.exit(1)

def validate_timeout(timeout_str):
    """Validate timeout input."""
    try:
        timeout = int(timeout_str)
        if timeout < 0 or timeout > 3600:
            raise ValueError()
        return timeout
    except ValueError:
        print("::error::Timeout must be a number between 0 and 3600")
        sys.exit(1)

# Get inputs
environment = os.environ.get('INPUT_ENVIRONMENT')
timeout_str = os.environ.get('INPUT_TIMEOUT', '300')

# Validate
validate_environment(environment)
timeout = validate_timeout(timeout_str)
```

## Output Definition

### Basic Output Structure

```yaml
outputs:
  output-name:
    description: 'Description of the output'
    value: ${{ steps.step-id.outputs.output-name }}
```

### Multiple Outputs

```yaml
outputs:
  version:
    description: 'Application version'
    value: ${{ steps.build.outputs.version }}
  
  commit-sha:
    description: 'Git commit SHA'
    value: ${{ steps.build.outputs.commit-sha }}
  
  build-time:
    description: 'Build timestamp'
    value: ${{ steps.build.outputs.build-time }}
  
  success:
    description: 'Whether build succeeded'
    value: ${{ steps.build.outputs.success }}
```

## Setting Outputs

### In Composite Actions

```yaml
steps:
  - name: Generate outputs
    id: generate
    run: |
      # Simple output
      echo "version=1.2.3" >> $GITHUB_OUTPUT
      
      # Computed output
      COMMIT=$(git rev-parse --short HEAD)
      echo "commit-sha=$COMMIT" >> $GITHUB_OUTPUT
      
      # Timestamp
      TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)
      echo "build-time=$TIMESTAMP" >> $GITHUB_OUTPUT
      
      # Boolean
      echo "success=true" >> $GITHUB_OUTPUT
    shell: bash

outputs:
  version:
    description: 'Version'
    value: ${{ steps.generate.outputs.version }}
  commit-sha:
    description: 'Commit'
    value: ${{ steps.generate.outputs.commit-sha }}
  build-time:
    description: 'Build time'
    value: ${{ steps.generate.outputs.build-time }}
  success:
    description: 'Success'
    value: ${{ steps.generate.outputs.success }}
```

### In JavaScript Actions

```javascript
const core = require('@actions/core');

// Set simple output
core.setOutput('version', '1.2.3');

// Set computed output
const commitSha = execSync('git rev-parse --short HEAD').toString().trim();
core.setOutput('commit-sha', commitSha);

// Set timestamp
const buildTime = new Date().toISOString();
core.setOutput('build-time', buildTime);

// Set boolean
core.setOutput('success', true);
```

### In Docker Actions (Python)

```python
import os
from datetime import datetime
import subprocess

def set_output(name, value):
    """Set GitHub Actions output."""
    with open(os.environ['GITHUB_OUTPUT'], 'a') as f:
        f.write(f"{name}={value}\n")

# Set outputs
set_output('version', '1.2.3')

# Computed output
commit_sha = subprocess.check_output(
    ['git', 'rev-parse', '--short', 'HEAD']
).decode().strip()
set_output('commit-sha', commit_sha)

# Timestamp
build_time = datetime.utcnow().isoformat() + 'Z'
set_output('build-time', build_time)

# Boolean
set_output('success', 'true')
```

## Multiline Outputs

### Setting Multiline Outputs

```yaml
steps:
  - name: Generate multiline output
    id: multiline
    run: |
      # Use delimiter for multiline content
      {
        echo "report<<EOF"
        echo "Line 1"
        echo "Line 2"
        echo "Line 3"
        echo "EOF"
      } >> $GITHUB_OUTPUT
    shell: bash

outputs:
  report:
    description: 'Multiline report'
    value: ${{ steps.multiline.outputs.report }}
```

### In JavaScript

```javascript
const core = require('@actions/core');

const report = `Line 1
Line 2
Line 3`;

core.setOutput('report', report);
```

### In Python

```python
def set_multiline_output(name, value):
    """Set multiline GitHub Actions output."""
    delimiter = 'EOF'
    with open(os.environ['GITHUB_OUTPUT'], 'a') as f:
        f.write(f"{name}<<{delimiter}\n")
        f.write(value)
        f.write(f"\n{delimiter}\n")

report = """Line 1
Line 2
Line 3"""

set_multiline_output('report', report)
```

## Using Outputs in Workflows

### Accessing Action Outputs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run action
        id: my-action
        uses: owner/action@v1
        with:
          input: 'value'
      
      - name: Use outputs
        run: |
          echo "Version: ${{ steps.my-action.outputs.version }}"
          echo "Commit: ${{ steps.my-action.outputs.commit-sha }}"
          echo "Success: ${{ steps.my-action.outputs.success }}"
      
      - name: Conditional on output
        if: steps.my-action.outputs.success == 'true'
        run: echo "Action succeeded"
```

### Passing Outputs Between Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.my-action.outputs.version }}
      commit: ${{ steps.my-action.outputs.commit-sha }}
    steps:
      - uses: actions/checkout@v3
      - id: my-action
        uses: owner/action@v1
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: |
          echo "Deploying version: ${{ needs.build.outputs.version }}"
          echo "Commit: ${{ needs.build.outputs.commit }}"
```

## Environment Variables

### Setting Environment Variables

```yaml
steps:
  - name: Set environment
    run: |
      echo "CUSTOM_VAR=value" >> $GITHUB_ENV
      echo "BUILD_ID=$(date +%s)" >> $GITHUB_ENV
    shell: bash
  
  - name: Use environment
    run: |
      echo "Custom: $CUSTOM_VAR"
      echo "Build ID: $BUILD_ID"
    shell: bash
```

### In JavaScript

```javascript
const core = require('@actions/core');

core.exportVariable('CUSTOM_VAR', 'value');
core.exportVariable('BUILD_ID', Date.now());
```

### In Python

```python
def set_env(name, value):
    """Set environment variable for subsequent steps."""
    with open(os.environ['GITHUB_ENV'], 'a') as f:
        f.write(f"{name}={value}\n")

set_env('CUSTOM_VAR', 'value')
set_env('BUILD_ID', str(int(time.time())))
```

## Secrets Handling

### Passing Secrets to Actions

```yaml
steps:
  - uses: owner/action@v1
    with:
      api-key: ${{ secrets.API_KEY }}
      token: ${{ secrets.GITHUB_TOKEN }}
```

### Using Secrets in Actions

**Composite:**
```yaml
steps:
  - name: Use secret
    run: |
      # Secret is available as input
      curl -H "Authorization: Bearer ${{ inputs.api-key }}" \
        https://api.example.com
    shell: bash
```

**JavaScript:**
```javascript
const apiKey = core.getInput('api-key', { required: true });
// Use apiKey (it won't be logged)
```

**Python:**
```python
api_key = os.environ.get('INPUT_API-KEY')
# Use api_key (it won't be logged)
```

### Masking Sensitive Values

```yaml
steps:
  - name: Mask value
    run: |
      SENSITIVE="secret-value"
      echo "::add-mask::$SENSITIVE"
      echo "Value: $SENSITIVE"  # Will be masked in logs
    shell: bash
```

**JavaScript:**
```javascript
const sensitive = 'secret-value';
core.setSecret(sensitive);
console.log(sensitive);  // Will be masked
```

## Advanced Patterns

### Dynamic Output Names

```yaml
steps:
  - name: Dynamic outputs
    id: dynamic
    run: |
      for i in 1 2 3; do
        echo "result-$i=value-$i" >> $GITHUB_OUTPUT
      done
    shell: bash
```

### JSON Outputs

```yaml
steps:
  - name: JSON output
    id: json
    run: |
      JSON='{"key":"value","number":42}'
      echo "data=$JSON" >> $GITHUB_OUTPUT
    shell: bash
  
  - name: Parse JSON
    run: |
      echo '${{ steps.json.outputs.data }}' | jq .
    shell: bash
```

### Conditional Outputs

```yaml
steps:
  - name: Conditional output
    id: conditional
    run: |
      if [ "${{ inputs.mode }}" = "production" ]; then
        echo "url=https://prod.example.com" >> $GITHUB_OUTPUT
      else
        echo "url=https://dev.example.com" >> $GITHUB_OUTPUT
      fi
    shell: bash
```

## Best Practices

1. **Validate inputs early**: Check all inputs before processing
2. **Provide defaults**: Use sensible defaults for optional inputs
3. **Clear descriptions**: Document what each input/output does
4. **Type conversion**: Handle string-to-type conversion properly
5. **Error messages**: Provide clear error messages for invalid inputs
6. **Mask secrets**: Always mask sensitive values in logs
7. **Document outputs**: Explain what each output contains
8. **Consistent naming**: Use kebab-case for input/output names

## Common Pitfalls

**Forgetting inputs are strings:**
```yaml
# Wrong
if: inputs.enable-feature == true

# Correct
if: inputs.enable-feature == 'true'
```

**Not validating inputs:**
```yaml
# Always validate before use
- name: Validate
  run: |
    if [ -z "${{ inputs.required-input }}" ]; then
      echo "::error::Input required"
      exit 1
    fi
  shell: bash
```

**Incorrect output reference:**
```yaml
# Wrong
value: ${{ outputs.result }}

# Correct
value: ${{ steps.step-id.outputs.result }}
```

**Not masking secrets:**
```yaml
# Always mask before logging
- run: |
    echo "::add-mask::${{ inputs.api-key }}"
  shell: bash
```

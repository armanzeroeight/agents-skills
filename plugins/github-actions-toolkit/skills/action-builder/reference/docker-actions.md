# Docker Actions

Docker actions run your code in a container, supporting any programming language and providing a consistent execution environment.

## Contents
- When to use Docker actions
- Structure and Dockerfile
- Inputs and outputs
- Entrypoint scripts
- Building and testing
- Performance optimization

## When to Use Docker Actions

**Best for:**
- Non-JavaScript languages (Python, Go, Ruby, etc.)
- Complex dependencies or specific runtime requirements
- Consistent environment across different runners
- Existing containerized tools

**Trade-offs:**
- Slower startup (container build/pull time)
- Larger action size
- More complex to maintain

**Examples:**
- Custom linting tools
- Data processing scripts
- Security scanners
- Code generators

## Structure and Files

### Required Files

```
action-name/
├── action.yml          # Action metadata
├── Dockerfile          # Container definition
├── entrypoint.sh       # Entry script (optional)
├── src/                # Source code
│   └── main.py
└── README.md
```

### action.yml Configuration

```yaml
name: 'Python Linter'
description: 'Run custom Python linting'
author: 'Your Name'

inputs:
  path:
    description: 'Path to Python files'
    required: true
    default: '.'
  config:
    description: 'Path to config file'
    required: false

outputs:
  issues-found:
    description: 'Number of issues found'

runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.path }}
    - ${{ inputs.config }}
```

## Dockerfile Patterns

### Basic Python Action

```dockerfile
FROM python:3.11-slim

# Install dependencies
COPY requirements.txt /requirements.txt
RUN pip install --no-cache-dir -r /requirements.txt

# Copy source code
COPY src/ /app/
WORKDIR /app

# Set entrypoint
ENTRYPOINT ["python", "/app/main.py"]
```

### Multi-stage Build

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /build
COPY . .
RUN go build -o /app/tool main.go

# Runtime stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
COPY --from=builder /app/tool /usr/local/bin/tool
ENTRYPOINT ["tool"]
```

### With Entrypoint Script

```dockerfile
FROM node:18-alpine

COPY package*.json ./
RUN npm ci --production

COPY . .

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

## Working with Inputs

### Accessing Inputs in Container

**Inputs are passed as:**
1. Command-line arguments (via `args` in action.yml)
2. Environment variables (prefixed with `INPUT_`)

**Example action.yml:**
```yaml
inputs:
  source-path:
    description: 'Source directory'
    required: true
  output-format:
    description: 'Output format'
    required: false
    default: 'json'

runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.source-path }}
    - ${{ inputs.output-format }}
```

**Accessing in Python:**
```python
import sys
import os

# From command-line arguments
source_path = sys.argv[1]
output_format = sys.argv[2]

# From environment variables
source_path = os.environ.get('INPUT_SOURCE-PATH')
output_format = os.environ.get('INPUT_OUTPUT-FORMAT', 'json')
```

**Accessing in entrypoint.sh:**
```bash
#!/bin/sh
set -e

SOURCE_PATH=$1
OUTPUT_FORMAT=$2

# Or from environment
SOURCE_PATH=${INPUT_SOURCE_PATH}
OUTPUT_FORMAT=${INPUT_OUTPUT_FORMAT:-json}

echo "Processing $SOURCE_PATH with format $OUTPUT_FORMAT"
```

## Setting Outputs

### From Python

```python
import os

def set_output(name, value):
    """Set GitHub Actions output."""
    with open(os.environ['GITHUB_OUTPUT'], 'a') as f:
        f.write(f"{name}={value}\n")

# Usage
issues_found = 5
set_output('issues-found', issues_found)
```

### From Shell Script

```bash
#!/bin/sh

ISSUES_FOUND=5
echo "issues-found=$ISSUES_FOUND" >> $GITHUB_OUTPUT
```

### From Go

```go
package main

import (
    "fmt"
    "os"
)

func setOutput(name, value string) error {
    f, err := os.OpenFile(os.Getenv("GITHUB_OUTPUT"), 
        os.O_APPEND|os.O_WRONLY, 0644)
    if err != nil {
        return err
    }
    defer f.Close()
    
    _, err = fmt.Fprintf(f, "%s=%s\n", name, value)
    return err
}

func main() {
    setOutput("issues-found", "5")
}
```

## Entrypoint Scripts

### Basic Entrypoint

```bash
#!/bin/sh
set -e  # Exit on error

# Parse inputs
SOURCE_PATH=$1
CONFIG_FILE=$2

# Validate inputs
if [ -z "$SOURCE_PATH" ]; then
    echo "::error::source-path is required"
    exit 1
fi

# Run main logic
python /app/main.py "$SOURCE_PATH" "$CONFIG_FILE"

# Set outputs
echo "status=success" >> $GITHUB_OUTPUT
```

### Advanced Entrypoint with Error Handling

```bash
#!/bin/bash
set -e

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

# Function to log errors
log_error() {
    echo "::error::$1"
    echo -e "${RED}Error: $1${NC}" >&2
}

# Function to log info
log_info() {
    echo "::notice::$1"
    echo -e "${GREEN}$1${NC}"
}

# Parse inputs
SOURCE_PATH=${INPUT_SOURCE_PATH:-$1}
CONFIG_FILE=${INPUT_CONFIG_FILE:-$2}

# Validate
if [ ! -d "$SOURCE_PATH" ]; then
    log_error "Source path does not exist: $SOURCE_PATH"
    exit 1
fi

# Run with error handling
if python /app/main.py "$SOURCE_PATH" "$CONFIG_FILE"; then
    log_info "Processing completed successfully"
    echo "status=success" >> $GITHUB_OUTPUT
else
    log_error "Processing failed"
    echo "status=failure" >> $GITHUB_OUTPUT
    exit 1
fi
```

## Working with GitHub Workspace

### Accessing Repository Files

```python
import os

# GitHub workspace is mounted at /github/workspace
workspace = os.environ.get('GITHUB_WORKSPACE', '/github/workspace')
file_path = os.path.join(workspace, 'src', 'main.py')

with open(file_path, 'r') as f:
    content = f.read()
```

### Writing Output Files

```python
import os

workspace = os.environ['GITHUB_WORKSPACE']
output_path = os.path.join(workspace, 'output.json')

with open(output_path, 'w') as f:
    f.write('{"status": "success"}')
```

## Building and Testing

### Local Testing

**Build the image:**
```bash
docker build -t my-action .
```

**Test with inputs:**
```bash
docker run --rm \
  -e INPUT_SOURCE_PATH=/workspace/src \
  -e INPUT_CONFIG_FILE=/workspace/config.yml \
  -v $(pwd):/workspace \
  my-action
```

**Test with GitHub environment:**
```bash
docker run --rm \
  -e GITHUB_WORKSPACE=/workspace \
  -e GITHUB_OUTPUT=/tmp/output.txt \
  -v $(pwd):/workspace \
  -v /tmp:/tmp \
  my-action /workspace/src config.yml
```

### Testing in Workflow

```yaml
name: Test Action

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Test action
        uses: ./
        with:
          source-path: './test-data'
          config-file: './test-config.yml'
      
      - name: Check outputs
        run: |
          echo "Issues found: ${{ steps.test.outputs.issues-found }}"
```

## Performance Optimization

### Use Smaller Base Images

```dockerfile
# Instead of
FROM python:3.11
# Use
FROM python:3.11-slim
# Or
FROM python:3.11-alpine
```

### Multi-stage Builds

```dockerfile
# Build dependencies in one stage
FROM python:3.11 AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Copy only what's needed to runtime
FROM python:3.11-slim
COPY --from=builder /root/.local /root/.local
COPY src/ /app/
ENV PATH=/root/.local/bin:$PATH
WORKDIR /app
ENTRYPOINT ["python", "main.py"]
```

### Layer Caching

```dockerfile
# Copy dependency files first (changes less frequently)
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy source code last (changes more frequently)
COPY src/ /app/
```

### Pre-built Images

```yaml
# Use pre-built image from registry
runs:
  using: 'docker'
  image: 'docker://ghcr.io/owner/action:v1'
```

## Complete Example: Python Linter Action

**action.yml:**
```yaml
name: 'Python Code Analyzer'
description: 'Analyze Python code for issues'
author: 'DevTools Team'

inputs:
  path:
    description: 'Path to Python files'
    required: true
    default: '.'
  severity:
    description: 'Minimum severity (info/warning/error)'
    required: false
    default: 'warning'

outputs:
  issues-found:
    description: 'Number of issues found'
  passed:
    description: 'Whether analysis passed'

runs:
  using: 'docker'
  image: 'Dockerfile'
```

**Dockerfile:**
```dockerfile
FROM python:3.11-slim

# Install analysis tools
RUN pip install --no-cache-dir \
    pylint==3.0.0 \
    flake8==6.1.0 \
    mypy==1.7.0

# Copy source
COPY src/ /app/
WORKDIR /app

# Set entrypoint
ENTRYPOINT ["python", "/app/analyze.py"]
```

**src/analyze.py:**
```python
#!/usr/bin/env python3
import os
import sys
import subprocess
from pathlib import Path

def set_output(name, value):
    """Set GitHub Actions output."""
    with open(os.environ['GITHUB_OUTPUT'], 'a') as f:
        f.write(f"{name}={value}\n")

def analyze(path, severity):
    """Run analysis tools."""
    issues = 0
    
    # Run pylint
    result = subprocess.run(
        ['pylint', path],
        capture_output=True,
        text=True
    )
    
    # Parse results
    for line in result.stdout.split('\n'):
        if severity in line.lower():
            issues += 1
            print(f"::warning::{line}")
    
    return issues

def main():
    # Get inputs
    path = os.environ.get('INPUT_PATH', '.')
    severity = os.environ.get('INPUT_SEVERITY', 'warning')
    
    # Validate
    if not Path(path).exists():
        print(f"::error::Path does not exist: {path}")
        sys.exit(1)
    
    # Run analysis
    issues = analyze(path, severity)
    
    # Set outputs
    set_output('issues-found', str(issues))
    set_output('passed', 'true' if issues == 0 else 'false')
    
    # Exit with appropriate code
    sys.exit(0 if issues == 0 else 1)

if __name__ == '__main__':
    main()
```

## Best Practices

1. **Use slim/alpine base images**: Reduce image size
2. **Multi-stage builds**: Separate build and runtime
3. **Layer caching**: Order Dockerfile for optimal caching
4. **Validate inputs**: Check required inputs early
5. **Error handling**: Proper exit codes and error messages
6. **Security**: Don't include secrets in image
7. **Documentation**: Clear README with examples
8. **Versioning**: Tag images with versions

## Common Issues

**Container fails to start:**
- Check Dockerfile syntax
- Verify entrypoint exists and is executable
- Test locally with `docker run`

**Inputs not available:**
- Check environment variable names (INPUT_ prefix)
- Verify args in action.yml
- Test with `-e` flags in docker run

**Outputs not set:**
- Ensure GITHUB_OUTPUT is written to
- Check file permissions
- Verify output format (name=value)

**Slow execution:**
- Use smaller base images
- Implement multi-stage builds
- Consider pre-built images
- Cache dependencies

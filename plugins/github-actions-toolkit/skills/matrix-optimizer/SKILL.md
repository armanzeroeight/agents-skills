---
name: matrix-optimizer
description: Optimize GitHub Actions matrix strategies for testing across multiple versions, platforms, and configurations. Use when configuring matrix builds, testing multiple versions, cross-platform testing, or optimizing CI resource usage. Trigger words include "matrix strategy", "test matrix", "multiple versions", "cross-platform".
---

# Matrix Optimizer

Configure and optimize GitHub Actions matrix strategies for efficient multi-version and multi-platform testing.

## Quick Start

Basic matrix for testing multiple Node.js versions:
```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
```

## Instructions

### Step 1: Identify Matrix Dimensions

**Common matrix dimensions:**
- **Language versions**: Node.js, Python, Ruby, Go versions
- **Operating systems**: ubuntu, macos, windows
- **Architectures**: x64, arm64
- **Dependency versions**: Database versions, framework versions
- **Feature flags**: Different configuration options

**Example dimensions:**
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
    node-version: [16, 18, 20]
    # This creates 9 jobs (3 OS × 3 versions)
```

### Step 2: Configure Matrix Strategy

**Basic matrix:**
```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        node-version: [18, 20]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

**Matrix with include:**
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest]
    node-version: [18, 20]
    include:
      # Add specific combination
      - os: windows-latest
        node-version: 20
      # Add extra variables for specific combination
      - os: ubuntu-latest
        node-version: 20
        experimental: true
```

**Matrix with exclude:**
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
    node-version: [16, 18, 20]
    exclude:
      # Skip Node 16 on Windows
      - os: windows-latest
        node-version: 16
      # Skip Node 16 on macOS
      - os: macos-latest
        node-version: 16
```

### Step 3: Optimize for Cost and Speed

**Fail-fast strategy:**
```yaml
strategy:
  fail-fast: false  # Continue all jobs even if one fails
  matrix:
    node-version: [16, 18, 20]
```

**Max parallel jobs:**
```yaml
strategy:
  max-parallel: 2  # Limit concurrent jobs
  matrix:
    node-version: [16, 18, 20]
```

**Conditional matrix:**
```yaml
strategy:
  matrix:
    os: [ubuntu-latest]
    # Add more OS only on main branch
    ${{ github.ref == 'refs/heads/main' && fromJSON('["macos-latest", "windows-latest"]') || fromJSON('[]') }}
```

### Step 4: Use Matrix Variables

**In job steps:**
```yaml
steps:
  - name: Display matrix values
    run: |
      echo "OS: ${{ matrix.os }}"
      echo "Version: ${{ matrix.node-version }}"
      echo "Experimental: ${{ matrix.experimental }}"
```

**In job configuration:**
```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    continue-on-error: ${{ matrix.experimental == true }}
    strategy:
      matrix:
        os: [ubuntu-latest]
        node-version: [18, 20]
        include:
          - node-version: 21
            experimental: true
```

### Step 5: Name Jobs Clearly

```yaml
jobs:
  test:
    name: Test on ${{ matrix.os }} with Node ${{ matrix.node-version }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        node-version: [18, 20]
```

## Common Patterns

### Language Version Matrix

**Node.js:**
```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
steps:
  - uses: actions/setup-node@v3
    with:
      node-version: ${{ matrix.node-version }}
```

**Python:**
```yaml
strategy:
  matrix:
    python-version: ['3.9', '3.10', '3.11', '3.12']
steps:
  - uses: actions/setup-python@v4
    with:
      python-version: ${{ matrix.python-version }}
```

**Go:**
```yaml
strategy:
  matrix:
    go-version: ['1.20', '1.21', '1.22']
steps:
  - uses: actions/setup-go@v4
    with:
      go-version: ${{ matrix.go-version }}
```

### Cross-Platform Matrix

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest, windows-latest]
    include:
      # Platform-specific configurations
      - os: ubuntu-latest
        install-cmd: sudo apt-get install
      - os: macos-latest
        install-cmd: brew install
      - os: windows-latest
        install-cmd: choco install

steps:
  - name: Install dependencies
    run: ${{ matrix.install-cmd }} package-name
```

### Database Version Matrix

```yaml
strategy:
  matrix:
    postgres-version: [12, 13, 14, 15]

services:
  postgres:
    image: postgres:${{ matrix.postgres-version }}
    env:
      POSTGRES_PASSWORD: postgres
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5
```

### Feature Flag Matrix

```yaml
strategy:
  matrix:
    feature:
      - name: baseline
        flags: ''
      - name: new-parser
        flags: '--enable-new-parser'
      - name: experimental
        flags: '--enable-experimental'

steps:
  - name: Run tests
    run: npm test ${{ matrix.feature.flags }}
```

## Optimization Strategies

### Reduce Matrix Size

**Before (12 jobs):**
```yaml
matrix:
  os: [ubuntu-latest, macos-latest, windows-latest]
  node-version: [16, 18, 20, 21]
```

**After (7 jobs):**
```yaml
matrix:
  # Test all versions on Linux only
  os: [ubuntu-latest]
  node-version: [16, 18, 20, 21]
  include:
    # Test latest version on other platforms
    - os: macos-latest
      node-version: 21
    - os: windows-latest
      node-version: 21
```

### Conditional Matrix Expansion

```yaml
strategy:
  matrix:
    # Always test on Linux
    os: [ubuntu-latest]
    node-version: [18, 20]
    include:
      # Full matrix only on main branch or release tags
      - ${{ github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/') }}:
        os: [macos-latest, windows-latest]
```

### Parallel vs Sequential

**High parallelism (faster, more expensive):**
```yaml
strategy:
  matrix:
    shard: [1, 2, 3, 4, 5, 6, 7, 8]
steps:
  - run: npm test -- --shard=${{ matrix.shard }}/8
```

**Limited parallelism (slower, cheaper):**
```yaml
strategy:
  max-parallel: 2
  matrix:
    shard: [1, 2, 3, 4, 5, 6, 7, 8]
```

### Caching Across Matrix

```yaml
steps:
  - uses: actions/cache@v3
    with:
      path: ~/.npm
      key: ${{ runner.os }}-node-${{ matrix.node-version }}-${{ hashFiles('**/package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-node-${{ matrix.node-version }}-
        ${{ runner.os }}-node-
```

## Advanced Patterns

### Dynamic Matrix from JSON

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: |
          # Generate matrix dynamically
          MATRIX='{"include":[{"os":"ubuntu-latest","version":"18"},{"os":"macos-latest","version":"20"}]}'
          echo "matrix=$MATRIX" >> $GITHUB_OUTPUT
  
  test:
    needs: setup
    strategy:
      matrix: ${{ fromJSON(needs.setup.outputs.matrix) }}
    runs-on: ${{ matrix.os }}
    steps:
      - run: echo "Testing on ${{ matrix.os }} with version ${{ matrix.version }}"
```

### Matrix with Outputs

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
    outputs:
      result-${{ matrix.os }}: ${{ steps.test.outputs.result }}
    steps:
      - id: test
        run: echo "result=passed" >> $GITHUB_OUTPUT
```

### Reusable Matrix Workflow

```yaml
# .github/workflows/reusable-matrix.yml
on:
  workflow_call:
    inputs:
      versions:
        required: true
        type: string

jobs:
  test:
    strategy:
      matrix:
        version: ${{ fromJSON(inputs.versions) }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing version ${{ matrix.version }}"

# Caller workflow
jobs:
  test:
    uses: ./.github/workflows/reusable-matrix.yml
    with:
      versions: '["18", "20", "21"]'
```

## Troubleshooting

**Too many jobs:**
- Use `exclude` to remove unnecessary combinations
- Test all versions on one OS, latest version on others
- Use conditional matrix expansion for PRs vs main

**Jobs failing inconsistently:**
- Set `fail-fast: false` to see all failures
- Check for race conditions or timing issues
- Verify platform-specific dependencies

**Slow matrix execution:**
- Increase `max-parallel` if budget allows
- Optimize caching strategy
- Consider test sharding within jobs

**Matrix not expanding:**
- Verify JSON syntax in `fromJSON()`
- Check that matrix variables are properly referenced
- Ensure `include` and `exclude` syntax is correct

## Best Practices

1. **Start small**: Begin with minimal matrix, expand as needed
2. **Test locally first**: Verify one configuration works before expanding
3. **Use fail-fast: false**: See all failures, not just first
4. **Name jobs clearly**: Include matrix values in job names
5. **Cache effectively**: Use matrix values in cache keys
6. **Optimize for PRs**: Smaller matrix for PRs, full matrix for main
7. **Document matrix**: Explain why each dimension is needed
8. **Monitor costs**: Track runner minutes usage

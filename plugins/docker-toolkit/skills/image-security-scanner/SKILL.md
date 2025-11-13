---
name: image-security-scanner
description: Scans Docker images for security vulnerabilities, outdated packages, and misconfigurations. Use when checking image security, finding vulnerabilities, or hardening containers.
---

# Image Security Scanner

Scan and secure Docker images for production deployment.

## Quick Start

Scan an image:
```bash
docker scan myapp:latest
# or
trivy image myapp:latest
```

## Instructions

### Step 1: Choose Scanning Tool

**Docker Scan** (built-in):
```bash
docker scan myapp:latest
```

**Trivy** (comprehensive):
```bash
trivy image myapp:latest
```

**Grype** (fast):
```bash
grype myapp:latest
```

**Snyk** (detailed):
```bash
snyk container test myapp:latest
```

### Step 2: Run Security Scan

**Basic scan**:
```bash
docker scan myapp:latest
```

**Detailed scan with Trivy**:
```bash
trivy image --severity HIGH,CRITICAL myapp:latest
```

**Scan with JSON output**:
```bash
trivy image -f json -o results.json myapp:latest
```

### Step 3: Analyze Results

Review findings by severity:
- **CRITICAL**: Immediate action required
- **HIGH**: Fix soon
- **MEDIUM**: Plan to fix
- **LOW**: Monitor

**Common vulnerabilities**:
- Outdated base image
- Vulnerable packages
- Known CVEs
- Misconfigurations

### Step 4: Fix Vulnerabilities

**Update base image**:
```dockerfile
# Before
FROM node:18-alpine3.17

# After
FROM node:18-alpine3.18
```

**Update packages**:
```dockerfile
RUN apk upgrade --no-cache
# or
RUN apt-get update && apt-get upgrade -y
```

**Remove vulnerable packages**:
```dockerfile
RUN apk del vulnerable-package
```

**Use distroless for minimal attack surface**:
```dockerfile
FROM gcr.io/distroless/nodejs18-debian11
```

### Step 5: Implement Security Best Practices

**Run as non-root**:
```dockerfile
USER nobody
# or
RUN adduser -D appuser
USER appuser
```

**Remove unnecessary tools**:
```dockerfile
RUN apk del apk-tools
```

**Use read-only filesystem**:
```dockerfile
# In docker-compose or k8s
read_only: true
```

**Add security labels**:
```dockerfile
LABEL security.scan-date="2024-01-15"
LABEL security.scanner="trivy"
```

### Step 6: Verify Fixes

Re-scan after fixes:
```bash
docker build -t myapp:latest .
trivy image myapp:latest
```

Compare before/after:
```bash
# Before: 15 HIGH, 5 CRITICAL
# After: 2 HIGH, 0 CRITICAL
```

## Scanning Patterns

**CI/CD Integration**:
```yaml
# GitHub Actions
- name: Scan image
  run: |
    docker build -t myapp:${{ github.sha }} .
    trivy image --exit-code 1 --severity CRITICAL myapp:${{ github.sha }}
```

**Pre-deployment scan**:
```bash
#!/bin/bash
IMAGE=$1
trivy image --severity HIGH,CRITICAL $IMAGE
if [ $? -ne 0 ]; then
  echo "Security vulnerabilities found!"
  exit 1
fi
```

**Scheduled scans**:
```bash
# Cron job to scan running images
0 2 * * * trivy image --severity HIGH,CRITICAL $(docker images -q)
```

## Security Hardening

**Minimal base image**:
```dockerfile
FROM alpine:3.18
# or
FROM gcr.io/distroless/static-debian11
```

**No secrets in image**:
```dockerfile
# Bad
ENV API_KEY=secret123

# Good
# Pass at runtime
docker run -e API_KEY=$API_KEY myapp
```

**Health checks**:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1
```

**Limit capabilities**:
```bash
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp
```

## Common Vulnerabilities

**Outdated base image**:
```dockerfile
# Vulnerable
FROM node:16-alpine

# Fixed
FROM node:18-alpine3.18
```

**Exposed secrets**:
```dockerfile
# Vulnerable
COPY .env .

# Fixed
# Use runtime secrets
```

**Running as root**:
```dockerfile
# Vulnerable
CMD ["node", "server.js"]

# Fixed
USER node
CMD ["node", "server.js"]
```

**Unnecessary packages**:
```dockerfile
# Vulnerable
RUN apk add curl wget git vim

# Fixed
RUN apk add --no-cache curl
```

## Scanning Tools Comparison

**Docker Scan**:
- Built into Docker
- Uses Snyk backend
- Easy to use
- Limited free scans

**Trivy**:
- Open source
- Fast and accurate
- Multiple output formats
- CI/CD friendly

**Grype**:
- Open source
- Very fast
- Good accuracy
- Simple CLI

**Snyk**:
- Commercial (free tier)
- Detailed reports
- Fix recommendations
- IDE integration

## Advanced

For production deployments:
- Implement image signing
- Use admission controllers
- Set up continuous scanning
- Monitor runtime security
- Implement security policies

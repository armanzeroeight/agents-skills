---
name: dockerfile-optimizer
description: Analyzes and optimizes Dockerfiles to reduce image size, improve build time, and enhance security. Use when optimizing Docker images, reducing build times, or improving Dockerfile structure.
---

# Dockerfile Optimizer

Optimize Docker images for size, speed, and security.

## Quick Start

Analyze current Dockerfile:
```bash
docker build -t myapp .
docker images myapp
# Note the size, then optimize
```

## Instructions

### Step 1: Analyze Current Dockerfile

Review for optimization opportunities:
- Base image choice
- Layer structure
- Build dependencies
- Caching strategy
- Security practices

### Step 2: Optimize Base Image

**Choose minimal base**:
```dockerfile
# Before: Large base
FROM ubuntu:22.04  # ~77MB

# After: Minimal base
FROM alpine:3.18   # ~7MB
# or
FROM node:18-alpine  # ~170MB vs node:18 ~990MB
```

**Use specific tags**:
```dockerfile
# Bad: Unpredictable
FROM node:latest

# Good: Specific version
FROM node:18.17-alpine3.18
```

### Step 3: Implement Multi-Stage Builds

**For compiled languages**:
```dockerfile
# Before: Single stage
FROM golang:1.21
WORKDIR /app
COPY . .
RUN go build -o app
CMD ["./app"]
# Result: ~800MB

# After: Multi-stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM alpine:3.18
COPY --from=builder /app/app /app
CMD ["/app"]
# Result: ~15MB
```

### Step 4: Optimize Layer Caching

**Order by change frequency**:
```dockerfile
# Bad: Code changes invalidate all layers
FROM node:18-alpine
COPY . .
RUN npm install
CMD ["node", "server.js"]

# Good: Dependencies cached separately
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]
```

### Step 5: Combine and Minimize Layers

**Reduce layer count**:
```dockerfile
# Before: Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean

# After: Single layer
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Step 6: Use .dockerignore

Create `.dockerignore`:
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
*.md
.vscode
.idea
dist
build
coverage
```

### Step 7: Remove Build Dependencies

**Clean up after install**:
```dockerfile
# Python example
RUN pip install --no-cache-dir -r requirements.txt

# Node example
RUN npm ci --only=production && npm cache clean --force

# Alpine example
RUN apk add --no-cache curl && \
    apk del build-dependencies
```

### Step 8: Add Security Improvements

**Run as non-root**:
```dockerfile
# Create user
RUN addgroup -g 1001 -S appuser && \
    adduser -u 1001 -S appuser -G appuser

# Switch to user
USER appuser

# Or use existing user
USER nobody
```

### Step 9: Verify Optimization

**Check image size**:
```bash
docker images myapp
docker history myapp
```

**Analyze layers**:
```bash
docker history myapp --no-trunc
```

**Use dive tool**:
```bash
dive myapp
```

## Optimization Patterns

**Node.js Optimization**:
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
USER node
EXPOSE 3000
CMD ["node", "server.js"]
```

**Python Optimization**:
```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
USER nobody
CMD ["python", "app.py"]
```

**Go Optimization**:
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM scratch
COPY --from=builder /app/app /app
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
EXPOSE 8080
CMD ["/app"]
```

## Common Issues

**Issue: Large image size**
- Use alpine or slim base images
- Implement multi-stage builds
- Remove build dependencies
- Use .dockerignore

**Issue: Slow builds**
- Optimize layer caching
- Order commands by change frequency
- Use build cache mounts
- Parallelize where possible

**Issue: Security vulnerabilities**
- Use minimal base images
- Keep images updated
- Run as non-root
- Scan with security tools

**Issue: Build cache not working**
- Check command order
- Avoid COPY . . early
- Use specific COPY commands
- Check .dockerignore

## Advanced

For detailed information, see:
- [Multi-Stage Builds](reference/multi-stage.md) - Advanced multi-stage patterns
- [Layer Caching](reference/caching.md) - Caching strategies and optimization
- [Base Images](reference/base-images.md) - Base image selection guide

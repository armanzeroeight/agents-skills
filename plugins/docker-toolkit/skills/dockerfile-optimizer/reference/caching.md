# Layer Caching Strategies

## Contents
- How Docker Caching Works
- Optimization Techniques
- Cache Invalidation Patterns
- Build Cache Mounts
- Best Practices

## How Docker Caching Works

Docker caches each layer during build. If a layer hasn't changed, Docker reuses the cached version.

**Cache invalidation**: When a layer changes, all subsequent layers are rebuilt.

**Example**:
```dockerfile
FROM node:18-alpine        # Layer 1: Cached (base image)
WORKDIR /app               # Layer 2: Cached (no change)
COPY package*.json ./      # Layer 3: Cached if files unchanged
RUN npm install            # Layer 4: Cached if Layer 3 cached
COPY . .                   # Layer 5: Invalidated (code changed)
RUN npm run build          # Layer 6: Rebuilt (Layer 5 changed)
```

## Optimization Techniques

### Order by Change Frequency

**Bad**: Code changes invalidate dependency install
```dockerfile
FROM node:18-alpine
COPY . .                   # Changes frequently
RUN npm install            # Rebuilt every time
CMD ["node", "server.js"]
```

**Good**: Dependencies cached separately
```dockerfile
FROM node:18-alpine
COPY package*.json ./      # Changes rarely
RUN npm install            # Cached most of the time
COPY . .                   # Changes frequently
CMD ["node", "server.js"]
```

### Separate Dependency Files

**Node.js**:
```dockerfile
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
```

**Python**:
```dockerfile
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY . .
```

**Go**:
```dockerfile
COPY go.mod go.sum ./
RUN go mod download
COPY . .
```

**Java (Maven)**:
```dockerfile
COPY pom.xml ./
RUN mvn dependency:go-offline
COPY src ./src
```

### Use Specific COPY Commands

**Bad**: Copies everything, invalidates cache often
```dockerfile
COPY . .
```

**Good**: Copy only what's needed
```dockerfile
COPY package*.json ./
COPY src ./src
COPY public ./public
```

### Combine Related Commands

**Bad**: Multiple layers
```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
```

**Good**: Single layer
```dockerfile
RUN apt-get update && \
    apt-get install -y curl git && \
    rm -rf /var/lib/apt/lists/*
```

## Cache Invalidation Patterns

### Pattern 1: Dependency Changes

When `package.json` changes, npm install runs again:
```dockerfile
COPY package*.json ./      # Cache invalidated here
RUN npm install            # Runs again
COPY . .                   # Also runs again
```

### Pattern 2: Code Changes

When source code changes, only code copy and subsequent layers rebuild:
```dockerfile
COPY package*.json ./      # Still cached
RUN npm install            # Still cached
COPY . .                   # Cache invalidated here
RUN npm run build          # Runs again
```

### Pattern 3: Base Image Updates

When base image updates, entire build runs:
```dockerfile
FROM node:18-alpine        # New version available
# All subsequent layers rebuild
```

## Build Cache Mounts

BuildKit provides cache mounts for package managers.

### NPM Cache Mount

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci
COPY . .
```

### Pip Cache Mount

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt ./
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
COPY . .
```

### Go Module Cache

```dockerfile
FROM golang:1.21-alpine
WORKDIR /app
COPY go.* ./
RUN --mount=type=cache,target=/go/pkg/mod \
    go mod download
COPY . .
```

### Apt Cache Mount

```dockerfile
FROM ubuntu:22.04
RUN --mount=type=cache,target=/var/cache/apt \
    --mount=type=cache,target=/var/lib/apt \
    apt-get update && \
    apt-get install -y curl git
```

### Maven Cache Mount

```dockerfile
FROM maven:3.9-eclipse-temurin-17
WORKDIR /app
COPY pom.xml ./
RUN --mount=type=cache,target=/root/.m2 \
    mvn dependency:go-offline
COPY src ./src
```

## Advanced Caching

### Multi-Stage with Shared Cache

```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

### Conditional Caching

```dockerfile
FROM node:18-alpine
ARG SKIP_CACHE=false
WORKDIR /app
COPY package*.json ./
RUN if [ "$SKIP_CACHE" = "true" ]; then \
      npm install --no-cache; \
    else \
      npm ci; \
    fi
COPY . .
```

Build with cache skip:
```bash
docker build --build-arg SKIP_CACHE=true -t myapp .
```

### External Cache Sources

Use registry as cache:
```bash
docker build \
  --cache-from myregistry/myapp:latest \
  --build-arg BUILDKIT_INLINE_CACHE=1 \
  -t myapp .
```

## Best Practices

### 1. Order Dockerfile Instructions

**Optimal order**:
1. Base image (FROM)
2. System packages (RUN apt-get)
3. Application dependencies (COPY package.json, RUN npm install)
4. Application code (COPY . .)
5. Build steps (RUN npm run build)
6. Configuration (ENV, EXPOSE)
7. Entry point (CMD, ENTRYPOINT)

### 2. Use .dockerignore

Prevent unnecessary cache invalidation:
```
node_modules
.git
.env
*.md
.vscode
dist
build
coverage
```

### 3. Leverage BuildKit

Enable BuildKit for better caching:
```bash
export DOCKER_BUILDKIT=1
docker build .
```

### 4. Pin Versions

**Bad**: Unpredictable caching
```dockerfile
FROM node:18
RUN npm install express
```

**Good**: Predictable caching
```dockerfile
FROM node:18.17-alpine3.18
COPY package-lock.json ./
RUN npm ci
```

### 5. Separate Build and Runtime Dependencies

```dockerfile
FROM node:18-alpine AS builder
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
```

## Debugging Cache Issues

### Check Layer Cache Status

```bash
docker build --progress=plain .
```

Look for:
- `CACHED` - Layer was reused
- `RUN` - Layer was rebuilt

### Inspect Build History

```bash
docker history myapp
```

### Force Rebuild

```bash
docker build --no-cache -t myapp .
```

### Rebuild from Specific Layer

```bash
docker build --target builder -t myapp:builder .
```

## Common Pitfalls

### Pitfall 1: COPY . . Too Early

**Problem**: Every code change rebuilds dependencies
```dockerfile
COPY . .
RUN npm install
```

**Solution**: Copy dependencies first
```dockerfile
COPY package*.json ./
RUN npm install
COPY . .
```

### Pitfall 2: Not Using .dockerignore

**Problem**: Unnecessary files invalidate cache
- `.git` directory changes
- `node_modules` copied then reinstalled
- Build artifacts included

**Solution**: Create comprehensive .dockerignore

### Pitfall 3: Combining Unrelated Commands

**Problem**: One change rebuilds everything
```dockerfile
RUN npm install && npm run build && npm test
```

**Solution**: Separate into logical layers
```dockerfile
RUN npm install
RUN npm run build
RUN npm test
```

### Pitfall 4: Not Pinning Versions

**Problem**: Unpredictable builds
```dockerfile
FROM node:latest
RUN npm install express
```

**Solution**: Pin all versions
```dockerfile
FROM node:18.17-alpine3.18
COPY package-lock.json ./
RUN npm ci
```

## Performance Metrics

### Measure Build Time

```bash
time docker build -t myapp .
```

### Compare Cache Effectiveness

**First build** (no cache):
```bash
docker build --no-cache -t myapp .
# Time: 5 minutes
```

**Second build** (with cache):
```bash
docker build -t myapp .
# Time: 10 seconds
```

### Analyze Layer Sizes

```bash
docker history myapp --no-trunc
```

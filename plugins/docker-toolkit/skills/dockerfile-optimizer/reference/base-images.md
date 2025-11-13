# Base Image Selection Guide

## Contents
- Image Variants Comparison
- Language-Specific Recommendations
- Security Considerations
- Size vs Compatibility Trade-offs
- Custom Base Images

## Image Variants Comparison

### Alpine Linux

**Size**: ~7MB base
**Package Manager**: apk
**Libc**: musl

**Pros**:
- Minimal size
- Security-focused
- Fast downloads
- Regular updates

**Cons**:
- musl libc compatibility issues
- Smaller package repository
- Some binaries need recompilation
- Different shell (ash vs bash)

**Best for**:
- Production deployments
- Microservices
- Size-critical applications
- Security-conscious projects

**Example**:
```dockerfile
FROM node:18-alpine
FROM python:3.11-alpine
FROM golang:1.21-alpine
```

### Debian/Ubuntu Slim

**Size**: ~30-80MB base
**Package Manager**: apt
**Libc**: glibc

**Pros**:
- Wide compatibility
- Large package repository
- Familiar tooling
- Better binary compatibility

**Cons**:
- Larger size
- More attack surface
- Slower downloads

**Best for**:
- Complex dependencies
- Development environments
- Legacy applications
- Maximum compatibility

**Example**:
```dockerfile
FROM node:18-slim
FROM python:3.11-slim
FROM ubuntu:22.04
```

### Distroless

**Size**: ~20-50MB
**Package Manager**: None
**Libc**: glibc

**Pros**:
- No package manager
- No shell
- Minimal attack surface
- Security-focused

**Cons**:
- Debugging difficult
- No shell access
- Limited tooling
- Requires multi-stage builds

**Best for**:
- Production deployments
- Maximum security
- Static binaries
- Compliance requirements

**Example**:
```dockerfile
FROM gcr.io/distroless/static-debian11
FROM gcr.io/distroless/base-debian11
FROM gcr.io/distroless/nodejs18-debian11
```

### Scratch

**Size**: 0MB (empty)
**Package Manager**: None
**Libc**: None

**Pros**:
- Absolute minimal size
- Maximum security
- No dependencies
- Perfect for static binaries

**Cons**:
- Only for static binaries
- No debugging tools
- No shell
- No CA certificates

**Best for**:
- Go applications
- Rust applications
- Static C/C++ binaries
- Minimal containers

**Example**:
```dockerfile
FROM scratch
COPY app /app
CMD ["/app"]
```

## Language-Specific Recommendations

### Node.js

**Production** (size priority):
```dockerfile
FROM node:18-alpine
# Size: ~170MB
```

**Production** (compatibility priority):
```dockerfile
FROM node:18-slim
# Size: ~240MB
```

**Development**:
```dockerfile
FROM node:18
# Size: ~990MB (includes build tools)
```

**Maximum security**:
```dockerfile
FROM gcr.io/distroless/nodejs18-debian11
# Size: ~180MB
```

### Python

**Production** (size priority):
```dockerfile
FROM python:3.11-alpine
# Size: ~50MB
```

**Production** (compatibility priority):
```dockerfile
FROM python:3.11-slim
# Size: ~120MB
```

**With compiled dependencies**:
```dockerfile
FROM python:3.11-slim
# Alpine may have issues with numpy, pandas, etc.
```

**Maximum security**:
```dockerfile
FROM gcr.io/distroless/python3-debian11
# Size: ~50MB
```

### Go

**Production** (minimal):
```dockerfile
FROM scratch
# Size: ~app size only (5-20MB)
```

**With CA certificates**:
```dockerfile
FROM gcr.io/distroless/static-debian11
# Size: ~2MB + app
```

**With libc**:
```dockerfile
FROM gcr.io/distroless/base-debian11
# Size: ~20MB + app
```

**Development**:
```dockerfile
FROM golang:1.21-alpine
# Size: ~300MB
```

### Java

**Production** (JRE only):
```dockerfile
FROM eclipse-temurin:17-jre-alpine
# Size: ~170MB
```

**Production** (compatibility):
```dockerfile
FROM eclipse-temurin:17-jre
# Size: ~270MB
```

**Development** (JDK):
```dockerfile
FROM eclipse-temurin:17-jdk
# Size: ~450MB
```

**Maximum security**:
```dockerfile
FROM gcr.io/distroless/java17-debian11
# Size: ~200MB
```

### Rust

**Production**:
```dockerfile
FROM scratch
# or
FROM gcr.io/distroless/static-debian11
# Size: ~app size only
```

**Development**:
```dockerfile
FROM rust:1.73-alpine
# Size: ~600MB
```

## Security Considerations

### Vulnerability Scanning

**Scan base images**:
```bash
docker scan node:18-alpine
docker scan node:18-slim
```

**Compare vulnerabilities**:
- Alpine typically has fewer CVEs
- Distroless has minimal attack surface
- Regular images have more packages = more vulnerabilities

### Security Best Practices

1. **Use specific tags**:
```dockerfile
# Bad
FROM node:latest

# Good
FROM node:18.17-alpine3.18
```

2. **Minimal base**:
```dockerfile
# Better security
FROM alpine:3.18
# vs
FROM ubuntu:22.04
```

3. **Regular updates**:
```bash
docker pull node:18-alpine
docker build --no-cache -t myapp .
```

4. **Run as non-root**:
```dockerfile
FROM node:18-alpine
USER node
```

## Size vs Compatibility Trade-offs

### Size Priority

**Smallest to largest**:
1. Scratch (0MB) - Static binaries only
2. Alpine (~7MB base) - Most languages
3. Distroless (~20MB) - Security focus
4. Slim (~80MB) - Good balance
5. Full (~500MB+) - Maximum compatibility

**Example progression**:
```dockerfile
# Smallest: 15MB total
FROM scratch
COPY app /app

# Small: 25MB total
FROM alpine:3.18
COPY app /app

# Medium: 100MB total
FROM node:18-alpine
COPY . .

# Large: 1GB total
FROM node:18
COPY . .
```

### Compatibility Priority

**Most compatible to least**:
1. Full Debian/Ubuntu - All packages available
2. Slim variants - Most packages available
3. Alpine - Limited packages, musl libc
4. Distroless - No package manager
5. Scratch - No OS

**When to choose compatibility**:
- Complex native dependencies
- Legacy applications
- Development environments
- Unfamiliar with Alpine quirks

### Balanced Approach

**Multi-stage with different bases**:
```dockerfile
# Build with full image
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Run with minimal image
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

## Custom Base Images

### Creating Custom Base

**For organization-wide use**:
```dockerfile
FROM node:18-alpine
RUN apk add --no-cache \
    curl \
    ca-certificates \
    && addgroup -g 1001 -S appuser \
    && adduser -u 1001 -S appuser -G appuser
USER appuser
WORKDIR /app
```

Save as `myorg/node:18-alpine-base`

**Use in applications**:
```dockerfile
FROM myorg/node:18-alpine-base
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["node", "server.js"]
```

### Benefits of Custom Base

- Consistent across projects
- Pre-installed common tools
- Security hardening applied once
- Faster builds (shared layers)
- Organizational standards

## Decision Matrix

| Requirement | Recommended Base |
|-------------|------------------|
| Smallest size | scratch or alpine |
| Maximum security | distroless or scratch |
| Best compatibility | slim or full |
| Python with numpy | slim (not alpine) |
| Go static binary | scratch |
| Node.js production | alpine |
| Java production | jre-alpine |
| Development | full variant |
| Legacy app | full debian/ubuntu |

## Migration Strategies

### From Full to Alpine

1. **Test compatibility**:
```dockerfile
FROM node:18-alpine
# Test if app works
```

2. **Handle musl issues**:
```dockerfile
FROM node:18-alpine
RUN apk add --no-cache libc6-compat
```

3. **Install missing packages**:
```dockerfile
FROM node:18-alpine
RUN apk add --no-cache python3 make g++
```

### From Alpine to Distroless

1. **Use multi-stage**:
```dockerfile
FROM node:18-alpine AS builder
# Build here

FROM gcr.io/distroless/nodejs18-debian11
COPY --from=builder /app /app
```

2. **No shell access**:
- Debug in builder stage
- Use exec form for CMD
- No runtime package installation

## Best Practices

1. **Start with alpine**: Try alpine first, fall back to slim if issues
2. **Use specific tags**: Never use `latest`
3. **Multi-stage builds**: Build with full, run with minimal
4. **Security scan**: Regularly scan base images
5. **Update regularly**: Keep base images current
6. **Document choice**: Explain why specific base chosen
7. **Test thoroughly**: Ensure compatibility before production

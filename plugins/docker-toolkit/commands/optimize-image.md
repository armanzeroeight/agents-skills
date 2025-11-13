---
description: Analyze Dockerfile and provide optimization recommendations for size, speed, and security
allowed-tools: Read, Bash(docker:*), Write
argument-hint: [dockerfile-path]
---

# Optimize Image

## Context

- Dockerfile location: !`find . -name "Dockerfile" -o -name "dockerfile" | head -5`
- Current images: !`docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | head -10`
- Docker version: !`docker --version`

## Your Task

Analyze the Dockerfile and provide comprehensive optimization recommendations.

### Arguments

- `$1` (optional): Path to Dockerfile (default: `./Dockerfile`)

### Steps

1. **Read and analyze Dockerfile**:
   ```bash
   cat Dockerfile
   ```

2. **Identify optimization opportunities**:

   **Base Image**:
   - Is alpine or slim variant used?
   - Is specific tag used (not `latest`)?
   - Can distroless be used?

   **Multi-Stage Build**:
   - Is multi-stage build used?
   - Are build dependencies separated from runtime?
   - Can final image be smaller?

   **Layer Optimization**:
   - Are dependencies copied before code?
   - Are RUN commands combined appropriately?
   - Is .dockerignore present?

   **Security**:
   - Is non-root user used?
   - Are secrets avoided in image?
   - Are unnecessary packages removed?

   **Caching**:
   - Are layers ordered by change frequency?
   - Are package manager caches cleared?
   - Can build cache mounts be used?

3. **Calculate current image size**:
   ```bash
   docker build -t temp-analysis .
   docker images temp-analysis --format "{{.Size}}"
   docker rmi temp-analysis
   ```

4. **Generate optimized Dockerfile**:

   Create `Dockerfile.optimized` with improvements:
   - Minimal base image
   - Multi-stage build (if applicable)
   - Optimized layer order
   - Security hardening
   - Cache optimization

5. **Provide comparison**:

   **Before**:
   - Image size: X MB
   - Layers: Y
   - Security issues: Z

   **After** (estimated):
   - Image size: A MB (B% reduction)
   - Layers: C
   - Security improvements: D

6. **Create optimization report**:

   ```markdown
   # Dockerfile Optimization Report

   ## Current Analysis
   - Base image: node:18 (990MB)
   - Build type: Single-stage
   - Security: Running as root
   - Layers: 12

   ## Recommendations

   ### High Priority
   1. Use alpine base (node:18-alpine)
      - Savings: ~800MB
      - Impact: High

   2. Implement multi-stage build
      - Savings: ~200MB
      - Impact: High

   3. Run as non-root user
      - Security: Critical
      - Impact: High

   ### Medium Priority
   4. Optimize layer caching
      - Build time: -50%
      - Impact: Medium

   5. Add .dockerignore
      - Build time: -20%
      - Impact: Medium

   ### Low Priority
   6. Combine RUN commands
      - Savings: ~10MB
      - Impact: Low

   ## Implementation

   See Dockerfile.optimized for complete implementation.

   ## Next Steps
   1. Review Dockerfile.optimized
   2. Test build: `docker build -f Dockerfile.optimized -t myapp:optimized .`
   3. Compare sizes: `docker images myapp`
   4. Test functionality
   5. Replace Dockerfile when satisfied
   ```

### Examples

**Optimize current Dockerfile**:
```
/optimize-image
```

**Optimize specific Dockerfile**:
```
/optimize-image docker/Dockerfile.prod
```

### Output Format

```
🔍 Analyzing Dockerfile...
==========================

Current Configuration:
  Base: node:18 (990MB)
  Type: Single-stage
  User: root
  Layers: 12

Optimization Opportunities:
  🔴 HIGH: Use alpine base (-800MB)
  🔴 HIGH: Multi-stage build (-200MB)
  🔴 HIGH: Run as non-root (security)
  🟡 MEDIUM: Optimize caching (-50% build time)
  🟢 LOW: Combine RUN commands (-10MB)

Estimated Results:
  Current: 1.2GB
  Optimized: 180MB
  Savings: 85%

✅ Created Dockerfile.optimized

Next Steps:
  1. Review: cat Dockerfile.optimized
  2. Build: docker build -f Dockerfile.optimized -t myapp:opt .
  3. Test: docker run myapp:opt
  4. Compare: docker images myapp
```

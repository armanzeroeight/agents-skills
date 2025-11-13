# Multi-Stage Build Patterns

## Contents
- Basic Multi-Stage Pattern
- Language-Specific Patterns
- Advanced Techniques
- Build Arguments and Targets
- Optimization Strategies

## Basic Multi-Stage Pattern

Multi-stage builds separate build-time dependencies from runtime, resulting in smaller final images.

**Basic structure**:
```dockerfile
# Stage 1: Build
FROM builder-image AS builder
WORKDIR /app
COPY source files
RUN build commands

# Stage 2: Runtime
FROM runtime-image
COPY --from=builder /app/artifact /app/
CMD ["run", "artifact"]
```

## Language-Specific Patterns

### Node.js Multi-Stage

**Development dependencies separated**:
```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Next.js optimized**:
```dockerfile
FROM node:18-alpine AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]
```

### Python Multi-Stage

**With build dependencies**:
```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /app/wheels /wheels
RUN pip install --no-cache /wheels/*
COPY . .
USER nobody
CMD ["python", "app.py"]
```

**With virtual environment**:
```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY . .
USER nobody
CMD ["python", "app.py"]
```

### Go Multi-Stage

**Static binary**:
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags="-w -s" -o app .

FROM scratch
COPY --from=builder /app/app /app
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
EXPOSE 8080
CMD ["/app"]
```

**With CGO dependencies**:
```dockerfile
FROM golang:1.21-alpine AS builder
RUN apk add --no-cache gcc musl-dev
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN go build -o app .

FROM alpine:3.18
RUN apk add --no-cache ca-certificates
COPY --from=builder /app/app /app
EXPOSE 8080
CMD ["/app"]
```

### Java Multi-Stage

**Maven build**:
```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
RUN addgroup -g 1001 -S appuser && \
    adduser -u 1001 -S appuser -G appuser
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Gradle build**:
```dockerfile
FROM gradle:8.4-jdk17 AS builder
WORKDIR /app
COPY build.gradle settings.gradle ./
COPY gradle ./gradle
RUN gradle dependencies --no-daemon
COPY src ./src
RUN gradle build --no-daemon

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
USER nobody
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

### Rust Multi-Stage

**Optimized build**:
```dockerfile
FROM rust:1.73-alpine AS builder
RUN apk add --no-cache musl-dev
WORKDIR /app
COPY Cargo.toml Cargo.lock ./
RUN mkdir src && echo "fn main() {}" > src/main.rs
RUN cargo build --release
RUN rm -rf src
COPY src ./src
RUN touch src/main.rs
RUN cargo build --release

FROM alpine:3.18
COPY --from=builder /app/target/release/app /app
EXPOSE 8080
CMD ["/app"]
```

## Advanced Techniques

### Multiple Build Stages

**Separate test and build**:
```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:18-alpine AS test
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm test

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

### Copying from Multiple Stages

```dockerfile
FROM node:18-alpine AS frontend-builder
WORKDIR /app
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ .
RUN npm run build

FROM golang:1.21-alpine AS backend-builder
WORKDIR /app
COPY backend/go.* ./
RUN go mod download
COPY backend/ .
RUN go build -o server

FROM alpine:3.18
COPY --from=frontend-builder /app/dist /static
COPY --from=backend-builder /app/server /server
EXPOSE 8080
CMD ["/server"]
```

### Named Stages for Reuse

```dockerfile
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./

FROM base AS development
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

FROM base AS production
RUN npm ci --only=production
COPY . .
CMD ["node", "server.js"]
```

## Build Arguments and Targets

### Build Arguments

```dockerfile
FROM node:18-alpine AS builder
ARG NODE_ENV=production
ARG API_URL
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

Build with arguments:
```bash
docker build --build-arg NODE_ENV=production --build-arg API_URL=https://api.example.com -t myapp .
```

### Build Targets

```dockerfile
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./

FROM base AS development
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

FROM base AS test
RUN npm ci
COPY . .
CMD ["npm", "test"]

FROM base AS production
RUN npm ci --only=production
COPY . .
RUN npm run build
CMD ["node", "dist/server.js"]
```

Build specific target:
```bash
docker build --target development -t myapp:dev .
docker build --target production -t myapp:prod .
```

## Optimization Strategies

### Dependency Caching

**Cache package manager layers**:
```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci

FROM node:18-alpine
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
CMD ["node", "server.js"]
```

### Parallel Builds

**Build independent stages in parallel**:
```dockerfile
FROM alpine:3.18 AS base
RUN apk add --no-cache ca-certificates

FROM golang:1.21-alpine AS api-builder
WORKDIR /app
COPY api/ .
RUN go build -o api

FROM node:18-alpine AS web-builder
WORKDIR /app
COPY web/ .
RUN npm ci && npm run build

FROM base
COPY --from=api-builder /app/api /api
COPY --from=web-builder /app/dist /static
CMD ["/api"]
```

### Minimal Final Image

**Copy only necessary artifacts**:
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && \
    npm prune --production

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
USER node
CMD ["node", "dist/server.js"]
```

## Best Practices

1. **Order stages by stability**: Most stable (base) to least stable (code)
2. **Name stages clearly**: Use descriptive names like `builder`, `deps`, `runner`
3. **Minimize final stage**: Only copy necessary artifacts
4. **Use specific base images**: Avoid `latest` tags
5. **Clean up in same layer**: Remove build artifacts in the same RUN command
6. **Leverage build cache**: Order commands by change frequency
7. **Use .dockerignore**: Exclude unnecessary files from context
8. **Security**: Run as non-root user in final stage

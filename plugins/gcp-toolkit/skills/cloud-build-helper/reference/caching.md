# Caching Strategies

## Docker Layer Caching
Pull previous image and use `--cache-from` flag to reuse layers.

## Dependency Caching
Cache node_modules, Maven dependencies, etc. in Cloud Storage or Container Registry.

## Build Artifact Caching
Store intermediate build artifacts for reuse across builds.

## Best Practices
- Always pull latest image for cache
- Push both tagged and latest images
- Use multi-stage builds
- Cache dependencies separately from application code

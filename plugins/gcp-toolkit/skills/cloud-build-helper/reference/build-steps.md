# Build Steps

## Step Configuration
Each step runs in a container with specified builder image.

## Common Builders
- `gcr.io/cloud-builders/docker` - Docker operations
- `gcr.io/cloud-builders/gcloud` - gcloud commands
- `gcr.io/cloud-builders/npm` - Node.js builds
- `gcr.io/cloud-builders/mvn` - Maven builds
- `gcr.io/cloud-builders/gradle` - Gradle builds

## Step Dependencies
Use `waitFor` to control execution order. Use `['-']` to run steps in parallel.

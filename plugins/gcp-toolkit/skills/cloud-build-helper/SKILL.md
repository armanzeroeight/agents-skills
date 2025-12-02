---
name: cloud-build-helper
description: Configures Google Cloud Build pipelines with caching, parallel builds, and optimization. Use when setting up Cloud Build, optimizing build performance, or configuring CI/CD pipelines.
---

# Cloud Build Helper

## Quick Start

Configure Cloud Build pipelines with proper caching and optimization for fast, efficient builds.

## Instructions

### Step 1: Create cloudbuild.yaml

```yaml
steps:
  # Build step
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA', '.']
  
  # Push to registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA']
  
  # Deploy
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'myapp'
      - '--image=gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA'
      - '--region=us-central1'

images:
  - 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA'

options:
  machineType: 'N1_HIGHCPU_8'
  logging: CLOUD_LOGGING_ONLY
```

### Step 2: Configure caching

```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        docker pull gcr.io/$PROJECT_ID/myapp:latest || exit 0
        docker build \
          --cache-from gcr.io/$PROJECT_ID/myapp:latest \
          -t gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA \
          -t gcr.io/$PROJECT_ID/myapp:latest \
          .
        docker push gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA
        docker push gcr.io/$PROJECT_ID/myapp:latest
```

### Step 3: Set up build triggers

```bash
# Create trigger from GitHub
gcloud builds triggers create github \
  --repo-name=my-repo \
  --repo-owner=my-org \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml

# Create trigger with substitutions
gcloud builds triggers create github \
  --repo-name=my-repo \
  --repo-owner=my-org \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml \
  --substitutions=_ENV=production,_REGION=us-central1
```

### Step 4: Optimize build performance

**Use parallel steps:**
```yaml
steps:
  # These run in parallel
  - name: 'gcr.io/cloud-builders/npm'
    id: 'install-frontend'
    args: ['install']
    dir: 'frontend'
    waitFor: ['-']
  
  - name: 'gcr.io/cloud-builders/npm'
    id: 'install-backend'
    args: ['install']
    dir: 'backend'
    waitFor: ['-']
  
  # This waits for both
  - name: 'gcr.io/cloud-builders/npm'
    args: ['run', 'build']
    waitFor: ['install-frontend', 'install-backend']
```

## Best Practices

1. Use caching to speed up builds
2. Implement parallel build steps
3. Use appropriate machine types
4. Store artifacts in Cloud Storage
5. Use substitution variables
6. Implement proper error handling
7. Monitor build performance
8. Use Cloud Build's built-in builders when possible

## Advanced

For detailed information, see:
- [Build Steps](reference/build-steps.md) - Build step configuration and patterns
- [Caching](reference/caching.md) - Caching strategies for faster builds
- [Triggers](reference/triggers.md) - Build trigger configuration

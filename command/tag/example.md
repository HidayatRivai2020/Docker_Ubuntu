# Examples

## Basic Examples
```bash
# Tag an image with a new name
docker tag nginx my-nginx

# Tag with specific version
docker tag nginx my-nginx:v1.0

# Tag using image ID
docker tag 4bb46517cac3 my-app:latest

# Create multiple tags for same image
docker tag myapp:latest myapp:v1.0.0
docker tag myapp:latest myapp:stable
```

## Version Tagging Examples
```bash
# Semantic versioning
docker tag myapp:latest myapp:v1.0.0
docker tag myapp:latest myapp:v1.0
docker tag myapp:latest myapp:v1

# Tag with build number
docker tag myapp:latest myapp:build-123

# Tag with commit hash
docker tag myapp:latest myapp:abc1234

# Tag with date
docker tag myapp:latest myapp:2025-11-24
```

## Registry Tagging Examples
```bash
# Tag for Docker Hub (username/repository:tag)
docker tag myapp username/myapp:latest
docker tag myapp username/myapp:v1.0.0

# Tag for private registry
docker tag myapp localhost:5000/myapp:latest
docker tag myapp registry.example.com/myapp:v1.0

# Tag with full registry path
docker tag myapp:latest docker.io/username/myapp:latest
```

## Environment Tagging Examples
```bash
# Tag for different environments
docker tag myapp:latest myapp:dev
docker tag myapp:latest myapp:staging
docker tag myapp:latest myapp:prod

# Tag with environment and version
docker tag myapp:latest myapp:prod-v1.0.0
docker tag myapp:latest myapp:dev-latest

# Tag for testing
docker tag myapp:latest myapp:test-branch-feature-x
```

## Advanced Examples
```bash
# Tag before pushing to registry
docker tag myapp:latest myregistry.azurecr.io/myapp:v1.0.0
docker push myregistry.azurecr.io/myapp:v1.0.0

# Tag multiple registries
docker tag myapp:latest registry1.com/myapp:v1.0
docker tag myapp:latest registry2.com/myapp:v1.0

# Tag with namespace
docker tag myapp:latest company/team/myapp:latest

# Create backup tag before updating
docker tag myapp:latest myapp:backup-$(date +%Y%m%d)
docker build -t myapp:latest .
```

## Common Use Cases

### Version Release Workflow
```bash
# Build image
docker build -t myapp:latest .

# Tag with version numbers
docker tag myapp:latest myapp:v2.1.0
docker tag myapp:latest myapp:v2.1
docker tag myapp:latest myapp:v2

# Tag for Docker Hub
docker tag myapp:latest username/myapp:v2.1.0
docker tag myapp:latest username/myapp:latest

# Push to registry
docker push username/myapp:v2.1.0
docker push username/myapp:latest
```

### Multi-Registry Deployment
```bash
# Tag for multiple registries
docker tag myapp:latest docker.io/username/myapp:v1.0
docker tag myapp:latest gcr.io/project/myapp:v1.0
docker tag myapp:latest registry.example.com/myapp:v1.0

# Push to all registries
docker push docker.io/username/myapp:v1.0
docker push gcr.io/project/myapp:v1.0
docker push registry.example.com/myapp:v1.0
```

### Development Workflow
```bash
# Build development image
docker build -t myapp:dev .

# Tag for testing
docker tag myapp:dev myapp:test-feature-auth

# After testing, promote to staging
docker tag myapp:dev myapp:staging

# After staging validation, promote to production
docker tag myapp:staging myapp:prod
docker tag myapp:staging myapp:v1.2.3
```

### CI/CD Integration
```bash
# Tag with CI build information
docker tag myapp:latest myapp:build-${BUILD_NUMBER}
docker tag myapp:latest myapp:commit-${GIT_COMMIT}
docker tag myapp:latest myapp:branch-${GIT_BRANCH}

# Tag with version from package.json or similar
VERSION=$(cat package.json | jq -r .version)
docker tag myapp:latest myapp:v${VERSION}

# Tag for different stages
docker tag myapp:latest myapp:${CI_ENVIRONMENT_NAME}
```

## CI/CD Integration

### GitHub Actions
```yaml
name: Build and Tag Docker Image

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build image
        run: docker build -t myapp:latest .
      
      - name: Tag with version
        run: |
          VERSION=${GITHUB_REF#refs/tags/}
          docker tag myapp:latest myapp:${VERSION}
          docker tag myapp:latest username/myapp:${VERSION}
          docker tag myapp:latest username/myapp:latest
      
      - name: Tag with commit SHA
        run: docker tag myapp:latest myapp:${GITHUB_SHA::7}
      
      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
      
      - name: Push images
        run: |
          docker push username/myapp:${GITHUB_REF#refs/tags/}
          docker push username/myapp:latest
```

### GitLab CI
```yaml
build-and-tag:
  stage: build
  script:
    # Build image
    - docker build -t $CI_PROJECT_NAME:latest .
    
    # Tag with version
    - docker tag $CI_PROJECT_NAME:latest $CI_PROJECT_NAME:$CI_COMMIT_TAG
    
    # Tag with commit SHA
    - docker tag $CI_PROJECT_NAME:latest $CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA
    
    # Tag for registry
    - docker tag $CI_PROJECT_NAME:latest $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
    - docker tag $CI_PROJECT_NAME:latest $CI_REGISTRY_IMAGE:latest
    
    # Push to registry
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - tags
```

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'myapp'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USERNAME = 'username'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:latest .'
            }
        }
        
        stage('Tag') {
            steps {
                script {
                    // Tag with build number
                    sh "docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:build-${BUILD_NUMBER}"
                    
                    // Tag with version from file
                    def version = readFile('VERSION').trim()
                    sh "docker tag ${DOCKER_IMAGE}:latest ${DOCKER_IMAGE}:v${version}"
                    
                    // Tag for registry
                    sh "docker tag ${DOCKER_IMAGE}:latest ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${DOCKER_IMAGE}:v${version}"
                    sh "docker tag ${DOCKER_IMAGE}:latest ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${DOCKER_IMAGE}:v${version}"
                    sh "docker push ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${DOCKER_IMAGE}:latest"
                }
            }
        }
    }
}
```

## Scripting Examples

### Batch Tagging Script
```bash
#!/bin/bash
# tag-release.sh - Tag image with multiple versions

IMAGE=$1
VERSION=$2

if [ -z "$IMAGE" ] || [ -z "$VERSION" ]; then
    echo "Usage: $0 <image> <version>"
    echo "Example: $0 myapp 1.2.3"
    exit 1
fi

echo "Tagging $IMAGE with version $VERSION"

# Extract version components
MAJOR=$(echo $VERSION | cut -d. -f1)
MINOR=$(echo $VERSION | cut -d. -f2)
PATCH=$(echo $VERSION | cut -d. -f3)

# Tag with full version
docker tag $IMAGE:latest $IMAGE:$VERSION
docker tag $IMAGE:latest $IMAGE:$MAJOR.$MINOR
docker tag $IMAGE:latest $IMAGE:$MAJOR

# Tag for Docker Hub
DOCKER_USERNAME="username"
docker tag $IMAGE:latest $DOCKER_USERNAME/$IMAGE:$VERSION
docker tag $IMAGE:latest $DOCKER_USERNAME/$IMAGE:latest

echo "Created tags:"
docker images $IMAGE
docker images $DOCKER_USERNAME/$IMAGE
```

### Multi-Registry Tagging Script
```bash
#!/bin/bash
# tag-multi-registry.sh - Tag and push to multiple registries

IMAGE=$1
VERSION=$2

if [ -z "$IMAGE" ] || [ -z "$VERSION" ]; then
    echo "Usage: $0 <image> <version>"
    exit 1
fi

# Define registries
REGISTRIES=(
    "docker.io/username"
    "gcr.io/project-id"
    "registry.example.com"
)

echo "Tagging $IMAGE:$VERSION for multiple registries..."

for REGISTRY in "${REGISTRIES[@]}"; do
    echo "Tagging for $REGISTRY..."
    docker tag $IMAGE:latest $REGISTRY/$IMAGE:$VERSION
    docker tag $IMAGE:latest $REGISTRY/$IMAGE:latest
    
    echo "Pushing to $REGISTRY..."
    docker push $REGISTRY/$IMAGE:$VERSION
    docker push $REGISTRY/$IMAGE:latest
done

echo "Tagging and pushing complete!"
```

### Environment Promotion Script
```bash
#!/bin/bash
# promote.sh - Promote image between environments

IMAGE=$1
FROM_ENV=$2
TO_ENV=$3

if [ -z "$IMAGE" ] || [ -z "$FROM_ENV" ] || [ -z "$TO_ENV" ]; then
    echo "Usage: $0 <image> <from_env> <to_env>"
    echo "Example: $0 myapp dev staging"
    exit 1
fi

echo "Promoting $IMAGE from $FROM_ENV to $TO_ENV..."

# Verify source image exists
if ! docker images | grep -q "$IMAGE:$FROM_ENV"; then
    echo "Error: $IMAGE:$FROM_ENV not found"
    exit 1
fi

# Create backup tag
BACKUP_TAG="$TO_ENV-backup-$(date +%Y%m%d-%H%M%S)"
if docker images | grep -q "$IMAGE:$TO_ENV"; then
    echo "Creating backup: $IMAGE:$BACKUP_TAG"
    docker tag $IMAGE:$TO_ENV $IMAGE:$BACKUP_TAG
fi

# Promote
docker tag $IMAGE:$FROM_ENV $IMAGE:$TO_ENV

echo "Successfully promoted $IMAGE from $FROM_ENV to $TO_ENV"
echo "Backup created: $IMAGE:$BACKUP_TAG"
```

### Cleanup Old Tags Script
```bash
#!/bin/bash
# cleanup-tags.sh - Remove old version tags

IMAGE=$1
KEEP_VERSIONS=${2:-5}

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image> [keep_versions]"
    echo "Example: $0 myapp 5"
    exit 1
fi

echo "Cleaning up old tags for $IMAGE, keeping last $KEEP_VERSIONS versions..."

# Get all version tags, sort, and remove old ones
docker images $IMAGE --format "{{.Tag}}" | \
    grep -E '^v[0-9]+\.[0-9]+\.[0-9]+$' | \
    sort -V -r | \
    tail -n +$((KEEP_VERSIONS + 1)) | \
    while read TAG; do
        echo "Removing $IMAGE:$TAG"
        docker rmi $IMAGE:$TAG
    done

echo "Cleanup complete!"
```

## Advanced Use Cases

### Automated Version Tagging
```bash
#!/bin/bash
# auto-version-tag.sh - Automatically determine and tag version

IMAGE=$1

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image>"
    exit 1
fi

# Try to get version from various sources
if [ -f package.json ]; then
    VERSION=$(jq -r .version package.json)
elif [ -f setup.py ]; then
    VERSION=$(python setup.py --version)
elif [ -f pom.xml ]; then
    VERSION=$(mvn help:evaluate -Dexpression=project.version -q -DforceStdout)
elif [ -f VERSION ]; then
    VERSION=$(cat VERSION)
else
    echo "Error: Cannot determine version"
    exit 1
fi

echo "Detected version: $VERSION"

# Tag with version
docker tag $IMAGE:latest $IMAGE:v$VERSION
docker tag $IMAGE:latest $IMAGE:latest

echo "Tagged $IMAGE with v$VERSION"
```

### Registry Migration Script
```bash
#!/bin/bash
# migrate-registry.sh - Migrate images from one registry to another

SOURCE_REGISTRY=$1
TARGET_REGISTRY=$2
IMAGE=$3

if [ -z "$SOURCE_REGISTRY" ] || [ -z "$TARGET_REGISTRY" ] || [ -z "$IMAGE" ]; then
    echo "Usage: $0 <source_registry> <target_registry> <image>"
    echo "Example: $0 old-registry.com new-registry.com myapp"
    exit 1
fi

echo "Migrating $IMAGE from $SOURCE_REGISTRY to $TARGET_REGISTRY..."

# Pull from source
docker pull $SOURCE_REGISTRY/$IMAGE

# Get all tags
TAGS=$(docker images $SOURCE_REGISTRY/$IMAGE --format "{{.Tag}}")

# Tag and push each version
for TAG in $TAGS; do
    echo "Processing $TAG..."
    docker tag $SOURCE_REGISTRY/$IMAGE:$TAG $TARGET_REGISTRY/$IMAGE:$TAG
    docker push $TARGET_REGISTRY/$IMAGE:$TAG
done

echo "Migration complete!"
```

### Tag Audit Script
```bash
#!/bin/bash
# audit-tags.sh - Audit and report on image tags

IMAGE=$1

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image>"
    exit 1
fi

echo "=== Tag Audit for $IMAGE ==="
echo

echo "All tags:"
docker images $IMAGE --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedAt}}"
echo

echo "Tag count: $(docker images $IMAGE | tail -n +2 | wc -l)"
echo

echo "Total size (all tags): $(docker images $IMAGE --format "{{.Size}}" | head -1)"
echo

echo "Tags by pattern:"
echo "  Version tags (vX.X.X): $(docker images $IMAGE --format "{{.Tag}}" | grep -c '^v[0-9]')"
echo "  Environment tags: $(docker images $IMAGE --format "{{.Tag}}" | grep -cE 'dev|staging|prod')"
echo "  Latest tags: $(docker images $IMAGE --format "{{.Tag}}" | grep -c 'latest')"
echo

echo "Oldest tag:"
docker images $IMAGE --format "table {{.Tag}}\t{{.CreatedAt}}" | tail -1
echo

echo "Newest tag:"
docker images $IMAGE --format "table {{.Tag}}\t{{.CreatedAt}}" | head -2 | tail -1
```

## Security Considerations

### Registry Authentication
```bash
# Login to Docker Hub before tagging
docker login

# Tag and push authenticated
docker tag myapp:latest username/myapp:v1.0
docker push username/myapp:v1.0

# Login to private registry
docker login registry.example.com

# Tag for private registry
docker tag myapp:latest registry.example.com/myapp:v1.0
```

### Secure Tag Naming
```bash
# Avoid sensitive information in tags
# Bad: myapp:prod-password-xyz
# Good: myapp:prod-v1.0.0

# Use consistent, predictable naming
docker tag myapp:latest myapp:v1.0.0
docker tag myapp:latest myapp:prod-v1.0.0

# Include build metadata if needed
docker tag myapp:latest myapp:v1.0.0-build123
```

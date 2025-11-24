# Examples

## Basic Examples
```bash
# Push image to Docker Hub
docker push username/myapp:latest

# Push with specific tag
docker push username/myapp:v1.0.0

# Push to private registry
docker push registry.example.com/myapp:latest

# Push all tags of a repository
docker push -a username/myapp
```

## Tag and Push Workflow
```bash
# Build image
docker build -t myapp:latest .

# Tag for Docker Hub
docker tag myapp:latest username/myapp:latest

# Push to Docker Hub
docker push username/myapp:latest

# Tag with version
docker tag myapp:latest username/myapp:v1.0.0
docker push username/myapp:v1.0.0
```

## Multi-Tag Push Examples
```bash
# Create multiple tags
docker tag myapp:latest username/myapp:latest
docker tag myapp:latest username/myapp:v1.0.0
docker tag myapp:latest username/myapp:stable

# Push all tags
docker push username/myapp:latest
docker push username/myapp:v1.0.0
docker push username/myapp:stable

# Or push all tags at once
docker push -a username/myapp
```

## Registry-Specific Examples
```bash
# Push to Docker Hub
docker push docker.io/username/myapp:latest

# Push to AWS ECR
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Push to Google Container Registry
docker push gcr.io/project-id/myapp:latest

# Push to Azure Container Registry
docker push myregistry.azurecr.io/myapp:latest

# Push to GitHub Container Registry
docker push ghcr.io/username/myapp:latest
```

## Advanced Examples
```bash
# Push with quiet output
docker push -q username/myapp:latest

# Push and monitor progress
docker push username/myapp:latest | tee push.log

# Push with specific compression
docker build --compress -t myapp .
docker push username/myapp:latest

# Push multi-platform image
docker buildx build --platform linux/amd64,linux/arm64 -t username/myapp:latest --push .
```

## Common Use Cases

### Version Release Workflow
```bash
# Build production image
docker build -t myapp:latest .

# Tag with version
VERSION="1.2.3"
docker tag myapp:latest username/myapp:$VERSION
docker tag myapp:latest username/myapp:latest

# Push both tags
docker push username/myapp:$VERSION
docker push username/myapp:latest

# Verify push
docker pull username/myapp:$VERSION
```

### Multi-Registry Deployment
```bash
# Build once
docker build -t myapp:v1.0 .

# Tag for multiple registries
docker tag myapp:v1.0 docker.io/username/myapp:v1.0
docker tag myapp:v1.0 gcr.io/project/myapp:v1.0
docker tag myapp:v1.0 registry.company.com/myapp:v1.0

# Push to all registries
docker push docker.io/username/myapp:v1.0
docker push gcr.io/project/myapp:v1.0
docker push registry.company.com/myapp:v1.0
```

### CI/CD Pipeline Push
```bash
# Build with CI metadata
docker build -t myapp:build-${BUILD_NUMBER} .

# Tag for registry
docker tag myapp:build-${BUILD_NUMBER} username/myapp:${GIT_COMMIT}
docker tag myapp:build-${BUILD_NUMBER} username/myapp:latest

# Push to registry
docker push username/myapp:${GIT_COMMIT}
docker push username/myapp:latest
```

### Development Workflow
```bash
# Build development image
docker build -t myapp:dev .

# Tag for dev registry
docker tag myapp:dev registry.dev.company.com/myapp:latest

# Push to development registry
docker push registry.dev.company.com/myapp:latest

# After testing, promote to staging
docker tag myapp:dev registry.staging.company.com/myapp:latest
docker push registry.staging.company.com/myapp:latest
```

## CI/CD Integration

### GitHub Actions
```yaml
name: Build and Push

on:
  push:
    tags:
      - 'v*'

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build image
        run: docker build -t myapp:latest .
      
      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
      
      - name: Tag images
        run: |
          VERSION=${GITHUB_REF#refs/tags/}
          docker tag myapp:latest ${{ secrets.DOCKER_USERNAME }}/myapp:$VERSION
          docker tag myapp:latest ${{ secrets.DOCKER_USERNAME }}/myapp:latest
      
      - name: Push images
        run: |
          VERSION=${GITHUB_REF#refs/tags/}
          docker push ${{ secrets.DOCKER_USERNAME }}/myapp:$VERSION
          docker push ${{ secrets.DOCKER_USERNAME }}/myapp:latest
```

### GitLab CI
```yaml
docker-push:
  stage: deploy
  script:
    # Build image
    - docker build -t $CI_PROJECT_NAME:$CI_COMMIT_SHA .
    
    # Login to registry
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
    
    # Tag for registry
    - docker tag $CI_PROJECT_NAME:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG
    - docker tag $CI_PROJECT_NAME:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    
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
                sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
            }
        }
        
        stage('Tag') {
            steps {
                sh """
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${DOCKER_IMAGE}:latest
                """
            }
        }
        
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh """
                        docker push ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_REGISTRY}/${DOCKER_USERNAME}/${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
    }
}
```

## Scripting Examples

### Automated Push Script
```bash
#!/bin/bash
# push-image.sh - Build, tag, and push Docker image

IMAGE_NAME=$1
VERSION=$2
REGISTRY=${3:-docker.io}
USERNAME=${4:-$USER}

if [ -z "$IMAGE_NAME" ] || [ -z "$VERSION" ]; then
    echo "Usage: $0 <image_name> <version> [registry] [username]"
    exit 1
fi

echo "Building image..."
docker build -t $IMAGE_NAME:latest .

echo "Tagging image..."
docker tag $IMAGE_NAME:latest $REGISTRY/$USERNAME/$IMAGE_NAME:$VERSION
docker tag $IMAGE_NAME:latest $REGISTRY/$USERNAME/$IMAGE_NAME:latest

echo "Pushing to $REGISTRY..."
docker push $REGISTRY/$USERNAME/$IMAGE_NAME:$VERSION
docker push $REGISTRY/$USERNAME/$IMAGE_NAME:latest

echo "✓ Push completed successfully"
```

### Multi-Registry Push Script
```bash
#!/bin/bash
# push-multi-registry.sh - Push to multiple registries

IMAGE=$1
TAG=$2

if [ -z "$IMAGE" ] || [ -z "$TAG" ]; then
    echo "Usage: $0 <image> <tag>"
    exit 1
fi

REGISTRIES=(
    "docker.io/$DOCKER_HUB_USERNAME"
    "gcr.io/$GCP_PROJECT_ID"
    "registry.company.com"
)

echo "Pushing $IMAGE:$TAG to multiple registries..."

for REGISTRY in "${REGISTRIES[@]}"; do
    FULL_IMAGE="$REGISTRY/$IMAGE:$TAG"
    echo "Tagging for $REGISTRY..."
    docker tag $IMAGE:$TAG $FULL_IMAGE
    
    echo "Pushing $FULL_IMAGE..."
    docker push $FULL_IMAGE
    
    if [ $? -eq 0 ]; then
        echo "✓ Successfully pushed to $REGISTRY"
    else
        echo "✗ Failed to push to $REGISTRY"
    fi
done
```

### Retry Push Script
```bash
#!/bin/bash
# retry-push.sh - Push with retry logic

IMAGE=$1
MAX_RETRIES=3
RETRY_DELAY=5

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image:tag>"
    exit 1
fi

for i in $(seq 1 $MAX_RETRIES); do
    echo "Push attempt $i of $MAX_RETRIES..."
    
    docker push $IMAGE
    
    if [ $? -eq 0 ]; then
        echo "✓ Push successful"
        exit 0
    fi
    
    if [ $i -lt $MAX_RETRIES ]; then
        echo "Push failed, retrying in $RETRY_DELAY seconds..."
        sleep $RETRY_DELAY
    fi
done

echo "✗ Push failed after $MAX_RETRIES attempts"
exit 1
```

### Conditional Push Script
```bash
#!/bin/bash
# conditional-push.sh - Push only if image changed

IMAGE=$1
TAG=$2

if [ -z "$IMAGE" ] || [ -z "$TAG" ]; then
    echo "Usage: $0 <image> <tag>"
    exit 1
fi

# Get local image digest
LOCAL_DIGEST=$(docker inspect --format='{{.Id}}' $IMAGE:$TAG)

# Get remote image digest
REMOTE_DIGEST=$(docker pull -q $IMAGE:$TAG 2>/dev/null && docker inspect --format='{{.Id}}' $IMAGE:$TAG)

if [ "$LOCAL_DIGEST" == "$REMOTE_DIGEST" ]; then
    echo "Image unchanged, skipping push"
    exit 0
fi

echo "Image changed, pushing..."
docker push $IMAGE:$TAG

echo "✓ Push completed"
```

## Security Considerations

### Scan Before Push
```bash
# Scan for vulnerabilities
docker scan myapp:latest

# Only push if scan passes
if docker scan myapp:latest | grep -q "0 vulnerabilities"; then
    docker push username/myapp:latest
else
    echo "Vulnerabilities found, not pushing"
    exit 1
fi
```

### Sign Images
```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Push signed image
docker push username/myapp:latest

# Verify signature
docker trust inspect username/myapp:latest
```

### Private Registry Security
```bash
# Login to private registry with token
echo $REGISTRY_TOKEN | docker login registry.company.com -u token --password-stdin

# Push to private registry
docker push registry.company.com/myapp:latest

# Logout after push
docker logout registry.company.com
```

## Troubleshooting Examples

### Verify Push Success
```bash
# Push image
docker push username/myapp:latest

# Verify by pulling
docker pull username/myapp:latest

# Check image digest
docker inspect username/myapp:latest --format='{{.RepoDigests}}'
```

### Monitor Push Progress
```bash
# Push with progress output
docker push username/myapp:latest 2>&1 | tee push.log

# Extract layer information
grep "Pushed" push.log

# Check push time
time docker push username/myapp:latest
```

### Debug Push Issues
```bash
# Enable debug logging
docker --debug push username/myapp:latest

# Check image exists
docker images username/myapp:latest

# Verify authentication
docker login

# Test registry connectivity
curl -I https://registry.hub.docker.com/v2/
```

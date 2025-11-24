# Examples

## Basic Examples

### docker manifest create
```bash
# Enable experimental features first
export DOCKER_CLI_EXPERIMENTAL=enabled

# Create manifest list from platform images
docker manifest create myapp:latest \
    myapp:amd64 \
    myapp:arm64

# Create with amend flag (update existing)
docker manifest create --amend myapp:latest \
    myapp:amd64 \
    myapp:arm64

# Create with multiple architectures
docker manifest create myapp:1.0 \
    myapp:1.0-amd64 \
    myapp:1.0-arm64 \
    myapp:1.0-armv7

# Create for insecure registry
docker manifest create --insecure registry.local:5000/myapp:latest \
    registry.local:5000/myapp:amd64 \
    registry.local:5000/myapp:arm64
```

### docker manifest annotate
```bash
# Annotate AMD64 image
docker manifest annotate myapp:latest myapp:amd64 \
    --os linux \
    --arch amd64

# Annotate ARM64 image
docker manifest annotate myapp:latest myapp:arm64 \
    --os linux \
    --arch arm64

# Annotate ARMv7 with variant
docker manifest annotate myapp:latest myapp:armv7 \
    --os linux \
    --arch arm \
    --variant v7

# Annotate Windows image
docker manifest annotate myapp:latest myapp:windows-amd64 \
    --os windows \
    --arch amd64 \
    --os-version 10.0.17763.1518
```

### docker manifest inspect
```bash
# Inspect manifest list
docker manifest inspect nginx:latest

# Inspect with verbose output
docker manifest inspect -v nginx:latest

# Inspect specific platform image
docker manifest inspect myapp:amd64

# Inspect and parse with jq
docker manifest inspect nginx:latest | jq '.manifests[].platform'

# Check if image is multi-platform
docker manifest inspect myapp:latest | jq '.mediaType'
```

### docker manifest push
```bash
# Push manifest list to registry
docker manifest push myapp:latest

# Push and remove local manifest
docker manifest push --purge myapp:latest

# Push to insecure registry
docker manifest push --insecure registry.local:5000/myapp:latest

# Push with all platforms
docker manifest push username/myapp:latest
```

### docker manifest rm
```bash
# Remove local manifest list
docker manifest rm myapp:latest

# Remove multiple manifests
docker manifest rm myapp:latest myapp:1.0 myapp:2.0

# Remove after inspection
docker manifest inspect myapp:test
docker manifest rm myapp:test
```

## Complete Multi-Platform Workflow

### Two-Platform Setup (AMD64 + ARM64)
```bash
# Step 1: Enable experimental features
export DOCKER_CLI_EXPERIMENTAL=enabled

# Step 2: Build platform-specific images
docker build --platform linux/amd64 -t myapp:amd64 .
docker build --platform linux/arm64 -t myapp:arm64 .

# Step 3: Push platform images
docker push myapp:amd64
docker push myapp:arm64

# Step 4: Create manifest list
docker manifest create myapp:latest \
    myapp:amd64 \
    myapp:arm64

# Step 5: Annotate platforms
docker manifest annotate myapp:latest myapp:amd64 \
    --os linux --arch amd64

docker manifest annotate myapp:latest myapp:arm64 \
    --os linux --arch arm64

# Step 6: Inspect before pushing
docker manifest inspect myapp:latest

# Step 7: Push manifest list
docker manifest push myapp:latest

# Step 8: Verify
docker manifest inspect myapp:latest
```

### Three-Platform Setup (AMD64 + ARM64 + ARMv7)
```bash
# Build all platforms
docker build --platform linux/amd64 -t myapp:1.0-amd64 .
docker build --platform linux/arm64 -t myapp:1.0-arm64 .
docker build --platform linux/arm/v7 -t myapp:1.0-armv7 .

# Push platform images
docker push myapp:1.0-amd64
docker push myapp:1.0-arm64
docker push myapp:1.0-armv7

# Create manifest list
docker manifest create myapp:1.0 \
    myapp:1.0-amd64 \
    myapp:1.0-arm64 \
    myapp:1.0-armv7

# Annotate each platform
docker manifest annotate myapp:1.0 myapp:1.0-amd64 \
    --os linux --arch amd64

docker manifest annotate myapp:1.0 myapp:1.0-arm64 \
    --os linux --arch arm64 --variant v8

docker manifest annotate myapp:1.0 myapp:1.0-armv7 \
    --os linux --arch arm --variant v7

# Push manifest
docker manifest push myapp:1.0
```

## Platform-Specific Examples

### ARM Variants
```bash
# Build ARM variants
docker build --platform linux/arm/v6 -t myapp:armv6 .
docker build --platform linux/arm/v7 -t myapp:armv7 .
docker build --platform linux/arm64 -t myapp:arm64 .

# Push images
docker push myapp:armv6
docker push myapp:armv7
docker push myapp:arm64

# Create manifest
docker manifest create myapp:arm \
    myapp:armv6 \
    myapp:armv7 \
    myapp:arm64

# Annotate with variants
docker manifest annotate myapp:arm myapp:armv6 \
    --os linux --arch arm --variant v6

docker manifest annotate myapp:arm myapp:armv7 \
    --os linux --arch arm --variant v7

docker manifest annotate myapp:arm myapp:arm64 \
    --os linux --arch arm64 --variant v8

# Push
docker manifest push myapp:arm
```

### Windows Containers
```bash
# Build Windows images
docker build -t myapp:windows-ltsc2019 -f Dockerfile.windows .
docker build -t myapp:windows-ltsc2022 -f Dockerfile.windows .

# Push images
docker push myapp:windows-ltsc2019
docker push myapp:windows-ltsc2022

# Create manifest
docker manifest create myapp:windows \
    myapp:windows-ltsc2019 \
    myapp:windows-ltsc2022

# Annotate with OS versions
docker manifest annotate myapp:windows myapp:windows-ltsc2019 \
    --os windows --arch amd64 --os-version 10.0.17763

docker manifest annotate myapp:windows myapp:windows-ltsc2022 \
    --os windows --arch amd64 --os-version 10.0.20348

# Push
docker manifest push myapp:windows
```

### Mixed OS Platforms
```bash
# Build Linux and Windows images
docker build --platform linux/amd64 -t myapp:linux-amd64 .
docker build -t myapp:windows-amd64 -f Dockerfile.windows .

# Push images
docker push myapp:linux-amd64
docker push myapp:windows-amd64

# Create cross-OS manifest
docker manifest create myapp:cross-platform \
    myapp:linux-amd64 \
    myapp:windows-amd64

# Annotate
docker manifest annotate myapp:cross-platform myapp:linux-amd64 \
    --os linux --arch amd64

docker manifest annotate myapp:cross-platform myapp:windows-amd64 \
    --os windows --arch amd64

# Push
docker manifest push myapp:cross-platform
```

## Inspection and Verification

### Check Manifest Structure
```bash
# View manifest list details
docker manifest inspect myapp:latest

# Check media type
docker manifest inspect myapp:latest | jq '.mediaType'

# List all platforms
docker manifest inspect myapp:latest | jq -r '.manifests[].platform | "\(.os)/\(.architecture)/\(.variant // "none")"'

# Get platform digests
docker manifest inspect myapp:latest | jq -r '.manifests[] | "\(.platform.architecture): \(.digest)"'

# Check manifest size
docker manifest inspect myapp:latest | jq '.manifests[] | {platform: .platform, size: .size}'
```

### Verify Platform Images
```bash
# Check each platform exists
docker manifest inspect myapp:latest | jq '.manifests[].digest' | while read digest; do
    echo "Checking $digest"
    docker manifest inspect myapp@$digest
done

# Verify image can be pulled on each platform
docker pull --platform linux/amd64 myapp:latest
docker pull --platform linux/arm64 myapp:latest

# Inspect pulled image
docker inspect myapp:latest --format '{{.Os}}/{{.Architecture}}/{{.Variant}}'
```

### Compare Manifests
```bash
# Compare two manifest lists
docker manifest inspect myapp:v1 > v1-manifest.json
docker manifest inspect myapp:v2 > v2-manifest.json
diff v1-manifest.json v2-manifest.json

# Check platform differences
docker manifest inspect myapp:v1 | jq '.manifests[].platform' > v1-platforms.json
docker manifest inspect myapp:v2 | jq '.manifests[].platform' > v2-platforms.json
diff v1-platforms.json v2-platforms.json
```

## Update and Maintenance

### Update Existing Manifest
```bash
# Update manifest with new platform image
docker build --platform linux/ppc64le -t myapp:ppc64le .
docker push myapp:ppc64le

# Amend existing manifest
docker manifest create --amend myapp:latest \
    myapp:amd64 \
    myapp:arm64 \
    myapp:ppc64le

# Annotate new platform
docker manifest annotate myapp:latest myapp:ppc64le \
    --os linux --arch ppc64le

# Push updated manifest
docker manifest push myapp:latest
```

### Replace Platform Image
```bash
# Build new version of platform image
docker build --platform linux/amd64 -t myapp:amd64-v2 .
docker push myapp:amd64-v2

# Remove old manifest
docker manifest rm myapp:latest

# Create new manifest with updated image
docker manifest create myapp:latest \
    myapp:amd64-v2 \
    myapp:arm64

# Annotate and push
docker manifest annotate myapp:latest myapp:amd64-v2 \
    --os linux --arch amd64
docker manifest push myapp:latest
```

### Version Tagging Strategy
```bash
# Create versioned manifests
docker manifest create myapp:1.0.0 myapp:1.0.0-amd64 myapp:1.0.0-arm64
docker manifest create myapp:1.0 myapp:1.0.0-amd64 myapp:1.0.0-arm64
docker manifest create myapp:1 myapp:1.0.0-amd64 myapp:1.0.0-arm64
docker manifest create myapp:latest myapp:1.0.0-amd64 myapp:1.0.0-arm64

# Push all tags
docker manifest push myapp:1.0.0
docker manifest push myapp:1.0
docker manifest push myapp:1
docker manifest push myapp:latest
```

## CI/CD Integration

### GitHub Actions
```yaml
name: Build Multi-Platform

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Enable experimental
        run: |
          mkdir -p ~/.docker
          echo '{"experimental": "enabled"}' > ~/.docker/config.json
          export DOCKER_CLI_EXPERIMENTAL=enabled
      
      - name: Build AMD64
        run: docker build --platform linux/amd64 -t myapp:amd64 .
      
      - name: Build ARM64
        run: docker build --platform linux/arm64 -t myapp:arm64 .
      
      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
      
      - name: Push images
        run: |
          docker push myapp:amd64
          docker push myapp:arm64
      
      - name: Create manifest
        run: |
          docker manifest create myapp:latest myapp:amd64 myapp:arm64
          docker manifest annotate myapp:latest myapp:amd64 --os linux --arch amd64
          docker manifest annotate myapp:latest myapp:arm64 --os linux --arch arm64
      
      - name: Push manifest
        run: docker manifest push myapp:latest
```

### GitLab CI
```yaml
build-multiplatform:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - export DOCKER_CLI_EXPERIMENTAL=enabled
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin $CI_REGISTRY
  script:
    # Build platforms
    - docker build --platform linux/amd64 -t $CI_REGISTRY_IMAGE:amd64 .
    - docker build --platform linux/arm64 -t $CI_REGISTRY_IMAGE:arm64 .
    
    # Push platform images
    - docker push $CI_REGISTRY_IMAGE:amd64
    - docker push $CI_REGISTRY_IMAGE:arm64
    
    # Create and push manifest
    - docker manifest create $CI_REGISTRY_IMAGE:latest $CI_REGISTRY_IMAGE:amd64 $CI_REGISTRY_IMAGE:arm64
    - docker manifest annotate $CI_REGISTRY_IMAGE:latest $CI_REGISTRY_IMAGE:amd64 --os linux --arch amd64
    - docker manifest annotate $CI_REGISTRY_IMAGE:latest $CI_REGISTRY_IMAGE:arm64 --os linux --arch arm64
    - docker manifest push $CI_REGISTRY_IMAGE:latest
```

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    environment {
        DOCKER_CLI_EXPERIMENTAL = 'enabled'
    }
    stages {
        stage('Build Platforms') {
            parallel {
                stage('AMD64') {
                    steps {
                        sh 'docker build --platform linux/amd64 -t myapp:amd64 .'
                        sh 'docker push myapp:amd64'
                    }
                }
                stage('ARM64') {
                    steps {
                        sh 'docker build --platform linux/arm64 -t myapp:arm64 .'
                        sh 'docker push myapp:arm64'
                    }
                }
            }
        }
        stage('Create Manifest') {
            steps {
                sh '''
                    docker manifest create myapp:latest myapp:amd64 myapp:arm64
                    docker manifest annotate myapp:latest myapp:amd64 --os linux --arch amd64
                    docker manifest annotate myapp:latest myapp:arm64 --os linux --arch arm64
                '''
            }
        }
        stage('Push Manifest') {
            steps {
                sh 'docker manifest push myapp:latest'
            }
        }
    }
}
```

## Troubleshooting Examples

### Test Platform Selection
```bash
# Pull and verify on different platforms
docker pull --platform linux/amd64 myapp:latest
docker inspect myapp:latest --format '{{.Architecture}}'

docker pull --platform linux/arm64 myapp:latest
docker inspect myapp:latest --format '{{.Architecture}}'

# Test on actual hardware
docker run --rm myapp:latest uname -m
```

### Debug Manifest Issues
```bash
# Check if manifest exists locally
docker manifest inspect myapp:latest 2>&1

# Verify platform images exist
docker manifest inspect myapp:amd64
docker manifest inspect myapp:arm64

# Check registry for manifest
curl -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
    https://registry.hub.docker.com/v2/myapp/manifests/latest

# Inspect with verbose
docker manifest inspect -v myapp:latest | jq
```

### Cleanup Test Manifests
```bash
# List local manifests (no direct command, check images)
docker images | grep manifest

# Remove test manifests
docker manifest rm myapp:test
docker manifest rm myapp:dev
docker manifest rm myapp:staging

# Clean up platform images
docker rmi myapp:amd64 myapp:arm64 myapp:armv7
```

## Advanced Use Cases

### Cross-Registry Manifest
```bash
# Build in one registry, manifest in another
docker build --platform linux/amd64 -t registry1.com/myapp:amd64 .
docker build --platform linux/arm64 -t registry2.com/myapp:arm64 .

docker push registry1.com/myapp:amd64
docker push registry2.com/myapp:arm64

# Create manifest pointing to different registries
docker manifest create myapp:latest \
    registry1.com/myapp:amd64 \
    registry2.com/myapp:arm64

docker manifest push myregistry.com/myapp:latest
```

### Conditional Platform Builds
```bash
#!/bin/bash
# Build only supported platforms

PLATFORMS=("linux/amd64" "linux/arm64" "linux/arm/v7")
TAG="myapp:multiplatform"

for PLATFORM in "${PLATFORMS[@]}"; do
    ARCH=$(echo $PLATFORM | cut -d'/' -f2)
    VARIANT=$(echo $PLATFORM | cut -d'/' -f3)
    IMAGE_TAG="myapp:${ARCH}${VARIANT:+-$VARIANT}"
    
    echo "Building $PLATFORM as $IMAGE_TAG"
    docker build --platform $PLATFORM -t $IMAGE_TAG . || echo "Failed to build $PLATFORM"
    docker push $IMAGE_TAG || echo "Failed to push $IMAGE_TAG"
done

# Create manifest from successfully built images
docker manifest create $TAG \
    $(docker images myapp --format "{{.Repository}}:{{.Tag}}" | grep -v multiplatform)

docker manifest push $TAG
```

### Automated Manifest Creation Script
```bash
#!/bin/bash
# auto-manifest.sh - Automated multi-platform manifest creation

IMAGE_NAME=${1:-myapp}
TAG=${2:-latest}
PLATFORMS=("amd64" "arm64" "armv7")

export DOCKER_CLI_EXPERIMENTAL=enabled

echo "Creating manifest for $IMAGE_NAME:$TAG"

# Build manifest create command
MANIFEST_CMD="docker manifest create $IMAGE_NAME:$TAG"
for PLATFORM in "${PLATFORMS[@]}"; do
    MANIFEST_CMD="$MANIFEST_CMD $IMAGE_NAME:$PLATFORM"
done

# Create manifest
eval $MANIFEST_CMD

# Annotate each platform
for PLATFORM in "${PLATFORMS[@]}"; do
    case $PLATFORM in
        amd64)
            docker manifest annotate $IMAGE_NAME:$TAG $IMAGE_NAME:$PLATFORM \
                --os linux --arch amd64
            ;;
        arm64)
            docker manifest annotate $IMAGE_NAME:$TAG $IMAGE_NAME:$PLATFORM \
                --os linux --arch arm64 --variant v8
            ;;
        armv7)
            docker manifest annotate $IMAGE_NAME:$TAG $IMAGE_NAME:$PLATFORM \
                --os linux --arch arm --variant v7
            ;;
    esac
done

# Inspect and push
docker manifest inspect $IMAGE_NAME:$TAG
docker manifest push $IMAGE_NAME:$TAG

echo "Manifest created and pushed successfully"
```

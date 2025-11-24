# Examples

## Basic Examples

### Simple Build
```bash
# Build image from Dockerfile in current directory
docker build .

# Build and tag the image
docker build -t myapp:1.0 .

# Build with specific Dockerfile
docker build -f Dockerfile.dev -t myapp:dev .

# Build from different directory
docker build -t myapp:1.0 /path/to/build/context

# Build with custom tag format
docker build -t myregistry.com/myapp:1.0.0 .
```

### Multiple Tags
```bash
# Tag image with multiple names
docker build -t myapp:1.0 -t myapp:latest .

# Tag with version and latest
docker build -t myapp:1.0.0 -t myapp:1.0 -t myapp:latest .

# Tag for different registries
docker build -t myregistry.com/myapp:1.0 -t dockerhub.com/user/myapp:1.0 .
```

### Build Arguments
```bash
# Pass build argument
docker build --build-arg VERSION=1.0 -t myapp .

# Multiple build arguments
docker build --build-arg ENV=production --build-arg DEBUG=false -t myapp .

# Build arguments from environment
export APP_VERSION=1.0.0
docker build --build-arg VERSION=$APP_VERSION -t myapp:$APP_VERSION .
```

### Cache Control
```bash
# Build without using cache
docker build --no-cache -t myapp:1.0 .

# Pull newer base image before building
docker build --pull -t myapp:1.0 .

# Use specific image as cache source
docker build --cache-from myapp:latest -t myapp:2.0 .

# Combine cache options
docker build --pull --cache-from myapp:latest -t myapp:2.0 .
```

## Advanced Examples

### Multi-Stage Builds
```dockerfile
# Example Dockerfile with multi-stage build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

```bash
# Build multi-stage image
docker build -t myapp:1.0 .

# Build specific stage only
docker build --target builder -t myapp:builder .

# Build and use previous stage as cache
docker build --cache-from myapp:builder --target builder -t myapp:builder .
docker build --cache-from myapp:builder -t myapp:1.0 .
```

### Build from Git Repository
```bash
# Build from GitHub repository
docker build -t myapp https://github.com/user/repo.git

# Build from specific branch
docker build -t myapp https://github.com/user/repo.git#develop

# Build from specific tag
docker build -t myapp https://github.com/user/repo.git#v1.0.0

# Build from subdirectory in repo
docker build -t myapp https://github.com/user/repo.git#main:docker

# Build with specific Dockerfile from repo
docker build -t myapp -f Dockerfile.prod https://github.com/user/repo.git
```

### Build from Stdin
```bash
# Build from Dockerfile via stdin (no build context)
docker build -t myapp - < Dockerfile

# Build with context from stdin (tar archive)
tar -czf - . | docker build -t myapp -

# Build from URL
docker build -t myapp https://example.com/dockerfile.tar.gz

# Build from stdin with context
cat Dockerfile | docker build -t myapp -f - .
```

### Platform-Specific Builds
```bash
# Build for specific platform
docker build --platform linux/amd64 -t myapp:amd64 .

# Build for ARM architecture
docker build --platform linux/arm64 -t myapp:arm64 .

# Build multi-platform image (requires buildx)
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .
```

### Build with Labels
```bash
# Add single label
docker build --label version=1.0 -t myapp .

# Add multiple labels
docker build \
  --label version=1.0 \
  --label maintainer="user@example.com" \
  --label description="My application" \
  -t myapp .

# Labels from file
echo "version=1.0" > labels.txt
echo "build-date=$(date -u +'%Y-%m-%dT%H:%M:%SZ')" >> labels.txt
docker build --label-file labels.txt -t myapp .
```

### Build with Network Options
```bash
# Build with host network
docker build --network host -t myapp .

# Build with custom network
docker network create build-network
docker build --network build-network -t myapp .

# Build without network access
docker build --network none -t myapp .

# Add custom host mapping
docker build --add-host myhost:192.168.1.100 -t myapp .
```

## Common Use Cases

### CI/CD Pipeline Build
```bash
#!/bin/bash
# CI/CD build script with versioning
set -e

# Get version from git
VERSION=$(git describe --tags --always --dirty)
BRANCH=$(git rev-parse --abbrev-ref HEAD)
COMMIT=$(git rev-parse --short HEAD)
BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')

echo "Building version: $VERSION"

# Build with metadata
docker build \
  --build-arg VERSION=$VERSION \
  --build-arg BUILD_DATE=$BUILD_DATE \
  --build-arg VCS_REF=$COMMIT \
  --label "org.opencontainers.image.version=$VERSION" \
  --label "org.opencontainers.image.created=$BUILD_DATE" \
  --label "org.opencontainers.image.revision=$COMMIT" \
  -t myapp:$VERSION \
  -t myapp:$COMMIT \
  -t myapp:latest \
  .

echo "Build completed: myapp:$VERSION"

# Run tests on built image
docker run --rm myapp:$VERSION npm test

echo "Tests passed. Ready to push."
```

### 2. Multi-Environment Build
```bash
#!/bin/bash
# Build for different environments
environment="$1"

if [ -z "$environment" ]; then
    echo "Usage: $0 <development|staging|production>"
    exit 1
fi

case $environment in
    development)
        DOCKERFILE="Dockerfile.dev"
        TAG="myapp:dev"
        BUILD_ARGS="--build-arg ENV=development --build-arg DEBUG=true"
        ;;
    staging)
        DOCKERFILE="Dockerfile"
        TAG="myapp:staging"
        BUILD_ARGS="--build-arg ENV=staging --build-arg DEBUG=false"
        ;;
    production)
        DOCKERFILE="Dockerfile"
        TAG="myapp:prod"
        BUILD_ARGS="--build-arg ENV=production --build-arg DEBUG=false --no-cache"
        ;;
    *)
        echo "Unknown environment: $environment"
        exit 1
        ;;
esac

echo "Building for $environment environment"
docker build $BUILD_ARGS -f $DOCKERFILE -t $TAG .

echo "Build completed: $TAG"
```

### 3. Optimized Build with Cache
```bash
#!/bin/bash
# Build script with intelligent caching
IMAGE_NAME="myapp"
VERSION="1.0.0"
CACHE_IMAGE="${IMAGE_NAME}:cache"

echo "=== Building $IMAGE_NAME:$VERSION ==="

# Try to pull previous version for cache
echo "Pulling cache image..."
if docker pull $CACHE_IMAGE 2>/dev/null; then
    echo "Using $CACHE_IMAGE as cache source"
    CACHE_FLAG="--cache-from $CACHE_IMAGE"
else
    echo "No cache image found, building from scratch"
    CACHE_FLAG=""
fi

# Build with cache
echo "Building image..."
BUILD_START=$(date +%s)

docker build \
  $CACHE_FLAG \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  -t $IMAGE_NAME:$VERSION \
  -t $IMAGE_NAME:latest \
  -t $CACHE_IMAGE \
  .

BUILD_END=$(date +%s)
BUILD_TIME=$((BUILD_END - BUILD_START))

echo "Build completed in ${BUILD_TIME}s"
echo "Image size: $(docker images $IMAGE_NAME:$VERSION --format '{{.Size}}')"

# Push cache image for future builds
echo "Pushing cache image..."
docker push $CACHE_IMAGE
```

### 4. Build with Tests
```bash
#!/bin/bash
# Build and test Docker image
set -e

IMAGE_NAME="myapp"
VERSION="${1:-latest}"

echo "=== Building and Testing $IMAGE_NAME:$VERSION ==="

# Build the image
echo "Step 1: Building image..."
docker build -t $IMAGE_NAME:$VERSION .

# Run security scan
echo "Step 2: Security scanning..."
if command -v trivy &> /dev/null; then
    trivy image --severity HIGH,CRITICAL $IMAGE_NAME:$VERSION
else
    echo "Trivy not installed, skipping security scan"
fi

# Run unit tests
echo "Step 3: Running unit tests..."
docker run --rm $IMAGE_NAME:$VERSION npm test

# Run integration tests
echo "Step 4: Running integration tests..."
docker run --rm -e NODE_ENV=test $IMAGE_NAME:$VERSION npm run test:integration

# Check image size
echo "Step 5: Checking image size..."
IMAGE_SIZE=$(docker images $IMAGE_NAME:$VERSION --format '{{.Size}}')
echo "Image size: $IMAGE_SIZE"

# Verify image layers
echo "Step 6: Analyzing image layers..."
docker history $IMAGE_NAME:$VERSION --human

echo "\n=== All checks passed ==="
echo "Image $IMAGE_NAME:$VERSION is ready for deployment"
```

### 5. Parallel Multi-Platform Build
```bash
#!/bin/bash
# Build for multiple platforms in parallel
set -e

IMAGE_NAME="myapp"
VERSION="1.0.0"
PLATFORMS=("linux/amd64" "linux/arm64" "linux/arm/v7")

echo "=== Building $IMAGE_NAME:$VERSION for multiple platforms ==="

# Create builder if it doesn't exist
if ! docker buildx ls | grep -q multiplatform-builder; then
    docker buildx create --name multiplatform-builder --use
    docker buildx inspect --bootstrap
fi

# Build for all platforms
echo "Building for platforms: ${PLATFORMS[*]}"
docker buildx build \
  --platform $(IFS=,; echo "${PLATFORMS[*]}") \
  --build-arg VERSION=$VERSION \
  -t $IMAGE_NAME:$VERSION \
  -t $IMAGE_NAME:latest \
  --push \
  .

echo "Multi-platform build completed"

# Verify manifests
echo "\nVerifying platform manifests:"
docker buildx imagetools inspect $IMAGE_NAME:$VERSION
```

### 6. Build with Secrets
```bash
#!/bin/bash
# Build with secrets (requires BuildKit)
export DOCKER_BUILDKIT=1

IMAGE_NAME="myapp"

# Create secret files
echo "Creating temporary secret files..."
echo "$NPM_TOKEN" > .npm-token
echo "$SSH_PRIVATE_KEY" > .ssh-key

# Build with secrets
echo "Building with secrets..."
docker build \
  --secret id=npm,src=.npm-token \
  --secret id=ssh,src=.ssh-key \
  --ssh default \
  -t $IMAGE_NAME:latest \
  .

# Clean up secrets
echo "Cleaning up secret files..."
rm -f .npm-token .ssh-key

echo "Build completed successfully"
```

### 7. Automated Nightly Build
```bash
#!/bin/bash
# Nightly build script
set -e

IMAGE_NAME="myapp"
DATE=$(date +%Y%m%d)
VERSION="nightly-$DATE"

echo "=== Nightly Build: $VERSION ==="
echo "Started: $(date)"

# Pull latest code
echo "Pulling latest code..."
git fetch origin
git checkout main
git pull origin main

# Clean old images
echo "Cleaning old nightly builds..."
docker images $IMAGE_NAME --filter "label=build-type=nightly" -q | xargs -r docker rmi -f

# Build with full cache refresh
echo "Building image..."
docker build \
  --pull \
  --no-cache \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg VERSION=$VERSION \
  --label "build-type=nightly" \
  --label "build-date=$DATE" \
  -t $IMAGE_NAME:$VERSION \
  -t $IMAGE_NAME:nightly \
  .

# Run smoke tests
echo "Running smoke tests..."
docker run --rm $IMAGE_NAME:$VERSION /app/smoke-test.sh

# Push to registry
echo "Pushing to registry..."
docker push $IMAGE_NAME:$VERSION
docker push $IMAGE_NAME:nightly

# Generate build report
REPORT_FILE="build-report-$DATE.txt"
cat > $REPORT_FILE <<EOF
Nightly Build Report
====================
Date: $(date)
Version: $VERSION
Image: $IMAGE_NAME:$VERSION
Size: $(docker images $IMAGE_NAME:$VERSION --format '{{.Size}}')
Git Commit: $(git rev-parse HEAD)
Build Status: SUCCESS
EOF

echo "Build report saved to $REPORT_FILE"
echo "Nightly build completed: $(date)"
```

### 8. Layer Analysis and Optimization
```bash
#!/bin/bash
# Analyze and optimize Docker image layers
IMAGE_NAME="$1"

if [ -z "$IMAGE_NAME" ]; then
    echo "Usage: $0 <image_name>"
    exit 1
fi

echo "=== Docker Image Analysis: $IMAGE_NAME ==="

# Show image size
echo "\nImage Size:"
docker images $IMAGE_NAME --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Show layer details
echo "\nLayer History:"
docker history $IMAGE_NAME --human --no-trunc

# Analyze large layers
echo "\nLarge Layers (>10MB):"
docker history $IMAGE_NAME --human | awk '$2 ~ /MB/ {size=$2; gsub(/MB/, "", size); if (size+0 > 10) print}'

# Use dive if available
if command -v dive &> /dev/null; then
    echo "\nRunning dive analysis..."
    dive $IMAGE_NAME --ci
else
    echo "\nNote: Install 'dive' for detailed layer analysis"
    echo "  https://github.com/wagoodman/dive"
fi

# Check for common issues
echo "\nChecking for common issues:"

# Check for apt cache
if docker run --rm $IMAGE_NAME ls /var/lib/apt/lists/ 2>/dev/null | grep -q .; then
    echo "  ⚠️  Found apt cache - consider cleaning with 'rm -rf /var/lib/apt/lists/*'"
fi

# Check for package manager cache
if docker run --rm $IMAGE_NAME ls /root/.cache 2>/dev/null | grep -q .; then
    echo "  ⚠️  Found cache in /root/.cache - consider removing"
fi

echo "\nAnalysis complete"
```
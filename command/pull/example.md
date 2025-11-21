# Docker pull Examples

## Basic Examples

### Pull Latest Image
```bash
# Pull latest version of ubuntu
docker pull ubuntu

# Equivalent to
docker pull ubuntu:latest

# Pull specific version
docker pull ubuntu:20.04

# Pull using digest
docker pull ubuntu@sha256:abcd1234...
```

### Pull from Different Registries
```bash
# Pull from Docker Hub (default)
docker pull nginx

# Pull from Google Container Registry
docker pull gcr.io/project/image:tag

# Pull from Amazon ECR
docker pull 123456789012.dkr.ecr.region.amazonaws.com/my-app:latest

# Pull from Azure Container Registry
docker pull myregistry.azurecr.io/myapp:v1.0

# Pull from private registry
docker pull myregistry.com:5000/myapp:latest
```

### Pull Multiple Tags
```bash
# Pull all tags for a repository
docker pull -a ubuntu

# Pull specific multiple tags
docker pull ubuntu:18.04
docker pull ubuntu:20.04
docker pull ubuntu:22.04

# Pull multiple images in sequence
for tag in 18.04 20.04 22.04; do
    docker pull ubuntu:$tag
done
```

## Platform-Specific Examples

### Multi-Platform Images
```bash
# Pull for specific platform
docker pull --platform linux/amd64 alpine
docker pull --platform linux/arm64 alpine
docker pull --platform linux/arm/v7 alpine

# Pull for current platform (default)
docker pull alpine

# Check available platforms
docker manifest inspect nginx | jq '.manifests[].platform'
```

### Architecture-Specific Pulls
```bash
# Pull for ARM architecture
docker pull --platform linux/arm64 nginx

# Pull for different architectures
docker pull --platform linux/amd64 mysql:8.0
docker pull --platform linux/arm64/v8 mysql:8.0

# Pull Windows containers (on Windows host)
docker pull --platform windows/amd64 mcr.microsoft.com/windows/servercore
```

## Advanced Examples

### Quiet and Verbose Pulls
```bash
# Quiet pull (minimal output)
docker pull -q ubuntu:20.04

# Normal pull with progress
docker pull ubuntu:20.04

# Pull with debugging (using Docker debug mode)
DOCKER_BUILDKIT=0 docker pull ubuntu:20.04
```

### Authenticated Pulls
```bash
# Login to registry first
docker login myregistry.com

# Then pull private image
docker pull myregistry.com/private/app:latest

# Pull with inline registry specification
docker pull username/private-repo:tag

# Logout after use
docker logout myregistry.com
```

### Conditional Pulls
```bash
# Pull only if image doesn't exist locally
image="nginx:1.20"
if ! docker images --format "{{.Repository}}:{{.Tag}}" | grep -q "^$image$"; then
    echo "Pulling $image..."
    docker pull "$image"
else
    echo "$image already exists locally"
fi
```

### Batch Pull Operations
```bash
# Pull multiple related images
images=(
    "nginx:1.20"
    "nginx:1.21"
    "nginx:alpine"
    "nginx:stable"
)

for image in "${images[@]}"; do
    echo "Pulling $image..."
    docker pull "$image"
done
```

## Real-World Use Cases

### 1. Development Environment Setup
```bash
#!/bin/bash
# Development environment image pull script
echo "Setting up development environment..."

# Base images
docker pull ubuntu:20.04
docker pull alpine:latest
docker pull node:16-alpine

# Database images
docker pull postgres:13
docker pull redis:6-alpine
docker pull mysql:8.0

# Tool images
docker pull nginx:alpine
docker pull traefik:latest

echo "Development images ready"
```

### 2. CI/CD Pipeline Preparation
```bash
#!/bin/bash
# CI/CD pipeline image preparation
project="myapp"
versions=("1.0.0" "1.1.0" "latest")
registry="myregistry.com"

echo "Preparing CI/CD images for $project..."

# Pull base images
docker pull node:16-alpine
docker pull nginx:alpine

# Pull application images
for version in "${versions[@]}"; do
    echo "Pulling $registry/$project:$version..."
    docker pull "$registry/$project:$version" || echo "Warning: Failed to pull $version"
done

# Pull testing tools
docker pull selenium/standalone-chrome
docker pull postman/newman

echo "CI/CD preparation completed"
```

### 3. Production Deployment
```bash
#!/bin/bash
# Production deployment image pull
set -e  # Exit on any error

REGISTRY="production.registry.com"
APP_NAME="webapp"
VERSION="${1:-latest}"

echo "Pulling production images for deployment..."

# Pull with verification
echo "Pulling $REGISTRY/$APP_NAME:$VERSION..."
if docker pull "$REGISTRY/$APP_NAME:$VERSION"; then
    echo "✓ Application image pulled successfully"
else
    echo "✗ Failed to pull application image"
    exit 1
fi

# Pull supporting services
echo "Pulling supporting services..."
docker pull "$REGISTRY/nginx-proxy:stable"
docker pull "$REGISTRY/monitoring:latest"

# Verify images
echo "Verifying pulled images..."
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedAt}}" | grep "$REGISTRY"

echo "Production images ready for deployment"
```

### 4. Automated Update Script
```bash
#!/bin/bash
# Automated image update script
echo "Checking for image updates..."

# List of critical images to keep updated
critical_images=(
    "nginx:latest"
    "postgres:13"
    "redis:6-alpine"
    "ubuntu:20.04"
)

for image in "${critical_images[@]}"; do
    echo "Checking $image..."
    
    # Get current image ID
    current_id=$(docker images --format "{{.ID}}" "$image" 2>/dev/null)
    
    # Pull latest version
    echo "Pulling latest $image..."
    docker pull "$image"
    
    # Get new image ID
    new_id=$(docker images --format "{{.ID}}" "$image" 2>/dev/null)
    
    # Check if image was updated
    if [ "$current_id" != "$new_id" ]; then
        echo "✓ $image was updated (old: $current_id, new: $new_id)"
        
        # Optional: restart containers using this image
        containers=$(docker ps --format "{{.Names}}" --filter "ancestor=$image")
        if [ -n "$containers" ]; then
            echo "Containers using $image: $containers"
            echo "Consider restarting these containers"
        fi
    else
        echo "○ $image is already up to date"
    fi
done

echo "Update check completed"
```

## Security and Verification Examples

### Content Trust and Verification
```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Pull with signature verification
docker pull alpine:latest

# Disable for specific pull
DOCKER_CONTENT_TRUST=0 docker pull untrusted/image:latest

# Manual signature verification
docker trust inspect alpine:latest
```

### Vulnerability Scanning
```bash
# Pull and scan image for vulnerabilities
image="nginx:latest"
echo "Pulling and scanning $image..."

# Pull image
docker pull "$image"

# Scan for vulnerabilities (if scanner available)
if command -v docker scan >/dev/null 2>&1; then
    docker scan "$image"
else
    echo "Docker scan not available, using alternative scanner"
    # Alternative: use trivy or other scanner
    trivy image "$image" 2>/dev/null || echo "Trivy not available"
fi
```

## Scripting and Automation

### Parallel Pull Script
```bash
#!/bin/bash
# Parallel image pulling for faster setup
images=(
    "ubuntu:20.04"
    "nginx:alpine"
    "postgres:13"
    "redis:6-alpine"
    "node:16-alpine"
)

echo "Pulling ${#images[@]} images in parallel..."

# Function to pull single image
pull_image() {
    local image="$1"
    echo "Starting pull: $image"
    if docker pull "$image"; then
        echo "✓ Completed: $image"
    else
        echo "✗ Failed: $image"
    fi
}

# Start parallel pulls
for image in "${images[@]}"; do
    pull_image "$image" &
done

# Wait for all pulls to complete
wait
echo "All pulls completed"
```

### Registry Migration Script
```bash
#!/bin/bash
# Migrate images from one registry to another
source_registry="old-registry.com"
target_registry="new-registry.com"
namespace="myorg"

images_to_migrate=(
    "webapp:latest"
    "api:v2.0"
    "worker:stable"
)

for image in "${images_to_migrate[@]}"; do
    source_image="$source_registry/$namespace/$image"
    target_image="$target_registry/$namespace/$image"
    
    echo "Migrating $source_image -> $target_image"
    
    # Pull from source
    docker pull "$source_image"
    
    # Tag for target
    docker tag "$source_image" "$target_image"
    
    # Push to target
    docker push "$target_image"
    
    # Optional: remove source image locally
    # docker rmi "$source_image"
    
    echo "✓ Migrated $image"
done
```

### Image Cache Warming
```bash
#!/bin/bash
# Warm up image cache on multiple nodes
nodes=("node1.example.com" "node2.example.com" "node3.example.com")
images=("nginx:alpine" "postgres:13" "redis:6-alpine")

warm_cache() {
    local node="$1"
    local image="$2"
    
    echo "Warming cache on $node for $image"
    ssh "$node" "docker pull $image" || echo "Failed to warm $image on $node"
}

echo "Warming image cache across cluster..."

for node in "${nodes[@]}"; do
    for image in "${images[@]}"; do
        warm_cache "$node" "$image" &
    done
done

wait
echo "Cache warming completed"
```
# Troubleshooting

## Common Issues

### Experimental Feature Not Enabled
**Problem**: Docker manifest command not recognized
```bash
# Error message
docker manifest create myapp:latest myapp:amd64
# docker: 'manifest' is not a docker command.

# Check if experimental is enabled
docker version --format '{{.Client.Experimental}}'

# Enable temporarily
export DOCKER_CLI_EXPERIMENTAL=enabled

# Enable permanently in config file
mkdir -p ~/.docker
cat > ~/.docker/config.json <<EOF
{
  "experimental": "enabled"
}
EOF

# Verify it works
docker manifest --help
```

### Manifest Not Found
**Problem**: Cannot inspect or push manifest that doesn't exist
```bash
# Error message
docker manifest inspect myapp:latest
# no such manifest: docker.io/library/myapp:latest

# Check if image exists locally
docker images myapp:latest

# Pull the image first
docker pull myapp:latest

# Check with full registry path
docker manifest inspect docker.io/username/myapp:latest

# Verify registry and credentials
docker login

# List available tags
curl -s https://registry.hub.docker.com/v2/repositories/username/myapp/tags | jq
```

### Platform-Specific Image Not Found
**Problem**: Cannot create manifest because platform image doesn't exist
```bash
# Error message
docker manifest create myapp:latest myapp:amd64 myapp:arm64
# Error response from daemon: manifest for myapp:amd64 not found

# Verify platform images exist
docker images | grep myapp

# Build missing platform images
docker build --platform linux/amd64 -t myapp:amd64 .
docker build --platform linux/arm64 -t myapp:arm64 .

# Push platform images to registry
docker push myapp:amd64
docker push myapp:arm64

# Then create manifest
docker manifest create myapp:latest myapp:amd64 myapp:arm64

# Verify images exist in registry
docker manifest inspect myapp:amd64
docker manifest inspect myapp:arm64
```

### Cannot Push Manifest
**Problem**: Access denied when pushing manifest to registry
```bash
# Error message
docker manifest push myapp:latest
# denied: requested access to the resource is denied

# Login to registry
docker login

# Check registry permissions
docker login registry.example.com

# Use full image path with registry
docker manifest create registry.example.com/username/myapp:latest \
    registry.example.com/username/myapp:amd64 \
    registry.example.com/username/myapp:arm64

docker manifest push registry.example.com/username/myapp:latest

# For Docker Hub, include username
docker manifest create username/myapp:latest \
    username/myapp:amd64 \
    username/myapp:arm64

# Verify credentials
cat ~/.docker/config.json
```

### Wrong Platform Pulled
**Problem**: Docker pulls incorrect architecture for the host system
```bash
# Check what was pulled
docker inspect myapp:latest --format '{{.Os}}/{{.Architecture}}'

# Check manifest list platforms
docker manifest inspect myapp:latest | jq -r '.manifests[].platform'

# Check available platforms in manifest
docker manifest inspect myapp:latest | \
    jq '.manifests[] | {platform: .platform, digest: .digest}'

# Verify annotations are correct
docker manifest inspect myapp:latest | \
    jq '.manifests[] | select(.platform.architecture=="arm64")'

# If platform missing, rebuild manifest
docker manifest rm myapp:latest
docker manifest create myapp:latest myapp:amd64 myapp:arm64

# Annotate correctly
docker manifest annotate myapp:latest myapp:amd64 \
    --os linux --arch amd64

docker manifest annotate myapp:latest myapp:arm64 \
    --os linux --arch arm64

# Push corrected manifest
docker manifest push myapp:latest

# Test pull on specific platform
docker pull --platform linux/arm64 myapp:latest
```

### Manifest Already Exists
**Problem**: Cannot create manifest because it already exists locally
```bash
# Error message
docker manifest create myapp:latest myapp:amd64 myapp:arm64
# Error: myapp:latest: manifest list already exists

# Option 1: Use --amend flag to update
docker manifest create --amend myapp:latest \
    myapp:amd64 \
    myapp:arm64

# Option 2: Remove existing manifest first
docker manifest rm myapp:latest
docker manifest create myapp:latest \
    myapp:amd64 \
    myapp:arm64

# Option 3: Check existing manifest
docker manifest inspect myapp:latest

# Verify what's in the manifest
docker manifest inspect myapp:latest | jq '.manifests[].platform'
```

### Annotation Not Applied
**Problem**: Platform annotations don't appear in manifest
```bash
# Check current manifest
docker manifest inspect myapp:latest | \
    jq '.manifests[] | select(.digest | contains("armv7"))'

# Verify image is in manifest list
docker manifest inspect myapp:latest | jq '.manifests[].platform'

# Annotations must be done BEFORE pushing
# Remove manifest if already pushed
docker manifest rm myapp:latest

# Create manifest
docker manifest create myapp:latest \
    myapp:amd64 \
    myapp:arm64 \
    myapp:armv7

# Annotate (before pushing!)
docker manifest annotate myapp:latest myapp:armv7 \
    --os linux \
    --arch arm \
    --variant v7

# Verify annotation
docker manifest inspect myapp:latest | \
    jq '.manifests[] | select(.platform.architecture=="arm")'

# Now push
docker manifest push myapp:latest
```

### Cross-Platform Build Fails
**Problem**: Building for non-native architecture fails
```bash
# Error message
docker build --platform linux/arm64 -t myapp:arm64 .
# exec format error

# Install QEMU for emulation
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

# Verify QEMU is working
docker run --rm --platform linux/arm64 alpine uname -m
# Should output: aarch64

# Try build again
docker build --platform linux/arm64 -t myapp:arm64 .

# Alternative: Use buildx (recommended)
docker buildx create --name multiplatform --use
docker buildx build --platform linux/amd64,linux/arm64 \
    -t myapp:latest .

# Or build on native platform
# Use ARM-based CI runners for ARM builds
```

## Debugging Commands

### Manifest Inspection
```bash
# Check if manifest is a manifest list
docker manifest inspect myapp:latest | jq '.mediaType'
# Should be: application/vnd.docker.distribution.manifest.list.v2+json

# List all platforms in manifest
docker manifest inspect myapp:latest | \
    jq -r '.manifests[] | "\(.platform.os)/\(.platform.architecture)/\(.platform.variant // "none")"'

# Check platform digests
docker manifest inspect myapp:latest | \
    jq '.manifests[] | {platform: .platform, digest: .digest}'

# Verify each platform image exists
docker manifest inspect myapp:latest | jq '.manifests[].digest' | \
    while read digest; do
        echo "Checking $digest"
        docker manifest inspect myapp@$(echo $digest | tr -d '"')
    done

# Export manifest details
docker manifest inspect myapp:latest > manifest-$(date +%Y%m%d).json
```

### Platform Verification
```bash
# Check current system platform
docker version --format '{{.Server.Os}}/{{.Server.Arch}}'
uname -m

# Test pulling different platforms
docker pull --platform linux/amd64 myapp:latest
docker inspect myapp:latest --format '{{.Architecture}}'

docker pull --platform linux/arm64 myapp:latest
docker inspect myapp:latest --format '{{.Architecture}}'

# Verify platform images work
docker run --rm --platform linux/amd64 myapp:latest uname -m
docker run --rm --platform linux/arm64 myapp:latest uname -m

# Check image layers by platform
docker manifest inspect myapp:amd64 | jq '.layers'
docker manifest inspect myapp:arm64 | jq '.layers'
```

### Registry Communication
```bash
# Test registry connectivity
curl -v https://registry.hub.docker.com/v2/

# Check authentication
docker login
cat ~/.docker/config.json

# Verify manifest exists in registry
curl -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
    -H "Authorization: Bearer $(echo $TOKEN)" \
    https://registry.hub.docker.com/v2/username/myapp/manifests/latest

# Check available tags
curl -s https://registry.hub.docker.com/v2/repositories/username/myapp/tags | jq

# Test insecure registry (if needed)
docker manifest inspect --insecure registry.local:5000/myapp:latest
```

### Build Environment Check
```bash
# Check BuildKit status
docker buildx ls

# Verify QEMU installation
docker run --rm --privileged multiarch/qemu-user-static --version

# Check available platforms
docker buildx inspect --bootstrap | grep Platforms

# Test cross-platform support
docker run --rm --platform linux/arm64 alpine uname -m
docker run --rm --platform linux/amd64 alpine uname -m

# Check Docker version
docker version
docker buildx version
```

## Error Messages and Solutions

### "no such manifest"
```bash
# Error: no such manifest: myapp:latest

# Solution 1: Create the manifest first
docker manifest create myapp:latest myapp:amd64 myapp:arm64

# Solution 2: Check registry path
docker manifest inspect docker.io/username/myapp:latest

# Solution 3: Pull image first if inspecting
docker pull myapp:latest
docker manifest inspect myapp:latest

# Solution 4: Verify credentials and permissions
docker login
docker manifest inspect username/myapp:latest
```

### "manifest unknown"
```bash
# Error: manifest unknown: manifest unknown

# Platform image not in registry
docker push myapp:amd64
docker push myapp:arm64

# Then create manifest
docker manifest create myapp:latest myapp:amd64 myapp:arm64

# Verify images exist
docker manifest inspect myapp:amd64
docker manifest inspect myapp:arm64
```

### "denied: requested access"
```bash
# Error: denied: requested access to the resource is denied

# Login to registry
docker login

# For private registry
docker login registry.example.com -u username

# Check write permissions
# Must have push access to repository

# Use correct image path
docker manifest create username/myapp:latest \
    username/myapp:amd64 \
    username/myapp:arm64

# For organization repos
docker manifest create organization/myapp:latest \
    organization/myapp:amd64 \
    organization/myapp:arm64
```

### "unsupported manifest media type"
```bash
# Error: unsupported manifest media type

# Check manifest type
docker manifest inspect myapp:latest | jq '.mediaType'

# Rebuild manifest list
docker manifest rm myapp:latest
docker manifest create myapp:latest myapp:amd64 myapp:arm64
docker manifest push myapp:latest

# Use Docker API v2 schema
# Ensure registry supports manifest lists

# Update Docker to latest version
docker version
```

### "exec format error"
```bash
# Error: standard_init_linux.go: exec format error

# Install QEMU for cross-compilation
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

# Verify QEMU
ls /proc/sys/fs/binfmt_misc/

# Enable BuildKit
export DOCKER_BUILDKIT=1

# Use buildx for better cross-compilation
docker buildx create --use
docker buildx build --platform linux/arm64 -t myapp:arm64 .

# Or build on native platform
# Use ARM hardware or ARM-based CI runners
```

## Recovery Procedures

### Recreate Broken Manifest
```bash
# Step 1: Remove broken manifest
docker manifest rm myapp:latest

# Step 2: Verify platform images exist
docker images | grep myapp
docker manifest inspect myapp:amd64
docker manifest inspect myapp:arm64

# Step 3: Pull platform images if needed
docker pull myapp:amd64
docker pull myapp:arm64

# Step 4: Recreate manifest
docker manifest create myapp:latest myapp:amd64 myapp:arm64

# Step 5: Annotate platforms
docker manifest annotate myapp:latest myapp:amd64 \
    --os linux --arch amd64

docker manifest annotate myapp:latest myapp:arm64 \
    --os linux --arch arm64

# Step 6: Verify before pushing
docker manifest inspect myapp:latest | jq

# Step 7: Push to registry
docker manifest push myapp:latest

# Step 8: Verify in registry
docker manifest inspect myapp:latest
```

### Fix Missing Platform
```bash
# Step 1: Identify missing platform
docker manifest inspect myapp:latest | jq '.manifests[].platform'

# Step 2: Build missing platform
docker build --platform linux/arm/v7 -t myapp:armv7 .

# Step 3: Push platform image
docker push myapp:armv7

# Step 4: Update manifest with --amend
docker manifest create --amend myapp:latest \
    myapp:amd64 \
    myapp:arm64 \
    myapp:armv7

# Step 5: Annotate new platform
docker manifest annotate myapp:latest myapp:armv7 \
    --os linux --arch arm --variant v7

# Step 6: Push updated manifest
docker manifest push myapp:latest

# Step 7: Verify all platforms
docker manifest inspect myapp:latest | \
    jq '.manifests[] | .platform'
```

### Migrate to Buildx
```bash
# If manifest commands are problematic, switch to buildx

# Step 1: Install buildx (if not already)
docker buildx version

# Step 2: Create builder instance
docker buildx create --name multiplatform --use

# Step 3: Build for all platforms at once
docker buildx build \
    --platform linux/amd64,linux/arm64,linux/arm/v7 \
    -t myapp:latest \
    --push \
    .

# Step 4: Verify result
docker buildx imagetools inspect myapp:latest

# Step 5: Test pull
docker pull myapp:latest
docker inspect myapp:latest --format '{{.Os}}/{{.Architecture}}'

# Benefits:
# - Simpler workflow
# - Automatic manifest creation
# - Better caching
# - Parallel builds
```

### Automated Recovery Script
```bash
#!/bin/bash
# manifest-recovery.sh - Recover broken manifest

IMAGE_NAME=$1
TAG=${2:-latest}

if [ -z "$IMAGE_NAME" ]; then
    echo "Usage: $0 <image_name> [tag]"
    exit 1
fi

echo "=== Manifest Recovery for $IMAGE_NAME:$TAG ==="

# Enable experimental
export DOCKER_CLI_EXPERIMENTAL=enabled

# Remove broken manifest
echo "Removing existing manifest..."
docker manifest rm $IMAGE_NAME:$TAG 2>/dev/null

# Check platform images
echo "Checking platform images..."
PLATFORMS=("amd64" "arm64" "armv7")
AVAILABLE=()

for PLATFORM in "${PLATFORMS[@]}"; do
    if docker manifest inspect $IMAGE_NAME:$PLATFORM &>/dev/null; then
        echo "  ✓ $PLATFORM found"
        AVAILABLE+=("$IMAGE_NAME:$PLATFORM")
    else
        echo "  ✗ $PLATFORM not found"
    fi
done

if [ ${#AVAILABLE[@]} -eq 0 ]; then
    echo "Error: No platform images found"
    exit 1
fi

# Create manifest
echo "Creating manifest from ${#AVAILABLE[@]} platforms..."
docker manifest create $IMAGE_NAME:$TAG "${AVAILABLE[@]}"

# Annotate platforms
for IMG in "${AVAILABLE[@]}"; do
    PLATFORM=$(echo $IMG | cut -d: -f2)
    case $PLATFORM in
        amd64)
            docker manifest annotate $IMAGE_NAME:$TAG $IMG \
                --os linux --arch amd64
            ;;
        arm64)
            docker manifest annotate $IMAGE_NAME:$TAG $IMG \
                --os linux --arch arm64 --variant v8
            ;;
        armv7)
            docker manifest annotate $IMAGE_NAME:$TAG $IMG \
                --os linux --arch arm --variant v7
            ;;
    esac
done

# Verify
echo "Verifying manifest..."
docker manifest inspect $IMAGE_NAME:$TAG

# Push
echo "Pushing manifest..."
docker manifest push $IMAGE_NAME:$TAG

echo "✓ Recovery complete!"
```

## Best Practices for Prevention

### Always Enable Experimental First
```bash
# Add to shell profile
echo 'export DOCKER_CLI_EXPERIMENTAL=enabled' >> ~/.bashrc
echo 'export DOCKER_CLI_EXPERIMENTAL=enabled' >> ~/.zshrc

# Or use config file
mkdir -p ~/.docker
cat > ~/.docker/config.json <<EOF
{
  "experimental": "enabled"
}
EOF
```

### Verify Before Pushing
```bash
# Always inspect before pushing
docker manifest inspect myapp:latest | jq

# Check all platforms present
docker manifest inspect myapp:latest | \
    jq -r '.manifests[] | .platform.architecture' | sort

# Verify annotations
docker manifest inspect myapp:latest | \
    jq '.manifests[] | {arch: .platform.architecture, variant: .platform.variant}'

# Test pull on each platform
for platform in linux/amd64 linux/arm64; do
    echo "Testing $platform"
    docker pull --platform $platform myapp:latest
done
```

### Use Consistent Naming
```bash
# Good naming convention
myapp:1.0.0-linux-amd64
myapp:1.0.0-linux-arm64
myapp:1.0.0-linux-armv7

# Avoid mixed formats
# myapp:1.0.0-amd64      # Inconsistent
# myapp:amd64-1.0.0      # Inconsistent
# myapp:1.0.0-arm64-v8   # Too specific
```

### Document Supported Platforms
```bash
# In README.md or PLATFORMS.md
cat > PLATFORMS.md <<EOF
# Supported Platforms

This image supports the following platforms:

- linux/amd64 (x86_64)
- linux/arm64 (aarch64)
- linux/arm/v7 (armv7l)
- linux/arm/v6 (armv6l)

## Verification

\`\`\`bash
docker manifest inspect myapp:latest
\`\`\`
EOF
```

### Automated Testing
```bash
# Test all platforms in CI
#!/bin/bash
PLATFORMS=("linux/amd64" "linux/arm64" "linux/arm/v7")

for PLATFORM in "${PLATFORMS[@]}"; do
    echo "Testing $PLATFORM"
    docker pull --platform $PLATFORM myapp:latest
    docker run --rm --platform $PLATFORM myapp:latest --version || exit 1
done

echo "All platforms tested successfully"
```

## See Also
- [example.md](example.md) - Manifest command examples
- [README.md](README.md) - Manifest command documentation
- [build](../build/README.md) - Building platform-specific images
- [push](../push/README.md) - Pushing images to registry

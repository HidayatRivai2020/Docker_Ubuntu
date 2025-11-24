# Troubleshooting

## Common Issues

### Build Context Too Large
**Problem**: Build takes too long or fails due to large context
```
Sending build context to Docker daemon  2.5GB
```

**Solutions**:
```bash
# Create .dockerignore file
cat > .dockerignore <<EOF
node_modules
.git
.env
*.log
dist
target
*.tmp
EOF

# Check context size before building
du -sh .

# Build from specific directory
docker build -f docker/Dockerfile -t myapp docker/

# Use smaller build context
mkdir build-context
cp -r src package.json Dockerfile build-context/
docker build -t myapp build-context/
```

### Cannot Find Dockerfile
**Problem**: Docker cannot locate the Dockerfile
```
unable to prepare context: unable to evaluate symlinks in Dockerfile path
```

**Solutions**:
```bash
# Specify Dockerfile explicitly
docker build -f Dockerfile -t myapp .

# Check if Dockerfile exists
ls -la Dockerfile

# Use absolute path
docker build -f /absolute/path/to/Dockerfile -t myapp /absolute/path/to/context

# Check current directory
pwd
ls -la
```

### Base Image Pull Failures
**Problem**: Cannot pull base image from registry
```
failed to solve with frontend dockerfile.v0: failed to create LLB definition: 
pull access denied for baseimage, repository does not exist or may require 'docker login'
```

**Solutions**:
```bash
# Login to registry
docker login

# Specify full image path
docker build --build-arg BASE_IMAGE=docker.io/library/ubuntu:22.04 -t myapp .

# Pull base image manually first
docker pull ubuntu:22.04
docker build -t myapp .

# Check registry availability
ping registry.hub.docker.com

# Use mirror or proxy
docker build --build-arg HTTP_PROXY=http://proxy:3128 -t myapp .
```

### Build Cache Issues
**Problem**: Unwanted cache usage or cache not working
```
# Changes not reflected in build
```

**Solutions**:
```bash
# Force rebuild without cache
docker build --no-cache -t myapp .

# Pull latest base image
docker build --pull -t myapp .

# Clear all build cache
docker builder prune

# Remove specific cached layers
docker builder prune --filter until=24h

# Check build cache usage
docker system df -v
```

### Out of Disk Space
**Problem**: Build fails due to insufficient disk space
```
no space left on device
```

**Solutions**:
```bash
# Check disk usage
df -h
docker system df

# Clean up unused resources
docker system prune -a

# Remove unused images
docker image prune -a

# Remove build cache
docker builder prune -a

# Check Docker root directory
docker info | grep "Docker Root Dir"
du -sh $(docker info | grep "Docker Root Dir" | cut -d: -f2)
```

### Network Issues During Build
**Problem**: Cannot reach external resources during build
```
ERROR: failed to solve: process "/bin/sh -c apt-get update" did not complete successfully
```

**Solutions**:
```bash
# Use host network
docker build --network host -t myapp .

# Add DNS servers
docker build --dns 8.8.8.8 --dns 8.8.4.4 -t myapp .

# Check network connectivity
ping google.com

# Test with simple Dockerfile
cat > test.Dockerfile <<EOF
FROM ubuntu:22.04
RUN apt-get update
EOF
docker build -f test.Dockerfile .

# Use proxy
docker build \
  --build-arg HTTP_PROXY=http://proxy:3128 \
  --build-arg HTTPS_PROXY=http://proxy:3128 \
  -t myapp .
```

### Multi-Stage Build Failures
**Problem**: Errors in multi-stage builds
```
failed to solve: stage builder: invalid from flag value builder: pull access denied
```

**Solutions**:
```bash
# Build specific stage for debugging
docker build --target builder -t myapp:builder .

# Check stage names
grep "^FROM" Dockerfile

# Verify COPY --from references
grep "COPY --from" Dockerfile

# Build stages separately
docker build --target stage1 -t myapp:stage1 .
docker build --target stage2 -t myapp:stage2 .

# Check for typos in stage names
cat Dockerfile | grep -E "FROM|COPY --from"
```

### Build Argument Issues
**Problem**: Build arguments not being used correctly
```
ARG variable not available in final image
```

**Solutions**:
```bash
# Ensure ARG is defined in Dockerfile
cat Dockerfile | grep ARG

# Pass build argument correctly
docker build --build-arg MY_VAR=value -t myapp .

# Check ARG scope (before/after FROM)
# ARG before FROM is only available for FROM instruction
# ARG after FROM is available for that stage

# Debug build arguments
docker build --build-arg MY_VAR=value --progress plain -t myapp .

# Print ARG values in Dockerfile
# RUN echo "MY_VAR = $MY_VAR"
```

### Permission Errors
**Problem**: Permission denied errors during build
```
permission denied while trying to connect to Docker daemon
```

**Solutions**:
```bash
# Use sudo
sudo docker build -t myapp .

# Add user to docker group
sudo usermod -aG docker $USER
# Log out and log back in

# Check Docker socket permissions
ls -la /var/run/docker.sock

# Fix Docker socket permissions (temporary)
sudo chmod 666 /var/run/docker.sock

# Restart Docker service
sudo systemctl restart docker
```

## Debugging Commands

### Build Inspection
```bash
# Build with verbose output
docker build --progress plain -t myapp .

# Build without output buffering
docker build --progress plain --no-cache -t myapp . 2>&1 | tee build.log

# Check Dockerfile syntax
docker run --rm -i hadolint/hadolint < Dockerfile

# Analyze Dockerfile
cat Dockerfile

# Test Dockerfile instructions individually
docker run -it ubuntu:22.04 bash
# Then run commands manually
```

### Context Analysis
```bash
# Check build context size
du -sh .

# List files in build context
find . -type f | head -20

# Check .dockerignore
cat .dockerignore

# Verify files being sent to daemon
docker build --progress plain . 2>&1 | grep "transferring context"

# Create minimal test context
mkdir test-context
cd test-context
echo "FROM ubuntu:22.04" > Dockerfile
docker build -t test .
```

### Cache Debugging
```bash
# Show cache usage
docker system df -v

# List build cache
docker builder ls

# Inspect build cache
docker buildx du

# Clear specific builder cache
docker builder prune --filter type=exec.cachemount

# Compare builds with/without cache
time docker build -t myapp:cached .
time docker build --no-cache -t myapp:nocache .
```

### Layer Analysis
```bash
# View image layers
docker history myapp:latest

# Detailed layer information
docker history --no-trunc myapp:latest

# Show layer sizes
docker history --human myapp:latest

# Inspect specific layer
docker inspect myapp:latest | jq '.[0].RootFS.Layers'

# Export and examine filesystem
docker save myapp:latest -o myapp.tar
tar -tvf myapp.tar
```

### Build Process Monitoring
```bash
# Monitor build in real-time
watch -n 1 'docker ps -a | grep build'

# Monitor system resources during build
top
htop
df -h

# Check Docker daemon logs
sudo journalctl -u docker.service -f

# Monitor build progress
docker build --progress plain -t myapp . 2>&1 | tee /dev/tty | grep -E "Step|RUN|COPY"
```

## Error Codes and Meanings

### Standard Exit Codes
- **0** - Successful build
- **1** - General build error
- **125** - Docker daemon error
- **127** - Command not found in build step

### Common Build Errors
- **"no such file or directory"** - File not in build context
- **"pull access denied"** - Cannot access base image
- **"no space left on device"** - Insufficient disk space
- **"context canceled"** - Build interrupted or timed out
- **"unknown flag"** - Invalid build option
- **"failed to compute cache key"** - Cache corruption

### Exit Code Checking
```bash
# Check build exit status
docker build -t myapp .
echo "Exit code: $?"

# Build with error handling
if docker build -t myapp .; then
    echo "Build successful"
else
    echo "Build failed with exit code $?"
    exit 1
fi

# Capture build output and errors
build_output=$(docker build -t myapp . 2>&1)
if [ $? -eq 0 ]; then
    echo "Build succeeded"
else
    echo "Build failed:"
    echo "$build_output"
fi
```

## Recovery Procedures

### Build Failure Recovery
```bash
#!/bin/bash
# Comprehensive build recovery script
set -e

IMAGE_NAME="myapp"

echo "=== Build Recovery for $IMAGE_NAME ==="

# Step 1: Clean environment
echo "Step 1: Cleaning environment..."
docker system prune -f
docker builder prune -f

# Step 2: Check disk space
echo "Step 2: Checking disk space..."
available_space=$(df /var/lib/docker | awk 'NR==2 {print $4}')
if [ $available_space -lt 5000000 ]; then
    echo "Warning: Low disk space ($(($available_space/1024))MB available)"
    echo "Cleaning up more aggressively..."
    docker system prune -a -f
fi

# Step 3: Verify Dockerfile
echo "Step 3: Verifying Dockerfile..."
if [ ! -f Dockerfile ]; then
    echo "Error: Dockerfile not found"
    exit 1
fi

# Step 4: Check build context
echo "Step 4: Checking build context..."
context_size=$(du -sm . | cut -f1)
echo "Build context size: ${context_size}MB"
if [ $context_size -gt 500 ]; then
    echo "Warning: Large build context. Consider using .dockerignore"
fi

# Step 5: Pull base image
echo "Step 5: Pulling base image..."
base_image=$(grep "^FROM" Dockerfile | head -1 | awk '{print $2}')
if docker pull "$base_image"; then
    echo "Base image pulled successfully"
else
    echo "Warning: Could not pull base image"
fi

# Step 6: Attempt build
echo "Step 6: Attempting build..."
if docker build --progress plain -t "$IMAGE_NAME" . 2>&1 | tee build.log; then
    echo "Build successful!"
else
    echo "Build failed. Check build.log for details"
    echo "Last 20 lines of build log:"
    tail -20 build.log
    exit 1
fi

echo "Recovery completed successfully"
```

### Cache Reset Procedure
```bash
#!/bin/bash
# Reset build cache and rebuild
echo "=== Build Cache Reset ==="

# Show current cache usage
echo "Current cache usage:"
docker system df

echo
read -p "Do you want to clear ALL build cache? (yes/no): " confirm

if [ "$confirm" = "yes" ]; then
    echo "Clearing build cache..."
    docker builder prune -a -f
    
    echo "Clearing system cache..."
    docker system prune -f
    
    echo "Cache cleared. New usage:"
    docker system df
    
    echo
    echo "Rebuilding without cache..."
    docker build --no-cache --pull -t myapp .
else
    echo "Cache reset cancelled"
fi
```

### Dockerfile Validation
```bash
#!/bin/bash
# Validate Dockerfile before building
DOCKERFILE="${1:-Dockerfile}"

echo "=== Dockerfile Validation ==="
echo "Validating: $DOCKERFILE"

if [ ! -f "$DOCKERFILE" ]; then
    echo "Error: $DOCKERFILE not found"
    exit 1
fi

# Check basic syntax
echo "\nChecking basic syntax..."
if grep -q "^FROM" "$DOCKERFILE"; then
    echo "✓ FROM instruction found"
else
    echo "✗ No FROM instruction found"
    exit 1
fi

# Check for common issues
echo "\nChecking for common issues..."

# Multiple FROM statements (multi-stage)
from_count=$(grep -c "^FROM" "$DOCKERFILE")
echo "FROM statements: $from_count"
if [ $from_count -gt 1 ]; then
    echo "  Multi-stage build detected"
fi

# Check for apt-get without cleanup
if grep -q "apt-get" "$DOCKERFILE"; then
    if ! grep -q "rm -rf /var/lib/apt/lists" "$DOCKERFILE"; then
        echo "⚠️  Warning: apt-get used without cleanup"
    fi
fi

# Check for EXPOSE
if grep -q "^EXPOSE" "$DOCKERFILE"; then
    echo "✓ EXPOSE instruction found"
    grep "^EXPOSE" "$DOCKERFILE"
fi

# Check for HEALTHCHECK
if grep -q "^HEALTHCHECK" "$DOCKERFILE"; then
    echo "✓ HEALTHCHECK instruction found"
else
    echo "⚠️  Warning: No HEALTHCHECK instruction"
fi

# Use hadolint if available
if command -v hadolint &> /dev/null; then
    echo "\nRunning hadolint..."
    hadolint "$DOCKERFILE"
else
    echo "\nNote: Install hadolint for detailed linting"
    echo "  https://github.com/hadolint/hadolint"
fi

echo "\nValidation complete"
```

### Emergency Build Procedure
```bash
#!/bin/bash
# Emergency minimal build when normal build fails
set -e

echo "=== Emergency Build Procedure ==="

# Create minimal Dockerfile
cat > Dockerfile.minimal <<EOF
FROM alpine:latest
RUN apk add --no-cache bash
COPY . /app
WORKDIR /app
CMD ["sh"]
EOF

echo "Created minimal Dockerfile"

# Build with minimal Dockerfile
echo "Building minimal image..."
if docker build -f Dockerfile.minimal -t myapp:minimal .; then
    echo "Minimal build successful"
    
    # Test the image
    echo "Testing minimal image..."
    docker run --rm myapp:minimal ls -la /app
    
    echo "Minimal image is working"
    echo "You can now incrementally add features to Dockerfile.minimal"
else
    echo "Even minimal build failed"
    echo "Checking Docker daemon..."
    sudo systemctl status docker
    
    echo "Checking disk space..."
    df -h
    
    echo "Checking Docker info..."
    docker info
fi
```
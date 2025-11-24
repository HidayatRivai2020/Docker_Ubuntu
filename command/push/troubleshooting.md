# Docker Push Troubleshooting

## Common Issues

### Authentication Required
**Problem**: Push fails with "authentication required" error
```bash
# Login to registry first
docker login

# For private registry
docker login registry.example.com

# Verify login status
cat ~/.docker/config.json | jq '.auths'

# Login with credentials
echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

# Check if logged into correct registry
docker login docker.io
```

### Image Not Found Locally
**Problem**: Cannot push image because it doesn't exist locally
```bash
# List local images
docker images

# Check if image exists with correct tag
docker images myapp:latest

# Build image first
docker build -t myapp:latest .

# Tag existing image
docker tag <image_id> username/myapp:latest

# Verify image ID
docker inspect myapp:latest --format='{{.Id}}'
```

### Access Denied / Permission Denied
**Problem**: Don't have permission to push to repository
```bash
# Verify repository ownership
# For Docker Hub: username/repository format required

# Check if logged in as correct user
docker login

# Verify you have write access to repository
# Create repository on registry web interface first

# Use correct repository name
docker tag myapp:latest username/myapp:latest
docker push username/myapp:latest

# For organizations, check team permissions
```

### Network Timeout
**Problem**: Push operation times out or fails due to network issues
```bash
# Check network connectivity
ping registry.hub.docker.com
ping registry.example.com

# Test registry connection
curl -I https://registry.hub.docker.com/v2/

# Check proxy settings
echo $HTTP_PROXY
echo $HTTPS_PROXY

# Configure Docker proxy if needed
# Edit /etc/systemd/system/docker.service.d/http-proxy.conf

# Retry push
docker push username/myapp:latest

# Use different network
# Check firewall rules
```

### Layer Already Exists Error
**Problem**: Push fails with "layer already exists" but doesn't complete
```bash
# This usually means layers are already on registry
# Pull the image to verify
docker pull username/myapp:latest

# Force push by retagging
docker tag myapp:latest username/myapp:latest
docker push username/myapp:latest

# Check registry status
curl https://status.docker.com/

# Clear local cache and retry
docker system prune -f
docker push username/myapp:latest
```

### Insufficient Storage / Disk Space
**Problem**: Not enough space to push image
```bash
# Check disk usage
df -h
docker system df

# Clean up unused images
docker system prune -a

# Remove dangling images
docker image prune

# Check registry quota
# For Docker Hub: check account limits

# Optimize image size
docker build --squash -t myapp:latest .
```

### Registry Not Responding
**Problem**: Registry server is down or unreachable
```bash
# Check registry status
curl -I https://registry.example.com/v2/

# Verify DNS resolution
nslookup registry.example.com

# Check registry health
curl https://registry.example.com/v2/_catalog

# Try alternative registry
docker tag myapp:latest ghcr.io/username/myapp:latest
docker push ghcr.io/username/myapp:latest

# Check service status pages
# Docker Hub: https://status.docker.com/
```

## Debugging Commands

### Verify Image and Tags
```bash
# List all images
docker images

# Check specific image
docker images username/myapp

# Inspect image details
docker inspect username/myapp:latest

# Check image size
docker images username/myapp:latest --format "{{.Size}}"

# Verify image ID
docker images --no-trunc username/myapp:latest
```

### Test Registry Connectivity
```bash
# Test Docker Hub
curl -v https://registry.hub.docker.com/v2/

# Test private registry
curl -v https://registry.example.com/v2/

# Check authentication
curl -u username:password https://registry.example.com/v2/_catalog

# Test with docker
docker pull alpine
docker push username/alpine:test
```

### Monitor Push Progress
```bash
# Push with verbose output
docker --debug push username/myapp:latest

# Monitor layer upload
docker push username/myapp:latest 2>&1 | tee push.log

# Check push status
docker push username/myapp:latest | grep -E "Pushed|Layer"

# Time the push
time docker push username/myapp:latest
```

### Inspect Credentials
```bash
# Check login status
cat ~/.docker/config.json

# List authenticated registries
cat ~/.docker/config.json | jq '.auths | keys'

# Verify credential helper
docker-credential-<helper> list

# Test authentication
docker login --username test --password test 2>&1
```

### Network Diagnostics
```bash
# Test DNS
dig registry.hub.docker.com

# Check routing
traceroute registry.hub.docker.com

# Test port connectivity
nc -zv registry.hub.docker.com 443

# Check proxy
curl -v --proxy $HTTP_PROXY https://registry.hub.docker.com/v2/
```

## Exit Codes and Meanings

### Standard Exit Codes
- **0** - Successful push
- **1** - General error (authentication, network, permission)
- **125** - Docker daemon error
- **126** - Command not executable
- **127** - Command not found

### Common Error Messages
- **"denied: requested access to the resource is denied"** - Authentication or permission issue
- **"unauthorized: authentication required"** - Need to login first
- **"name unknown"** - Repository doesn't exist
- **"tag does not exist"** - Image tag not found locally
- **"manifest invalid"** - Image manifest is corrupted

### Checking Exit Codes
```bash
# Push and check result
docker push username/myapp:latest
echo $?

# Capture error
ERROR=$(docker push username/myapp:latest 2>&1)
if [ $? -ne 0 ]; then
    echo "Push failed: $ERROR"
fi
```

## Recovery Procedures

### Reset and Retry Push
```bash
# Step 1: Verify image exists
docker images myapp:latest

# Step 2: Re-tag image
docker tag myapp:latest username/myapp:latest

# Step 3: Logout and login
docker logout
docker login

# Step 4: Retry push
docker push username/myapp:latest

# Step 5: Verify success
docker pull username/myapp:latest
```

### Fix Authentication Issues
```bash
# Step 1: Logout
docker logout registry.example.com

# Step 2: Clear credentials
rm ~/.docker/config.json

# Step 3: Login again
docker login registry.example.com

# Step 4: Verify authentication
docker info | grep Username

# Step 5: Push image
docker push registry.example.com/myapp:latest
```

### Handle Network Issues
```bash
#!/bin/bash
# network-push.sh - Push with network retry

IMAGE=$1
MAX_RETRIES=5
RETRY_DELAY=10

for i in $(seq 1 $MAX_RETRIES); do
    echo "Push attempt $i..."
    
    docker push $IMAGE
    
    if [ $? -eq 0 ]; then
        echo "✓ Push successful"
        exit 0
    fi
    
    echo "Push failed, checking network..."
    ping -c 1 registry.hub.docker.com > /dev/null 2>&1
    
    if [ $? -ne 0 ]; then
        echo "Network issue detected"
    fi
    
    if [ $i -lt $MAX_RETRIES ]; then
        echo "Retrying in $RETRY_DELAY seconds..."
        sleep $RETRY_DELAY
    fi
done

echo "✗ Push failed after $MAX_RETRIES attempts"
exit 1
```

### Automated Debugging Script
```bash
#!/bin/bash
# docker-push-debug.sh - Comprehensive push diagnostics

IMAGE=$1

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image:tag>"
    exit 1
fi

echo "=== Docker Push Debug Report ==="
echo "Image: $IMAGE"
echo "Timestamp: $(date)"
echo

echo "=== Image Verification ==="
if docker images | grep -q "$(echo $IMAGE | cut -d: -f1)"; then
    echo "✓ Image exists locally"
    docker images | grep "$(echo $IMAGE | cut -d: -f1)"
else
    echo "✗ Image not found locally"
    exit 1
fi
echo

echo "=== Authentication Status ==="
if [ -f ~/.docker/config.json ]; then
    echo "Authenticated registries:"
    cat ~/.docker/config.json | jq '.auths | keys'
else
    echo "✗ No authentication found"
fi
echo

echo "=== Registry Connectivity ==="
REGISTRY=$(echo $IMAGE | cut -d/ -f1)
if [[ ! "$REGISTRY" =~ \. ]]; then
    REGISTRY="registry.hub.docker.com"
fi

echo "Testing $REGISTRY..."
if curl -f -s -o /dev/null https://$REGISTRY/v2/; then
    echo "✓ Registry is accessible"
else
    echo "✗ Cannot reach registry"
fi
echo

echo "=== Network Diagnostics ==="
echo "DNS resolution:"
nslookup $REGISTRY | grep Address || echo "DNS lookup failed"
echo

echo "Port connectivity:"
nc -zv $REGISTRY 443 2>&1 || echo "Port 443 not reachable"
echo

echo "=== Disk Space ==="
df -h | grep -E "Filesystem|/$"
echo
docker system df
echo

echo "=== Attempting Push ==="
docker push $IMAGE
RESULT=$?

if [ $RESULT -eq 0 ]; then
    echo "✓ Push successful"
else
    echo "✗ Push failed with exit code $RESULT"
fi
```

### Verify Push Success
```bash
#!/bin/bash
# verify-push.sh - Verify image was pushed successfully

IMAGE=$1

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image:tag>"
    exit 1
fi

echo "Verifying push of $IMAGE..."

# Get local digest
LOCAL_DIGEST=$(docker inspect $IMAGE --format='{{index .RepoDigests 0}}')
echo "Local digest: $LOCAL_DIGEST"

# Pull from registry
docker pull $IMAGE > /dev/null 2>&1

# Get registry digest
REMOTE_DIGEST=$(docker inspect $IMAGE --format='{{index .RepoDigests 0}}')
echo "Remote digest: $REMOTE_DIGEST"

if [ "$LOCAL_DIGEST" == "$REMOTE_DIGEST" ]; then
    echo "✓ Push verified - digests match"
    exit 0
else
    echo "✗ Push verification failed - digests don't match"
    exit 1
fi
```

### Clean and Retry
```bash
#!/bin/bash
# clean-and-push.sh - Clean up and retry push

IMAGE=$1

echo "Cleaning Docker system..."
docker system prune -f

echo "Removing old image if exists..."
docker rmi $IMAGE 2>/dev/null

echo "Rebuilding image..."
docker build -t $IMAGE .

echo "Logging in to registry..."
docker login

echo "Pushing image..."
docker push $IMAGE

if [ $? -eq 0 ]; then
    echo "✓ Push successful"
    
    echo "Verifying..."
    docker pull $IMAGE
    echo "✓ Verified"
else
    echo "✗ Push failed"
    exit 1
fi
```

## Best Practices for Troubleshooting

### Before Pushing
```bash
# 1. Verify image exists
docker images | grep myapp

# 2. Check image size (large images take longer)
docker images myapp:latest --format "{{.Size}}"

# 3. Ensure logged in
docker login

# 4. Test registry connectivity
curl -I https://registry.hub.docker.com/v2/

# 5. Check disk space
df -h
```

### When Push Fails
```bash
# 1. Check error message
docker push username/myapp:latest 2>&1 | tee error.log

# 2. Verify authentication
cat ~/.docker/config.json

# 3. Test network
ping registry.hub.docker.com

# 4. Check Docker daemon
systemctl status docker

# 5. Review Docker logs
sudo journalctl -u docker.service -n 50
```

### For Large Images
```bash
# 1. Optimize image size
docker build --squash -t myapp .

# 2. Use .dockerignore
echo "node_modules" >> .dockerignore

# 3. Multi-stage builds
# Use multi-stage Dockerfile

# 4. Push during off-peak hours
# Schedule pushes

# 5. Use faster network
# Push from server with better bandwidth
```

### Regular Maintenance
```bash
# Clean up old images
docker image prune -a

# Check registry quotas
# Monitor usage on registry dashboard

# Update Docker
sudo apt-get update && sudo apt-get upgrade docker-ce

# Test push regularly
# Include in CI/CD pipeline
```

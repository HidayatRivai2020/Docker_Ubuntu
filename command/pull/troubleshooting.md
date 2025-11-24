# Troubleshooting

## Common Issues

### Network Connection Problems
**Problem**: Cannot connect to Docker registry
```
Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: TLS handshake timeout
```

**Solutions**:
```bash
# Check network connectivity
ping registry-1.docker.io
curl -I https://registry-1.docker.io/v2/

# Check DNS resolution
nslookup registry-1.docker.io

# Try with different DNS servers
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf

# Configure proxy if needed
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080

# Restart Docker daemon
sudo systemctl restart docker
```

### Authentication Failures
**Problem**: Access denied or unauthorized
```
Error response from daemon: pull access denied for private/repo
```

**Solutions**:
```bash
# Login to registry
docker login
# Or specify registry
docker login myregistry.com

# Check login status
docker info | grep Username

# Login with token (for CI/CD)
echo "$REGISTRY_TOKEN" | docker login --username "$REGISTRY_USER" --password-stdin

# Check stored credentials
cat ~/.docker/config.json

# Clear and re-login
docker logout
docker login
```

### Image Not Found
**Problem**: Repository or tag does not exist
```
Error response from daemon: pull access denied for nginx, repository does not exist
```

**Solutions**:
```bash
# Verify image name and tag
docker search nginx

# Check available tags
curl -s "https://registry.hub.docker.com/v2/repositories/nginx/tags/" | jq '.results[].name'

# Use correct registry URL
docker pull registry.hub.docker.com/nginx:latest

# Check if image exists
curl -s "https://registry.hub.docker.com/v2/repositories/nginx/"
```

### Rate Limiting Issues
**Problem**: Docker Hub rate limits exceeded
```
toomanyrequests: You have reached your pull rate limit
```

**Solutions**:
```bash
# Login to increase rate limit
docker login

# Use alternative registry
docker pull quay.io/nginx/nginx:latest

# Implement retry logic
for i in {1..3}; do
    if docker pull nginx:latest; then
        break
    else
        echo "Attempt $i failed, retrying in 60 seconds..."
        sleep 60
    fi
done

# Use Docker Hub alternatives
docker pull gcr.io/distroless/nginx
```

### Disk Space Issues
**Problem**: Insufficient disk space during pull
```
no space left on device
```

**Solutions**:
```bash
# Check available space
df -h
docker system df

# Clean up Docker resources
docker system prune -f
docker image prune -f

# Remove unused volumes
docker volume prune -f

# Check Docker root directory
docker info | grep "Docker Root Dir"

# Move Docker to larger partition (if needed)
sudo systemctl stop docker
sudo mv /var/lib/docker /new/location/
sudo ln -s /new/location/docker /var/lib/docker
sudo systemctl start docker
```

### SSL/TLS Certificate Issues
**Problem**: Certificate verification failures
```
Error response from daemon: Get https://registry.com: x509: certificate signed by unknown authority
```

**Solutions**:
```bash
# Add registry to insecure registries (not recommended for production)
sudo nano /etc/docker/daemon.json
# Add: {"insecure-registries": ["registry.com:5000"]}
sudo systemctl restart docker

# Install certificates
sudo mkdir -p /etc/docker/certs.d/registry.com:5000
sudo cp ca.crt /etc/docker/certs.d/registry.com:5000/

# Update system certificates
sudo cp registry.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# Skip TLS verification (temporary)
docker pull --disable-content-trust registry.com/image:tag
```

### Platform Compatibility Issues
**Problem**: Image not available for current platform
```
no matching manifest for linux/arm64/v8 in the manifest list entries
```

**Solutions**:
```bash
# Check available platforms
docker manifest inspect nginx

# Pull for specific platform
docker pull --platform linux/amd64 nginx

# Use multi-arch image
docker pull nginx:latest  # Usually supports multiple platforms

# Find ARM-compatible alternatives
docker search nginx | grep arm
docker pull arm64v8/nginx
```

## Debugging Commands

### Network Diagnostics
```bash
# Test registry connectivity
curl -v https://registry-1.docker.io/v2/

# Check Docker daemon configuration
docker info

# Test DNS resolution
nslookup registry-1.docker.io
dig registry-1.docker.io

# Check network routes
traceroute registry-1.docker.io

# Test with different protocols
wget --spider https://registry-1.docker.io/v2/
```

### Registry Investigation
```bash
# Check registry API version
curl -s https://registry-1.docker.io/v2/ | jq .

# List repository tags
curl -s "https://registry.hub.docker.com/v2/repositories/nginx/tags/" | jq '.results[].name'

# Get image manifest
docker manifest inspect nginx:latest

# Check image layers
curl -s -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
  "https://registry-1.docker.io/v2/library/nginx/manifests/latest"
```

### Authentication Debugging
```bash
# Check current authentication
docker info | grep -i username

# Inspect stored credentials
cat ~/.docker/config.json | jq .

# Test authentication manually
curl -u username:password https://registry.com/v2/_catalog

# Check token validity
echo "$DOCKER_TOKEN" | base64 -d | jq .
```

### System Resource Analysis
```bash
# Check disk usage by Docker
docker system df -v

# Monitor disk usage during pull
watch -n 1 'df -h; echo; docker system df'

# Check memory usage
free -h
docker stats --no-stream

# Monitor network traffic
sudo netstat -i
sudo iftop -i eth0
```

### Docker Daemon Debugging
```bash
# Check Docker daemon logs
sudo journalctl -u docker.service --since "10 minutes ago"

# Run Docker in debug mode
sudo dockerd --debug --log-level=debug

# Check daemon configuration
cat /etc/docker/daemon.json

# Verify daemon status
sudo systemctl status docker
```

## Error Codes and Meanings

### HTTP Status Codes
- **401** - Unauthorized (authentication required)
- **403** - Forbidden (insufficient permissions)
- **404** - Not found (repository or tag doesn't exist)
- **429** - Too many requests (rate limited)
- **500** - Internal server error (registry issue)
- **502/503** - Bad gateway/Service unavailable (registry down)

### Docker-Specific Errors
- **Repository does not exist** - Incorrect image name or private repo
- **Tag does not exist** - Invalid tag or version
- **TLS handshake timeout** - Network connectivity issues
- **Certificate signed by unknown authority** - SSL/TLS issues
- **No space left on device** - Insufficient disk space
- **Too many requests** - Docker Hub rate limiting

### Checking Pull Status
```bash
# Check if pull was successful
if docker pull nginx:latest; then
    echo "Pull successful"
else
    echo "Pull failed with exit code: $?"
fi

# Monitor pull progress
docker pull nginx:latest 2>&1 | tee pull.log

# Check pulled image
docker images nginx:latest
docker inspect nginx:latest
```

## Recovery Procedures

### Network Recovery
```bash
# Step 1: Diagnose network issues
ping -c 4 8.8.8.8
ping -c 4 registry-1.docker.io

# Step 2: Reset DNS configuration
sudo systemctl restart systemd-resolved

# Step 3: Restart networking
sudo systemctl restart networking

# Step 4: Restart Docker daemon
sudo systemctl restart docker

# Step 5: Test connectivity
docker pull hello-world
```

### Authentication Recovery
```bash
#!/bin/bash
# Comprehensive authentication reset
echo "Resetting Docker authentication..."

# Step 1: Logout from all registries
docker logout
for registry in docker.io gcr.io quay.io; do
    docker logout $registry 2>/dev/null || true
done

# Step 2: Clear credential storage
rm -f ~/.docker/config.json

# Step 3: Re-login
read -p "Enter Docker Hub username: " username
docker login -u "$username"

# Step 4: Test authentication
if docker pull hello-world; then
    echo "✓ Authentication restored"
else
    echo "✗ Authentication still failing"
fi
```

### Registry Failover
```bash
#!/bin/bash
# Automatic registry failover for critical images
pull_with_failover() {
    local image="$1"
    local base_name="$(echo $image | cut -d'/' -f2- | cut -d':' -f1)"
    local tag="$(echo $image | cut -d':' -f2)"
    
    # Define registry alternatives
    local registries=(
        "docker.io"
        "quay.io"
        "gcr.io/google-containers"
    )
    
    for registry in "${registries[@]}"; do
        local full_image="$registry/$base_name:$tag"
        echo "Trying to pull from $registry..."
        
        if docker pull "$full_image" 2>/dev/null; then
            echo "✓ Successfully pulled from $registry"
            # Retag to original name if needed
            if [ "$full_image" != "$image" ]; then
                docker tag "$full_image" "$image"
            fi
            return 0
        else
            echo "✗ Failed to pull from $registry"
        fi
    done
    
    echo "✗ All registries failed for $image"
    return 1
}

# Usage
pull_with_failover "nginx:alpine"
```

### Emergency Image Recovery
```bash
#!/bin/bash
# Emergency recovery when normal pulls fail
echo "Starting emergency image recovery..."

# Step 1: Clean up corrupted pulls
docker system prune -f

# Step 2: Reset Docker daemon
sudo systemctl stop docker
sudo rm -rf /var/lib/docker/tmp/*
sudo systemctl start docker

# Step 3: Use alternative methods
echo "Trying alternative image sources..."

# Try different base images
alternatives=(
    "alpine:latest"
    "ubuntu:20.04"
    "busybox:latest"
)

for image in "${alternatives[@]}"; do
    echo "Trying $image..."
    if timeout 300 docker pull "$image"; then
        echo "✓ Successfully pulled $image"
        break
    else
        echo "✗ Failed to pull $image"
    fi
done

# Step 4: Verify Docker functionality
docker run --rm alpine echo "Docker is working"
```

### Automated Recovery Script
```bash
#!/bin/bash
# Comprehensive Docker pull recovery
recover_docker_pull() {
    local image="$1"
    local max_attempts=3
    local wait_time=60
    
    echo "Starting recovery for $image..."
    
    for attempt in $(seq 1 $max_attempts); do
        echo "Recovery attempt $attempt/$max_attempts"
        
        case $attempt in
            1)
                echo "  Trying normal pull..."
                if docker pull "$image"; then
                    return 0
                fi
                ;;
            2)
                echo "  Clearing Docker cache and retrying..."
                docker system prune -f
                sudo systemctl restart docker
                sleep 30
                if docker pull "$image"; then
                    return 0
                fi
                ;;
            3)
                echo "  Trying with authentication reset..."
                docker logout
                echo "Please re-authenticate:"
                docker login
                if docker pull "$image"; then
                    return 0
                fi
                ;;
        esac
        
        if [ $attempt -lt $max_attempts ]; then
            echo "  Waiting $wait_time seconds before next attempt..."
            sleep $wait_time
        fi
    done
    
    echo "✗ All recovery attempts failed for $image"
    return 1
}

# Usage
# recover_docker_pull "nginx:latest"
```
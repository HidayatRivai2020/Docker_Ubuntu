# Troubleshooting

## Common Issues

### Image in Use by Container
**Problem**: Cannot remove image because it's being used by containers
```
Error response from daemon: conflict: unable to remove repository reference
```

**Solutions**:
```bash
# Check which containers are using the image
docker ps -a --filter "ancestor=<image_name>"

# Stop and remove containers first
docker stop <container_name>
docker rm <container_name>

# Then remove the image
docker rmi <image_name>

# Or force remove (not recommended)
docker rmi -f <image_name>
```

### Image Not Found
**Problem**: Specified image doesn't exist or name is incorrect
```
Error: No such image: <image_name>
```

**Solutions**:
```bash
# List all available images
docker images

# Search for images with partial name
docker images | grep <partial_name>

# Use exact repository:tag format
docker images --format "{{.Repository}}:{{.Tag}}"

# Remove by image ID instead
docker rmi <image_id>
```

### Permission Denied
**Problem**: Insufficient permissions to remove images
```
Got permission denied while trying to connect to Docker daemon
```

**Solutions**:
```bash
# Use sudo (temporary fix)
sudo docker rmi <image_name>

# Add user to docker group (permanent fix)
sudo usermod -aG docker $USER
# Log out and log back in

# Check docker group membership
groups $USER | grep docker

# Verify Docker daemon is running
sudo systemctl status docker
```

### Image Has Dependent Child Images
**Problem**: Cannot remove parent image that has child images
```
Error response from daemon: conflict: unable to delete <image_id> (cannot be forced)
```

**Solutions**:
```bash
# Check image dependencies
docker image inspect <image_name> --format='{{.RepoTags}}'

# Remove child images first
docker images --filter "since=<parent_image>" -q | xargs docker rmi

# Or remove all related images
docker rmi -f $(docker images -q --filter "reference=<repository>*")

# Use --force to remove parent (may break child images)
docker rmi -f <parent_image>
```

### Cannot Remove Running Container Image
**Problem**: Image is being used by running containers
```
Error: conflict: unable to remove repository reference (must force)
```

**Solutions**:
```bash
# Find running containers using the image
docker ps --filter "ancestor=<image_name>"

# Stop running containers
docker stop $(docker ps -q --filter "ancestor=<image_name>")

# Remove stopped containers
docker rm $(docker ps -aq --filter "ancestor=<image_name>")

# Now remove the image
docker rmi <image_name>
```

### Disk Space Issues
**Problem**: Operation fails due to insufficient disk space
```
no space left on device
```

**Solutions**:
```bash
# Check available disk space
df -h
docker system df

# Clean up other Docker resources first
docker system prune -f

# Remove dangling images
docker image prune -f

# Clean up unused volumes
docker volume prune -f

# Then try removing the image again
docker rmi <image_name>
```

### Layer Sharing Conflicts
**Problem**: Cannot remove image layers shared with other images
```
Error: unable to delete layer (referenced by other images)
```

**Solutions**:
```bash
# Check which images share layers
docker history <image_name>

# Find images with common layers
docker inspect $(docker images -q) --format '{{.Id}} {{.RepoTags}}' | sort

# Remove specific tag instead of entire image
docker rmi <repository>:<specific_tag>

# Force removal of shared layers (use cautiously)
docker rmi -f <image_name>
```

## Debugging Commands

### Image Analysis
```bash
# Detailed image information
docker inspect <image_name>

# Image history and layers
docker history <image_name>

# Check image size breakdown
docker history <image_name> --format "table {{.CreatedBy}}\t{{.Size}}"

# Image metadata
docker inspect <image_name> --format='{{json .Config}}' | jq .
```

### Container Dependencies
```bash
# Find all containers using specific image
docker ps -a --filter "ancestor=<image_name>" --format "table {{.Names}}\t{{.Status}}\t{{.CreatedAt}}"

# Check container image references
docker inspect <container_name> --format='{{.Config.Image}}'

# List all container-image relationships
docker ps -a --format "{{.Names}}\t{{.Image}}" | sort -k2

# Find orphaned containers
docker ps -a --filter "status=exited" --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
```

### System Resource Check
```bash
# Docker system overview
docker system df -v

# Image storage details
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedAt}}"

# Check Docker daemon logs
sudo journalctl -u docker.service --since "10 minutes ago"

# Monitor disk usage in real-time
watch -n 1 'docker system df'
```

### Layer Investigation
```bash
# Examine image layers
docker inspect <image_name> --format='{{.RootFS.Layers}}'

# Compare layer usage between images
for image in $(docker images -q); do
    echo "=== Image: $image ==="
    docker inspect $image --format='{{.RepoTags}} {{.RootFS.Layers}}'
done

# Find images sharing specific layer
layer_id="<layer_sha>"
for image in $(docker images -q); do
    if docker inspect $image --format='{{.RootFS.Layers}}' | grep -q $layer_id; then
        docker inspect $image --format='{{.RepoTags}}'
    fi
done
```

### Network and Registry Issues
```bash
# Check image registry information
docker inspect <image_name> --format='{{.RepoDigests}}'

# Verify image authenticity
docker inspect <image_name> --format='{{.Config.Labels}}'

# Check for corrupted image metadata
docker inspect <image_name> --format='{{.GraphDriver}}'

# Validate image integrity
docker image inspect <image_name> --format='{{.Architecture}} {{.Os}}'
```

## Error Codes and Meanings

### Standard Exit Codes
- **0** - Successful removal
- **1** - General error
- **125** - Docker daemon error
- **404** - Image not found
- **409** - Conflict (image in use)

### Docker-Specific Errors
- **Image in use** - Containers reference the image
- **Has dependent child images** - Other images built from this image
- **Layer in use** - Shared layers prevent removal
- **Permission denied** - Insufficient Docker daemon access
- **Cannot be forced** - System-level removal restriction

### Checking Removal Status
```bash
# Verify image was removed
docker images | grep <image_name>
echo $?  # 1 = not found (success), 0 = still exists

# Check removal operation success
if docker rmi <image_name> 2>/dev/null; then
    echo "Image removed successfully"
else
    echo "Failed to remove image"
fi

# Get detailed error information
docker rmi <image_name> 2>&1 | tee removal.log
```

## Recovery Procedures

### Force Remove Stuck Images
```bash
# Step 1: Stop all containers using the image
docker stop $(docker ps -q --filter "ancestor=<image_name>")

# Step 2: Remove all containers using the image
docker rm $(docker ps -aq --filter "ancestor=<image_name>")

# Step 3: Try normal removal
docker rmi <image_name>

# Step 4: If still fails, force removal
docker rmi -f <image_name>

# Step 5: Clean up dangling layers
docker image prune -f
```

### Resolve Layer Conflicts
```bash
#!/bin/bash
# Script to resolve layer sharing conflicts
image_to_remove="$1"

if [ -z "$image_to_remove" ]; then
    echo "Usage: $0 <image_name>"
    exit 1
fi

echo "Analyzing image dependencies for: $image_to_remove"

# Find all images sharing layers
echo "Finding images with shared layers..."
layers=$(docker inspect "$image_to_remove" --format='{{join .RootFS.Layers "\n"}}' 2>/dev/null)

if [ -z "$layers" ]; then
    echo "Image not found or no layer information available"
    exit 1
fi

echo "Checking for layer conflicts..."
for layer in $layers; do
    echo "Layer: $layer"
    for image in $(docker images -q); do
        if [ "$image" != "$(docker inspect "$image_to_remove" --format='{{.Id}}' 2>/dev/null | cut -d':' -f2 | cut -c1-12)" ]; then
            if docker inspect "$image" --format='{{.RootFS.Layers}}' 2>/dev/null | grep -q "$layer"; then
                echo "  Shared with: $(docker inspect "$image" --format='{{.RepoTags}}' 2>/dev/null)"
            fi
        fi
    done
done

echo "Attempting removal..."
docker rmi "$image_to_remove"
```

### Emergency Cleanup
```bash
#!/bin/bash
# Emergency image cleanup when normal removal fails
echo "Emergency Docker image cleanup..."

# Step 1: Stop Docker daemon
echo "Stopping Docker daemon..."
sudo systemctl stop docker

# Step 2: Clean up Docker data (DANGEROUS - USE WITH CAUTION)
read -p "WARNING: This will remove ALL Docker data. Continue? (y/N): " confirm
if [[ "$confirm" =~ ^[Yy]$ ]]; then
    sudo rm -rf /var/lib/docker/overlay2/*
    sudo rm -rf /var/lib/docker/image/*
    sudo rm -rf /var/lib/docker/containers/*
fi

# Step 3: Start Docker daemon
echo "Starting Docker daemon..."
sudo systemctl start docker

# Step 4: Verify cleanup
echo "Verifying cleanup..."
docker images
docker ps -a

echo "Emergency cleanup completed"
echo "NOTE: All images and containers have been removed"
```

### Automated Recovery Script
```bash
#!/bin/bash
# Comprehensive image removal with automatic recovery
remove_image_with_recovery() {
    local image="$1"
    local max_attempts=3
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        echo "Attempt $attempt to remove $image..."
        
        if docker rmi "$image" 2>/dev/null; then
            echo "✓ Successfully removed $image"
            return 0
        fi
        
        echo "✗ Removal failed, attempting recovery..."
        
        case $attempt in
            1)
                echo "  Stopping containers using the image..."
                docker stop $(docker ps -q --filter "ancestor=$image" 2>/dev/null) 2>/dev/null || true
                docker rm $(docker ps -aq --filter "ancestor=$image" 2>/dev/null) 2>/dev/null || true
                ;;
            2)
                echo "  Force removal attempt..."
                docker rmi -f "$image" 2>/dev/null && return 0
                ;;
            3)
                echo "  Cleaning up Docker system..."
                docker system prune -f
                docker rmi -f "$image" 2>/dev/null && return 0
                ;;
        esac
        
        ((attempt++))
        sleep 2
    done
    
    echo "✗ Failed to remove $image after $max_attempts attempts"
    echo "Manual intervention required"
    return 1
}

# Usage example
# remove_image_with_recovery "ubuntu:old"
```
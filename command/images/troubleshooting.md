# Troubleshooting

## Common Issues

### No Images Listed
**Problem**: Docker images command returns empty or shows no images
```bash
# Check Docker daemon status
sudo systemctl status docker

# Verify Docker is running
docker info

# Check if images exist in different location
sudo docker images

# Check Docker root directory
docker info | grep "Docker Root Dir"
```

### Permission Denied
**Problem**: Access denied when listing images
```bash
# Check if Docker daemon is running
sudo systemctl status docker

# Ensure user is in docker group
groups $USER | grep docker

# Add user to docker group
sudo usermod -aG docker $USER
# Log out and log back in

# Temporary solution with sudo
sudo docker images
```

### Truncated Output
**Problem**: Image IDs or names are cut off
```bash
# Show full image IDs
docker images --no-trunc

# Use custom format for better display
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.Size}}"

# Increase terminal width
export COLUMNS=200
docker images
```

### Missing or Dangling Images
**Problem**: Expected images not showing or showing as <none>
```bash
# Show all images including intermediate layers
docker images -a

# Show only dangling images
docker images --filter "dangling=true"

# Check if images were accidentally removed
docker history <image_id>

# Rebuild missing images
docker build -t <tag> .
```

### Slow Image Listing
**Problem**: docker images command takes too long
```bash
# Check system resources
df -h
free -h

# Check Docker daemon logs
sudo journalctl -u docker.service --since "5 minutes ago"

# Restart Docker daemon
sudo systemctl restart docker

# Clean up system
docker system prune -f
```

### Incorrect Image Sizes
**Problem**: Image sizes seem wrong or inconsistent
```bash
# Check actual disk usage
docker system df

# Show detailed space usage
docker system df -v

# Check individual image layers
docker history <image_name>

# Compare with inspect output
docker inspect <image_name> --format='{{.Size}}'
```

### Filter Not Working
**Problem**: Filters not returning expected results
```bash
# Verify filter syntax
docker images --filter "dangling=true"

# Use exact filter values
docker images --filter "reference=ubuntu:20.04"

# Check available filter options
docker images --help | grep -A 10 filter

# Debug with verbose output
docker images --filter "before=nginx:latest" -v
```

## Debugging Commands

### Image Information Analysis
```bash
# Detailed image information
docker inspect <image_name>

# Image history and layers
docker history <image_name>

# Image configuration
docker inspect <image_name> --format='{{json .Config}}' | jq .

# Image metadata
docker inspect <image_name> --format='{{json .ContainerConfig}}' | jq .
```

### System Resource Check
```bash
# Docker system information
docker info

# Disk space usage
docker system df
docker system df -v

# Storage driver info
docker info | grep -A 10 "Storage Driver"

# Check available space
df -h $(docker info --format '{{.DockerRootDir}}')
```

### Image Repository Investigation
```bash
# List all repositories
docker images --format "{{.Repository}}" | sort | uniq

# Count images per repository
docker images --format "{{.Repository}}" | sort | uniq -c

# Find duplicate images
docker images --format "{{.ID}}\t{{.Repository}}:{{.Tag}}" | sort

# Check image creation timeline
docker images --format "{{.CreatedAt}}\t{{.Repository}}:{{.Tag}}" | sort
```

### Network and Registry Issues
```bash
# Test registry connectivity
curl -I https://registry.hub.docker.com/v2/

# Check Docker Hub rate limits
docker pull --help | grep rate

# Verify image exists remotely
curl -s "https://registry.hub.docker.com/v2/repositories/<repo>/tags/" | jq .

# Check local registry configuration
docker info | grep -A 5 "Registry"
```

### Performance Analysis
```bash
# Monitor Docker daemon performance
top -p $(pgrep dockerd)

# Check I/O statistics
iostat -x 1 5

# Monitor disk usage in real-time
watch -n 1 'docker system df'

# Check for resource constraints
dmesg | grep -i docker
```

## Error Codes and Meanings

### Standard Exit Codes
- **0** - Successful operation
- **1** - General error
- **125** - Docker daemon error
- **126** - Permission error
- **127** - Command not found

### Docker-Specific Errors
- **Cannot connect to Docker daemon** - Docker service not running
- **Permission denied** - User not in docker group
- **No such image** - Image doesn't exist locally
- **Invalid filter** - Incorrect filter syntax
- **Out of disk space** - Insufficient storage

### Checking Command Status
```bash
# Check last command exit code
echo $?

# Verbose error output
docker images --debug

# Check Docker daemon logs
sudo journalctl -u docker.service | tail -20
```

## Recovery Procedures

### Fix Missing Images
```bash
# Step 1: Verify Docker daemon is running
sudo systemctl status docker

# Step 2: Check Docker root directory permissions
sudo ls -la $(docker info --format '{{.DockerRootDir}}')

# Step 3: Restart Docker daemon
sudo systemctl restart docker

# Step 4: Re-pull missing images
docker pull <missing_image>

# Step 5: Verify images are back
docker images
```

### Resolve Permission Issues
```bash
# Step 1: Check current user groups
groups $USER

# Step 2: Add user to docker group
sudo usermod -aG docker $USER

# Step 3: Apply group changes
newgrp docker
# Or log out and back in

# Step 4: Verify access
docker images

# Step 5: If still issues, check Docker socket
sudo ls -la /var/run/docker.sock
```

### Clean Up Corrupted Images
```bash
#!/bin/bash
# Comprehensive image cleanup and recovery
echo "Starting Docker image recovery..."

# Step 1: Stop Docker daemon
echo "Stopping Docker daemon..."
sudo systemctl stop docker

# Step 2: Check filesystem
echo "Checking Docker storage..."
sudo fsck $(docker info --format '{{.DockerRootDir}}' 2>/dev/null | cut -d'/' -f2) 2>/dev/null || echo "Filesystem check skipped"

# Step 3: Start Docker daemon
echo "Starting Docker daemon..."
sudo systemctl start docker

# Step 4: Clean up corrupted data
echo "Cleaning up corrupted images..."
docker system prune -a -f

# Step 5: Verify system health
echo "Verifying Docker health..."
docker info
docker images

echo "Recovery completed"
```

### Disk Space Recovery
```bash
#!/bin/bash
# Emergency disk space cleanup for images
echo "Emergency Docker cleanup..."

# Show current usage
echo "Current disk usage:"
docker system df

# Remove dangling images
echo "Removing dangling images..."
docker image prune -f

# Remove unused images
echo "Removing unused images..."
docker image prune -a -f

# Remove everything if critically low on space
read -p "Critical cleanup (removes all unused data)? [y/N]: " response
if [[ "$response" =~ ^[Yy]$ ]]; then
    docker system prune -a -f --volumes
fi

# Show final usage
echo "Final disk usage:"
docker system df
```

### Automated Health Check
```bash
#!/bin/bash
# Docker images health check script
echo "=== Docker Images Health Check ==="
echo "Date: $(date)"
echo

# Check Docker daemon
echo "1. Docker Daemon Status:"
sudo systemctl is-active docker
echo

# Check image count
image_count=$(docker images -q 2>/dev/null | wc -l)
echo "2. Total Images: $image_count"
echo

# Check dangling images
dangling_count=$(docker images --filter "dangling=true" -q 2>/dev/null | wc -l)
echo "3. Dangling Images: $dangling_count"
if [ $dangling_count -gt 0 ]; then
    echo "   WARNING: Consider cleaning up dangling images"
fi
echo

# Check disk usage
echo "4. Disk Usage:"
docker system df 2>/dev/null || echo "   Unable to check disk usage"
echo

# Check for large images
echo "5. Largest Images (top 5):"
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" 2>/dev/null | head -6 || echo "   Unable to list images"
echo

echo "Health check completed"
```
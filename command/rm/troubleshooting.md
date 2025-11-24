# Troubleshooting

## Common Issues

### Cannot Remove Running Container
**Problem**: Error when trying to remove running container
```
Error response from daemon: You cannot remove a running container
```

**Solutions**:
```bash
# Option 1: Stop first, then remove
docker stop my-container
docker rm my-container

# Option 2: Force remove (not recommended)
docker rm -f my-container

# Option 3: Check container status
docker ps -a | grep my-container
```

### Container Not Found
**Problem**: Container doesn't exist or name is incorrect
```
Error: No such container: my-container
```

**Solutions**:
```bash
# List all containers to verify name
docker ps -a

# Search for container with partial name
docker ps -a | grep partial-name

# Use container ID instead of name
docker rm 1a2b3c4d5e6f
```

### Permission Denied
**Problem**: Insufficient permissions to remove container
```
Got permission denied while trying to connect to Docker daemon
```

**Solutions**:
```bash
# Use sudo (temporary fix)
sudo docker rm my-container

# Add user to docker group (permanent fix)
sudo usermod -aG docker $USER
# Log out and log back in

# Check docker group membership
groups $USER | grep docker
```

### Linked Container Issues
**Problem**: Cannot remove container that has links
```
Error: Conflict, cannot remove container that has links
```

**Solutions**:
```bash
# Remove links first
docker rm -l link-name

# Or force remove with links
docker rm -f my-container

# Check container links
docker inspect my-container | grep Links
```

### Volume Mount Issues
**Problem**: Container removal leaves orphaned volumes

**Solutions**:
```bash
# Always use -v flag to remove anonymous volumes
docker rm -v my-container

# Clean up orphaned volumes
docker volume prune

# List volumes to check
docker volume ls
```

## Debugging Commands

### Container Status Check
```bash
# Check container status
docker ps -a --filter name=my-container

# Detailed container information
docker inspect my-container

# Check container processes
docker top my-container
```

### System Information
```bash
# Check Docker daemon status
sudo systemctl status docker

# Check disk space
docker system df

# Check running processes
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### Resource Investigation
```bash
# Check container resource usage
docker stats my-container --no-stream

# Check container logs before removal
docker logs my-container --tail 50

# Check container filesystem changes
docker diff my-container
```

### Network and Link Analysis
```bash
# Check container networks
docker inspect my-container --format '{{.NetworkSettings.Networks}}'

# Check container links
docker inspect my-container --format '{{.HostConfig.Links}}'

# List all networks
docker network ls
```

## Error Codes and Meanings

### Standard Exit Codes
- **0** - Successful removal
- **1** - General error
- **125** - Docker daemon error
- **404** - Container not found
- **409** - Conflict (container running or has links)

### Docker-Specific Errors
- **Container running** - Use `docker stop` first
- **Has links** - Remove links or use `--force`
- **Permission denied** - Check Docker daemon permissions
- **Daemon not running** - Start Docker service

### Checking Removal Status
```bash
# Verify container was removed
docker ps -a | grep my-container

# Check removal success (should return empty)
echo $?  # 0 = success, non-zero = error
```

## Recovery Procedures

### Stuck Container Removal
```bash
# Step 1: Try graceful removal
docker rm my-container

# Step 2: Stop and remove
docker stop my-container && docker rm my-container

# Step 3: Force removal
docker rm -f my-container

# Step 4: Check Docker daemon
sudo systemctl restart docker
```

### Bulk Removal Failures
```bash
#!/bin/bash
# Script with error handling for bulk removal
containers=("web" "db" "cache")

for container in "${containers[@]}"; do
    echo "Removing $container..."
    if docker rm -f "$container" 2>/dev/null; then
        echo "✓ Successfully removed $container"
    else
        echo "✗ Failed to remove $container"
        # Log error for investigation
        docker ps -a --filter name="$container"
    fi
done
```

### Emergency Cleanup
```bash
# When normal removal fails, nuclear option
# WARNING: This removes ALL containers

# Stop all containers
docker stop $(docker ps -q) 2>/dev/null || true

# Remove all containers
docker rm -f $(docker ps -aq) 2>/dev/null || true

# Clean up everything
docker system prune -a -f --volumes
```

### Data Recovery Before Removal
```bash
# Copy important files before removal
docker cp my-container:/important/data ./backup/

# Create image from container (preserve state)
docker commit my-container my-backup-image

# Export container filesystem
docker export my-container > container-backup.tar
```

## Prevention Best Practices

### Safe Removal Workflow
```bash
# 1. Check container status
docker ps -a --filter name=my-container

# 2. Backup important data
docker cp my-container:/data ./backup/

# 3. Stop gracefully
docker stop my-container

# 4. Remove with volumes
docker rm -v my-container

# 5. Verify removal
docker ps -a | grep my-container
```
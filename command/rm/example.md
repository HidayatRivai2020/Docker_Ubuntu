# Examples

## Basic Examples

### Remove Single Container
```bash
# Remove stopped container by name
docker rm my-container

# Remove stopped container by ID
docker rm 1a2b3c4d5e6f

# Remove stopped container by partial ID
docker rm 1a2b
```

### Remove Multiple Containers
```bash
# Remove multiple containers
docker rm container1 container2 container3

# Remove all stopped containers
docker rm $(docker ps -aq --filter status=exited)

# Remove containers using pattern matching
docker rm $(docker ps -aq --filter name=test*)
```

## Force Removal Examples

### Remove Running Container
```bash
# Force remove running container (not recommended)
docker rm -f my-running-container

# Better approach: stop then remove
docker stop my-running-container
docker rm my-running-container

# Combined stop and remove
docker stop my-running-container && docker rm my-running-container
```

### Batch Force Removal
```bash
# Force remove all containers (dangerous!)
docker rm -f $(docker ps -aq)

# Force remove all stopped containers
docker rm -f $(docker ps -aq --filter status=exited)
```

## Volume Management Examples

### Remove with Volumes
```bash
# Remove container and its anonymous volumes
docker rm -v my-container

# Remove multiple containers with volumes
docker rm -v container1 container2

# Remove all stopped containers with volumes
docker rm -v $(docker ps -aq --filter status=exited)
```

### Named Volume Handling
```bash
# Named volumes are NOT removed with -v flag
docker rm -v my-container  # Named volumes persist

# To remove named volumes separately
docker volume rm my-named-volume

# List volumes to check what remains
docker volume ls
```

## Advanced Examples

### Conditional Removal
```bash
# Remove containers older than 24 hours
docker container prune --filter "until=24h"

# Remove containers with specific labels
docker rm $(docker ps -aq --filter label=environment=test)

# Remove containers created before specific container
docker rm $(docker ps -aq --filter before=my-container)
```

### Safe Removal Scripts
```bash
#!/bin/bash
# Safe container removal script
container_name="my-app"

if docker ps -q --filter name="$container_name" | grep -q .; then
    echo "Stopping $container_name..."
    docker stop "$container_name"
fi

if docker ps -aq --filter name="$container_name" | grep -q .; then
    echo "Removing $container_name..."
    docker rm -v "$container_name"
    echo "Container removed successfully"
else
    echo "Container $container_name not found"
fi
```

## Common Use Cases

### Development Workflow
```bash
# Clean up test containers after development
docker stop test-db test-web test-cache
docker rm -v test-db test-web test-cache

# Or as one-liner
docker rm -f -v test-db test-web test-cache
```

### CI/CD Pipeline Cleanup
```bash
# Remove build containers after CI job
docker rm $(docker ps -aq --filter label=ci-job=build-123)

# Clean up temporary containers
docker container prune -f
```

### Maintenance Tasks
```bash
# Weekly cleanup of stopped containers
docker rm $(docker ps -aq --filter status=exited --filter status=dead)

# Remove containers that exited with errors
docker rm $(docker ps -aq --filter exited=1)
```

### Emergency Cleanup
```bash
# Free up disk space quickly
docker container prune -f
docker image prune -f
docker volume prune -f

# Nuclear option (removes everything)
docker system prune -a -f --volumes
```

## Common Patterns

### Stop and Remove Pattern
```bash
# Function to stop and remove container
stop_and_remove() {
    local container=$1
    docker stop "$container" 2>/dev/null || true
    docker rm -v "$container" 2>/dev/null || true
}

# Usage
stop_and_remove my-container
```

### Bulk Operations
```bash
# Remove all containers with specific prefix
docker rm -f $(docker ps -aq --filter name=temp-)

# Remove containers older than specific date
docker rm $(docker ps -aq --filter before=2023-01-01)
```
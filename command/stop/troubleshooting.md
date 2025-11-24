# Troubleshooting

## Common Issues

### Container Won't Stop
**Problem**: Container doesn't respond to stop command
```bash
# Check if container is running
docker ps | grep <container_name>

# Force stop with shorter timeout
docker stop -t 5 <container_name>

# If still running, force kill
docker kill <container_name>
```

### Timeout Issues
**Problem**: Container takes too long to stop
```bash
# Increase timeout for graceful shutdown
docker stop -t 30 <container_name>

# Check what processes are running
docker exec <container_name> ps aux
```

### Multiple Container Stop Failures
**Problem**: Some containers in batch stop fail
```bash
# Stop containers one by one to identify problematic ones
for container in container1 container2 container3; do
    echo "Stopping $container..."
    docker stop "$container" || echo "Failed to stop $container"
done
```

### Database Container Stop Issues
**Problem**: Database containers exit with errors when stopped
```bash
# Allow more time for database cleanup
docker stop -t 60 mysql_container

# Check database logs before stopping
docker logs mysql_container --tail 50
```

### Permission Errors
**Problem**: Access denied when stopping containers
```bash
# Check if Docker daemon is running
sudo systemctl status docker

# Ensure user is in docker group
groups $USER | grep docker

# If not in group, add user
sudo usermod -aG docker $USER
```

## Debugging Commands

### Check Container Status
```bash
# View all containers with status
docker ps -a

# Check specific container details
docker inspect <container_name>

# Monitor container in real-time
docker stats <container_name>
```

### Process Investigation
```bash
# Check processes inside container
docker exec <container_name> ps aux

# Check process tree
docker exec <container_name> ps auxf

# Monitor system calls (if available)
docker exec <container_name> strace -p 1
```

### Log Analysis
```bash
# View container logs
docker logs <container_name>

# Follow logs in real-time
docker logs -f <container_name>

# View logs with timestamps
docker logs -t <container_name>

# View recent logs only
docker logs --tail 100 <container_name>
```

### Resource Monitoring
```bash
# Check resource usage
docker stats --no-stream <container_name>

# Check disk usage
docker system df

# Check for resource constraints
docker inspect <container_name> | grep -i memory
docker inspect <container_name> | grep -i cpu
```

## Exit Codes and Meanings

### Standard Exit Codes
- **0** - Clean shutdown (SIGTERM handled properly)
- **1** - General application error
- **2** - Misuse of shell command
- **125** - Docker daemon error
- **126** - Container command not executable
- **127** - Container command not found
- **137** - Container killed by SIGKILL (timeout exceeded)
- **143** - Container terminated by SIGTERM

### Timeout-Related Codes
- **137** - Process killed after timeout (SIGKILL)
- **143** - Process terminated gracefully (SIGTERM)

### Checking Exit Codes
```bash
# Check exit code of stopped container
docker inspect <container_name> --format='{{.State.ExitCode}}'

# View detailed state information
docker inspect <container_name> --format='{{json .State}}' | jq

# Check when container was stopped
docker inspect <container_name> --format='{{.State.FinishedAt}}'
```

## Recovery Procedures

### Force Stop Unresponsive Container
```bash
# Step 1: Try graceful stop with extended timeout
docker stop -t 30 <container_name>

# Step 2: If still running, force kill
docker kill <container_name>

# Step 3: Verify container is stopped
docker ps | grep <container_name>

# Step 4: Check exit status
docker inspect <container_name> --format='{{.State.Status}}'
```

### Handle Stuck Container
```bash
# Check Docker daemon logs
sudo journalctl -u docker.service --since "10 minutes ago"

# Restart Docker daemon (last resort)
sudo systemctl restart docker

# Verify all containers after daemon restart
docker ps -a
```

### Batch Stop with Error Handling
```bash
#!/bin/bash
# Script to stop multiple containers with proper error handling
containers=("web" "db" "cache")
timeout=30

for container in "${containers[@]}"; do
    echo "Stopping $container with $timeout second timeout..."
    if docker stop -t $timeout "$container"; then
        echo "✓ Successfully stopped $container"
    else
        echo "✗ Failed to stop $container, trying force kill..."
        if docker kill "$container"; then
            echo "✓ Force killed $container"
        else
            echo "✗ Failed to kill $container - manual intervention required"
        fi
    fi
done
```
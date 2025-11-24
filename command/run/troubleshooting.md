# Troubleshooting

## Common Issues

### Port Already in Use
**Problem**: Cannot bind to host port because it's already occupied
```bash
# Check what's using the port
netstat -tulpn | grep :8080
# or
ss -tulpn | grep :8080

# Use different host port
docker run -p 8081:80 nginx

# Stop conflicting service
sudo systemctl stop apache2
# or kill specific process
sudo kill -9 <PID>
```

### Permission Denied
**Problem**: Access denied when accessing mounted volumes or running commands
```bash
# Check file permissions on host
ls -la /host/path

# Run container as specific user
docker run --user $(id -u):$(id -g) -v /host/path:/container/path ubuntu

# Fix ownership on host
sudo chown -R $USER:$USER /host/path

# Run with root (temporary solution)
docker run --user root -v /host/path:/container/path ubuntu
```

### Container Exits Immediately
**Problem**: Container starts but exits right away
```bash
# Check container logs
docker logs <container_name>

# Run container interactively to debug
docker run -it <image_name> bash

# Check if main process is running
docker run <image_name> ps aux

# Override entrypoint to debug
docker run --entrypoint="" <image_name> bash
```

### Image Not Found
**Problem**: Docker cannot find the specified image
```bash
# Check if image exists locally
docker images | grep <image_name>

# Pull image manually
docker pull <image_name>:<tag>

# Verify image name and tag
docker search <image_name>

# Check available tags on Docker Hub
curl -s "https://registry.hub.docker.com/v2/repositories/<image_name>/tags/" | jq '.results[].name'
```

### Out of Disk Space
**Problem**: No space left on device when running containers
```bash
# Check disk usage
df -h
docker system df

# Clean up unused resources
docker system prune -a

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Check container sizes
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### Network Issues
**Problem**: Container cannot connect to network or internet
```bash
# Check container network settings
docker inspect <container_name> --format='{{.NetworkSettings.Networks}}'

# Test network connectivity
docker run --rm alpine ping google.com

# Use host networking
docker run --network host <image_name>

# Check DNS resolution
docker run --rm alpine nslookup google.com
```

### Resource Constraints
**Problem**: Container fails due to memory or CPU limits
```bash
# Check container resource usage
docker stats <container_name>

# Increase memory limit
docker run -m 2g <image_name>

# Increase CPU limit
docker run --cpus="2.0" <image_name>

# Check system resources
free -h
cat /proc/cpuinfo | grep processor | wc -l
```

## Debugging Commands

### Container Status and Logs
```bash
# View container logs
docker logs <container_name>

# Follow logs in real-time
docker logs -f <container_name>

# View logs with timestamps
docker logs -t <container_name>

# View recent logs only
docker logs --tail 100 <container_name>

# Check container status
docker ps -a --filter name=<container_name>
```

### Container Inspection
```bash
# Inspect container configuration
docker inspect <container_name>

# Check specific configuration
docker inspect <container_name> --format='{{.Config.Env}}'
docker inspect <container_name> --format='{{.Mounts}}'
docker inspect <container_name> --format='{{.NetworkSettings.Ports}}'

# Check container filesystem changes
docker diff <container_name>
```

### Process and Resource Monitoring
```bash
# Check processes inside container
docker exec <container_name> ps aux

# Check process tree
docker exec <container_name> ps auxf

# Monitor resource usage
docker stats <container_name> --no-stream

# Check container top processes
docker top <container_name>

# Execute interactive shell
docker exec -it <container_name> bash
```

### Network and Connectivity
```bash
# Test network connectivity
docker exec <container_name> ping google.com

# Check network interfaces
docker exec <container_name> ip addr show

# Check routing table
docker exec <container_name> ip route show

# Test specific port
docker exec <container_name> nc -zv host.docker.internal 80
```

### Image and Layer Analysis
```bash
# Check image history
docker history <image_name>

# Check image layers
docker inspect <image_name> --format='{{.RootFS.Layers}}'

# Analyze image size
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Check image vulnerabilities (if scanner available)
docker scan <image_name>
```

## Exit Codes and Meanings

### Standard Exit Codes
- **0** - Successful execution
- **1** - General application error
- **2** - Misuse of shell command
- **125** - Docker daemon error
- **126** - Container command not executable
- **127** - Container command not found
- **128** - Invalid exit argument
- **130** - Container terminated by Ctrl+C (SIGINT)
- **137** - Container killed by SIGKILL
- **143** - Container terminated by SIGTERM

### Application-Specific Codes
- **3-124** - Application-specific error codes
- **255** - Exit status out of range

### Checking Exit Codes
```bash
# Check exit code of stopped container
docker inspect <container_name> --format='{{.State.ExitCode}}'

# View detailed state information
docker inspect <container_name> --format='{{json .State}}' | jq

# Check when container exited
docker inspect <container_name> --format='{{.State.FinishedAt}}'

# View container events
docker events --filter container=<container_name>
```

## Recovery Procedures

### Debug Failed Container Start
```bash
# Step 1: Check recent containers
docker ps -a --limit 5

# Step 2: Check logs of failed container
docker logs <failed_container_id>

# Step 3: Try running with different options
docker run -it --entrypoint="" <image_name> bash

# Step 4: Check image integrity
docker inspect <image_name>
```

### Handle Resource Issues
```bash
# Check system resources
docker system df
df -h

# Clean up resources
docker system prune -f

# Restart Docker daemon if needed
sudo systemctl restart docker

# Check daemon logs
sudo journalctl -u docker.service --since "10 minutes ago"
```

### Network Troubleshooting
```bash
#!/bin/bash
# Network troubleshooting script
container_name="$1"

echo "=== Network Troubleshooting for $container_name ==="

echo "1. Container network settings:"
docker inspect "$container_name" --format='{{.NetworkSettings.Networks}}'

echo "2. Testing external connectivity:"
docker exec "$container_name" ping -c 3 google.com || echo "External connectivity failed"

echo "3. Testing DNS resolution:"
docker exec "$container_name" nslookup google.com || echo "DNS resolution failed"

echo "4. Checking network interfaces:"
docker exec "$container_name" ip addr show

echo "5. Checking routing:"
docker exec "$container_name" ip route show
```

### Automated Debugging Script
```bash
#!/bin/bash
# Comprehensive container debugging script
container_name="$1"

if [ -z "$container_name" ]; then
    echo "Usage: $0 <container_name>"
    exit 1
fi

echo "=== Docker Container Debug Report ==="
echo "Container: $container_name"
echo "Timestamp: $(date)"
echo

echo "=== Container Status ==="
docker ps -a --filter name="$container_name"
echo

echo "=== Container Logs (last 50 lines) ==="
docker logs --tail 50 "$container_name"
echo

echo "=== Container Configuration ==="
docker inspect "$container_name" --format='{{json .Config}}' | jq .
echo

echo "=== Resource Usage ==="
docker stats --no-stream "$container_name"
echo

echo "=== Network Settings ==="
docker inspect "$container_name" --format='{{json .NetworkSettings}}' | jq .
echo

echo "=== Volume Mounts ==="
docker inspect "$container_name" --format='{{json .Mounts}}' | jq .
```
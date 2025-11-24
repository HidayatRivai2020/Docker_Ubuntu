# Troubleshooting

## Common Issues

### No Logs Available
**Problem**: `docker logs` command shows no output
```
# Command returns empty output
docker logs mycontainer
```

**Solutions**:
```bash
# Check if container exists and is running
docker ps -a | grep mycontainer

# Check container status
docker inspect --format='{{.State.Status}}' mycontainer

# Check if container has generated any logs
docker inspect --format='{{.LogPath}}' mycontainer
ls -la $(docker inspect --format='{{.LogPath}}' mycontainer)

# Check log driver configuration
docker inspect --format='{{.HostConfig.LogConfig.Type}}' mycontainer

# Try getting logs from recently stopped container
docker logs --since 1h mycontainer
```

### Permission Denied
**Problem**: Cannot access container logs due to permissions
```
Error: permission denied while trying to access log file
```

**Solutions**:
```bash
# Use sudo to access logs
sudo docker logs mycontainer

# Check Docker daemon permissions
sudo systemctl status docker

# Add user to docker group
sudo usermod -aG docker $USER
# Log out and log back in

# Check log file permissions
sudo ls -la /var/lib/docker/containers/*/
```

### Logs Too Large
**Problem**: Log output overwhelms terminal or takes too long
```
# Large log output causes terminal to hang
docker logs large-container
```

**Solutions**:
```bash
# Limit output with --tail
docker logs --tail 100 mycontainer

# Use time-based filtering
docker logs --since 1h mycontainer

# Redirect to file for large outputs
docker logs mycontainer > container_logs.txt 2>&1

# Use pagination
docker logs --tail 100 mycontainer | less

# Stream to file while viewing
docker logs -f mycontainer | tee logs.txt
```

### Log Following Stops
**Problem**: `docker logs -f` stops following unexpectedly
```
# Following stops without obvious reason
docker logs -f mycontainer
```

**Solutions**:
```bash
# Check if container is still running
docker ps | grep mycontainer

# Restart following if container restarted
docker logs -f --since 1m mycontainer

# Use timeout to auto-restart following
timeout 300 docker logs -f mycontainer || docker logs -f mycontainer

# Check system resources
free -h
df -h /var/lib/docker
```

### Timestamp Issues
**Problem**: Timestamps are missing or incorrect
```
# Logs show without timestamps or wrong time
docker logs -t mycontainer
```

**Solutions**:
```bash
# Ensure container timezone is set
docker run -e TZ=America/New_York myimage

# Check host system time
date
timedatectl status

# Verify container time configuration
docker exec mycontainer date

# Check log driver timestamp format
docker inspect --format='{{.HostConfig.LogConfig}}' mycontainer
```

### Log Driver Issues
**Problem**: Logs not available due to log driver configuration
```
Error: configured logging driver does not support reading
```

**Solutions**:
```bash
# Check current log driver
docker inspect --format='{{.HostConfig.LogConfig.Type}}' mycontainer

# List supported log drivers
docker info | grep "Logging Driver"

# Run container with json-file driver
docker run --log-driver json-file myimage

# Check daemon configuration
sudo cat /etc/docker/daemon.json | grep log-driver

# Restart container with different log driver
docker run --log-driver=json-file --name newcontainer myimage
```

### Network/Remote Logging Issues
**Problem**: Logs not reaching external logging systems
```
# Logs not appearing in external systems (syslog, fluentd, etc.)
```

**Solutions**:
```bash
# Test local json-file logging
docker run --log-driver json-file --name test nginx
docker logs test

# Check network connectivity to logging service
telnet logging-server 514

# Verify logging configuration
docker inspect --format='{{.HostConfig.LogConfig}}' mycontainer

# Test with different log driver
docker run --log-driver syslog --log-opt syslog-address=tcp://localhost:514 myimage
```

## Debugging Commands

### Container Log Diagnostics
```bash
# Check container logging configuration
docker inspect mycontainer | jq '.[0].HostConfig.LogConfig'

# Find container log file location
docker inspect --format='{{.LogPath}}' mycontainer

# Check log file size and permissions
sudo ls -lh $(docker inspect --format='{{.LogPath}}' mycontainer)

# Manually read log file
sudo cat $(docker inspect --format='{{.LogPath}}' mycontainer)

# Check log rotation settings
docker inspect mycontainer | jq '.[0].HostConfig.LogConfig.Config'
```

### System Log Analysis
```bash
# Check Docker daemon logs
sudo journalctl -u docker.service --since "1 hour ago"

# Check system log space
df -h /var/lib/docker

# Check for Docker daemon errors
sudo journalctl -u docker.service | grep -i error

# Monitor Docker daemon in real-time
sudo journalctl -u docker.service -f

# Check system resource usage
docker system df
docker system info
```

### Log Driver Testing
```bash
# Test different log drivers
echo "Testing json-file driver:"
docker run --rm --log-driver json-file alpine echo "test json-file"

echo "Testing syslog driver:"
docker run --rm --log-driver syslog alpine echo "test syslog"

echo "Testing journald driver:"
docker run --rm --log-driver journald alpine echo "test journald"

# Check which drivers are available
docker info | grep -A 10 "Log.*Drivers"
```

### Container State Analysis
```bash
# Check container lifecycle
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Command}}"

# Check container restart history
docker inspect --format='{{.RestartCount}}' mycontainer

# Get container creation and start times
docker inspect --format='Created: {{.Created}} Started: {{.State.StartedAt}}' mycontainer

# Check if container process is still running
docker exec mycontainer ps aux
```

## Error Codes and Meanings

### Standard Exit Codes
- **0** - Successful log retrieval
- **1** - General error (container not found, permission denied)
- **125** - Docker daemon error
- **126** - Container not executable
- **127** - Container command not found

### Docker-Specific Log Errors
- **"no such container"** - Container doesn't exist or was removed
- **"configured logging driver does not support reading"** - Incompatible log driver
- **"permission denied"** - Insufficient permissions to access logs
- **"log file does not exist"** - Container hasn't generated logs yet
- **"log driver not supported"** - Invalid or unavailable log driver

### Error Checking
```bash
# Check command exit status
docker logs mycontainer
echo "Exit code: $?"

# Capture and analyze errors
error_output=$(docker logs mycontainer 2>&1)
if echo "$error_output" | grep -q "no such container"; then
    echo "Container not found"
elif echo "$error_output" | grep -q "permission denied"; then
    echo "Permission issue"
fi

# Check for specific error patterns
docker logs mycontainer 2>&1 | grep -E "Error|Failed|Exception" || echo "No errors in logs"
```

## Recovery Procedures

### Container Log Recovery
```bash
#!/bin/bash
# Recover logs when standard commands fail
container="$1"

if [ -z "$container" ]; then
    echo "Usage: $0 <container_name>"
    exit 1
fi

echo "=== Log Recovery for $container ==="

# Step 1: Check if container exists
if ! docker ps -a --format "{{.Names}}" | grep -q "^$container$"; then
    echo "Container '$container' not found"
    echo "Available containers:"
    docker ps -a --format "table {{.Names}}\t{{.Status}}"
    exit 1
fi

echo "Container found ✓"

# Step 2: Check container status
status=$(docker inspect --format='{{.State.Status}}' "$container")
echo "Container status: $status"

# Step 3: Check log driver
log_driver=$(docker inspect --format='{{.HostConfig.LogConfig.Type}}' "$container")
echo "Log driver: $log_driver"

if [ "$log_driver" != "json-file" ]; then
    echo "⚠️ Warning: Log driver '$log_driver' may not support log reading"
fi

# Step 4: Try to get log file path
log_path=$(docker inspect --format='{{.LogPath}}' "$container")
echo "Log file path: $log_path"

if [ -f "$log_path" ]; then
    echo "Log file exists ✓"
    echo "Log file size: $(du -h "$log_path" | cut -f1)"
    
    # Try to read logs directly
    echo "Attempting direct log read..."
    if sudo tail -10 "$log_path" >/dev/null 2>&1; then
        echo "Direct log read successful ✓"
        sudo tail -20 "$log_path"
    else
        echo "Direct log read failed ✗"
    fi
else
    echo "Log file not found ✗"
fi

# Step 5: Try standard docker logs with different options
echo
echo "Trying different log retrieval methods:"

# Try without any options
if docker logs "$container" >/dev/null 2>&1; then
    echo "Standard logs available ✓"
else
    echo "Standard logs failed ✗"
fi

# Try with time filter
if docker logs --since 24h "$container" >/dev/null 2>&1; then
    echo "Time-filtered logs available ✓"
else
    echo "Time-filtered logs failed ✗"
fi

# Try with tail
if docker logs --tail 10 "$container" >/dev/null 2>&1; then
    echo "Tail logs available ✓"
else
    echo "Tail logs failed ✗"
fi
```

### Log Driver Recovery
```bash
#!/bin/bash
# Recover from log driver issues
echo "=== Log Driver Recovery ==="

# Check current default log driver
default_driver=$(docker info | grep "Logging Driver" | awk '{print $3}')
echo "Default log driver: $default_driver"

# Check available log drivers
echo "Available log drivers:"
docker info | grep -A 10 "Log.*Drivers" | tail -n +2

# Check daemon configuration
echo
echo "Docker daemon log configuration:"
if [ -f "/etc/docker/daemon.json" ]; then
    echo "Daemon config file exists:"
    sudo cat /etc/docker/daemon.json | grep -A 5 -B 5 "log"
else
    echo "No daemon.json file found"
fi

# Test creating container with json-file driver
echo
echo "Testing container creation with json-file driver..."
test_container="log-test-$(date +%s)"

if docker run --name "$test_container" --log-driver json-file alpine echo "test log" >/dev/null 2>&1; then
    echo "Container creation successful ✓"
    
    # Test log retrieval
    if docker logs "$test_container" >/dev/null 2>&1; then
        echo "Log retrieval successful ✓"
    else
        echo "Log retrieval failed ✗"
    fi
    
    # Cleanup
    docker rm "$test_container" >/dev/null 2>&1
else
    echo "Container creation failed ✗"
fi

echo
echo "Recommendations:"
echo "1. Use json-file log driver for containers requiring log access"
echo "2. Configure log rotation to prevent disk space issues"
echo "3. Consider external logging solutions for production"
```

### System Recovery
```bash
#!/bin/bash
# Recover from system-level log issues
echo "=== System Log Recovery ==="

# Check disk space
echo "Checking disk space:"
df -h /var/lib/docker
echo

# Check Docker daemon status
echo "Docker daemon status:"
sudo systemctl status docker --no-pager
echo

# Check for log rotation issues
echo "Checking log file sizes:"
sudo find /var/lib/docker/containers -name "*-json.log" -exec ls -lh {} \; | \
    awk '$5 ~ /[0-9]+[GM]/ {print $9 ": " $5}' | head -10

echo
echo "Large log files (>100MB):"
sudo find /var/lib/docker/containers -name "*-json.log" -size +100M -exec ls -lh {} \;

# Check system logs for Docker issues
echo
echo "Recent Docker daemon errors:"
sudo journalctl -u docker.service --since "1 hour ago" | grep -i error | tail -5

# Memory and resource check
echo
echo "System resources:"
echo "Memory usage: $(free | awk 'NR==2{printf "%.1f%%", $3*100/$2}')"
echo "Docker space usage:"
docker system df

# Cleanup recommendations
echo
echo "Cleanup commands if needed:"
echo "  Clean unused containers: docker container prune"
echo "  Clean unused images: docker image prune"
echo "  Clean everything unused: docker system prune -a"
echo "  Restart Docker daemon: sudo systemctl restart docker"
```

### Emergency Log Access
```bash
#!/bin/bash
# Emergency access to container logs when docker logs fails
container_name="$1"

if [ -z "$container_name" ]; then
    echo "Usage: $0 <container_name>"
    exit 1
fi

echo "=== Emergency Log Access ==="

# Find container ID
container_id=$(docker ps -aq --filter name="$container_name")

if [ -z "$container_id" ]; then
    echo "Container not found: $container_name"
    exit 1
fi

echo "Container ID: $container_id"

# Try to access log file directly
log_dir="/var/lib/docker/containers/$container_id"
log_file="$log_dir/$container_id-json.log"

echo "Log file: $log_file"

if [ -f "$log_file" ]; then
    echo "Reading log file directly..."
    echo "File size: $(sudo du -h "$log_file" | cut -f1)"
    echo "Last 20 lines:"
    sudo tail -20 "$log_file" | while read line; do
        # Parse JSON log format
        echo "$line" | python3 -c "
import json
import sys
try:
    data = json.loads(sys.stdin.read())
    print(f\"{data.get('time', '')}: {data.get('log', '').rstrip()}\")
except:
    print(sys.stdin.read().strip())
" 2>/dev/null || echo "$line"
    done
else
    echo "Log file not found: $log_file"
    echo "Available files in container directory:"
    sudo ls -la "$log_dir" 2>/dev/null || echo "Container directory not found"
fi

# Try alternative methods
echo
echo "Alternative access methods:"

# Try exec if container is running
if docker inspect --format='{{.State.Status}}' "$container_name" 2>/dev/null | grep -q "running"; then
    echo "Container is running - trying exec access:"
    docker exec "$container_name" ps aux 2>/dev/null || echo "Exec failed"
fi

# Check for mounted log volumes
echo
echo "Checking for mounted log volumes:"
docker inspect "$container_name" | grep -A 5 -B 5 "Mounts" 2>/dev/null || echo "No mount information"
```
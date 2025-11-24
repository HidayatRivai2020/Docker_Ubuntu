# Troubleshooting

## Common Issues

### No Containers Displayed
**Problem**: `docker ps` shows no output or empty table
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS   PORTS   NAMES
```

**Solutions**:
```bash
# Check if any containers exist
docker ps -a

# Verify Docker daemon is running
sudo systemctl status docker

# Start Docker daemon if stopped
sudo systemctl start docker

# Check for running containers specifically
docker container ls

# Look for recently stopped containers
docker ps -a --filter "since=1h"
```

### Permission Denied
**Problem**: Cannot access Docker daemon
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solutions**:
```bash
# Use sudo (temporary fix)
sudo docker ps

# Add user to docker group (permanent fix)
sudo usermod -aG docker $USER
# Log out and log back in

# Verify group membership
groups $USER | grep docker

# Check Docker socket permissions
ls -la /var/run/docker.sock
```

### Filter Not Working
**Problem**: Filters return unexpected results or no results
```
# This might not work as expected
docker ps --filter name=web*
```

**Solutions**:
```bash
# Use exact name matching
docker ps --filter name=web-server

# Use partial name matching (no wildcards needed)
docker ps --filter name=web

# Check filter syntax
docker ps --filter "status=running"
docker ps --filter "label=environment=production"

# Verify filter values exist
docker ps -a --no-trunc | grep web
```

### Truncated Output
**Problem**: Container information is cut off in display
```
CONTAINER ID   IMAGE     COMMAND                  CREATED   STATUS
1a2b3c4d5e6f   nginx     "/docker-entrypoint.…"   2 hours   Up 2 hours
```

**Solutions**:
```bash
# Show full output without truncation
docker ps --no-trunc

# Use custom format to show specific fields
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Command}}"

# Adjust terminal width
export COLUMNS=200
docker ps

# Use format templates for specific information
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"
```

### Format Template Errors
**Problem**: Invalid template syntax in `--format` option
```
template: :1:15: executing template: map has no entry for key "InvalidField"
```

**Solutions**:
```bash
# Check available template fields
docker ps --help | grep -A 20 "format"

# Use correct field names (case-sensitive)
docker ps --format "{{.Names}}"     # Correct
docker ps --format "{{.Name}}"      # Incorrect

# Test templates step by step
docker ps --format "{{.ID}}"
docker ps --format "{{.ID}}\t{{.Names}}"

# Valid template fields:
# .ID, .Image, .Command, .CreatedAt, .RunningFor
# .Ports, .Status, .Size, .Names, .Labels, .Mounts, .Networks
```

### Network Information Missing
**Problem**: Network details not showing in ps output
```
# Networks column is empty or not visible
```

**Solutions**:
```bash
# Use custom format to show networks
docker ps --format "table {{.Names}}\t{{.Networks}}"

# Inspect container for detailed network info
docker inspect container_name | jq '.[].NetworkSettings.Networks'

# Check if container is connected to networks
docker network ls
docker network inspect bridge

# Use inspect for comprehensive network details
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name
```

### Size Information Not Available
**Problem**: Size column shows 0B or is empty
```
NAMES     SIZE
web       0B
```

**Solutions**:
```bash
# Ensure container is running for accurate size
docker ps -s --filter status=running

# Check if filesystem supports size calculation
docker system df

# Use docker stats for real-time resource usage
docker stats --no-stream

# Inspect container filesystem usage
docker exec container_name du -sh /
```

## Debugging Commands

### Container Discovery
```bash
# Comprehensive container listing
docker ps -a --no-trunc

# Check all container states
for status in created restarting running removing paused exited dead; do
    count=$(docker ps -aq --filter status=$status | wc -l)
    echo "$status: $count containers"
done

# Find containers by partial name
container_pattern="web"
docker ps -a --filter name=$container_pattern

# Search in all container metadata
docker ps -a --format "{{.Names}} {{.Image}} {{.Command}}" | grep -i search_term
```

### Filter Debugging
```bash
# Test filter combinations
echo "Testing filters..."

# Status filters
for status in running paused exited dead; do
    count=$(docker ps --filter status=$status -q | wc -l)
    echo "Status $status: $count containers"
done

# Label filters
docker ps --format "{{.Names}} {{.Labels}}" | grep -v "^$"

# Image filters
docker ps -a --format "{{.Image}}" | sort | uniq -c

# Network filters
for network in $(docker network ls --format "{{.Name}}"); do
    count=$(docker ps --filter network=$network -q | wc -l)
    [ $count -gt 0 ] && echo "Network $network: $count containers"
done
```

### System Information
```bash
# Docker system status
docker version
docker info

# Check daemon connectivity
docker system ping

# System resource usage
docker system df
docker system events --since 1h | head -20

# Container statistics
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

### Output Analysis
```bash
# Analyze container output patterns
echo "=== Container Analysis ==="

# Count by status
echo "Status Distribution:"
docker ps -a --format "{{.Status}}" | \
    sed 's/Up .*/Running/' | \
    sed 's/Exited .*/Stopped/' | \
    sort | uniq -c

echo "Image Distribution:"
docker ps -a --format "{{.Image}}" | sort | uniq -c | sort -nr

echo "Creation Timeline:"
docker ps -a --format "{{.CreatedAt}} {{.Names}}" | sort
```

### Network Troubleshooting
```bash
# Network connectivity analysis
echo "=== Network Analysis ==="

# Container network mappings
for container in $(docker ps --format "{{.Names}}"); do
    echo "Container: $container"
    docker inspect --format='{{range .NetworkSettings.Networks}}Network: {{.NetworkID}} IP: {{.IPAddress}}{{end}}' $container
    echo
done

# Port mapping analysis
echo "Port Mappings:"
docker ps --format "{{.Names}} {{.Ports}}" | grep -v "^.*  $" | \
    while read name ports; do
        echo "$name: $ports"
    done
```

## Error Codes and Meanings

### Standard Exit Codes
- **0** - Successful command execution
- **1** - General error (daemon not accessible, invalid options)
- **125** - Docker daemon error
- **126** - Container command not executable
- **127** - Container command not found

### Docker-Specific Errors
- **Connection refused** - Docker daemon not running
- **Permission denied** - User lacks Docker daemon access
- **No such container** - Container ID/name not found
- **Invalid filter** - Malformed filter syntax
- **Template error** - Invalid format template syntax

### Exit Code Checking
```bash
# Check command success
docker ps > /dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "Docker ps successful"
else
    echo "Docker ps failed with exit code: $?"
fi

# Capture and analyze errors
output=$(docker ps 2>&1)
if echo "$output" | grep -q "permission denied"; then
    echo "Permission issue detected"
elif echo "$output" | grep -q "connection refused"; then
    echo "Docker daemon not running"
fi
```

## Recovery Procedures

### Docker Daemon Recovery
```bash
#!/bin/bash
# Recover Docker daemon access
echo "=== Docker Daemon Recovery ==="

# Check if daemon is running
if ! systemctl is-active docker >/dev/null 2>&1; then
    echo "Docker daemon not running, starting..."
    sudo systemctl start docker
    sleep 5
fi

# Test connectivity
if docker ps >/dev/null 2>&1; then
    echo "Docker daemon accessible ✓"
else
    echo "Docker daemon not accessible ✗"
    
    # Check permissions
    if groups $USER | grep -q docker; then
        echo "User in docker group ✓"
    else
        echo "Adding user to docker group..."
        sudo usermod -aG docker $USER
        echo "Please log out and log back in"
    fi
fi

# Display status
echo "Current status:"
sudo systemctl status docker --no-pager
```

### Filter Recovery
```bash
#!/bin/bash
# Recover from filter issues
filter_term="$1"

if [ -z "$filter_term" ]; then
    echo "Usage: $0 <search_term>"
    exit 1
fi

echo "=== Filter Recovery for: $filter_term ==="

# Try different filter approaches
echo "Searching by name:"
docker ps -a --filter name="$filter_term" --format "table {{.Names}}\t{{.Status}}"

echo "Searching by image:"
docker ps -a --filter ancestor="$filter_term" --format "table {{.Names}}\t{{.Image}}"

echo "Searching in all metadata:"
docker ps -a --format "{{.Names}} {{.Image}} {{.Command}}" | grep -i "$filter_term"

echo "Partial ID search:"
docker ps -a | grep "$filter_term"
```

### Output Format Recovery
```bash
#!/bin/bash
# Recover from format template issues
echo "=== Format Template Recovery ==="

# Test basic templates
echo "Testing basic templates:"
templates=(
    "{{.ID}}"
    "{{.Names}}"
    "{{.Image}}"
    "{{.Status}}"
    "{{.Command}}"
)

for template in "${templates[@]}"; do
    echo "Testing: $template"
    if docker ps --format "$template" >/dev/null 2>&1; then
        echo "  ✓ Works"
        docker ps --format "$template" | head -3
    else
        echo "  ✗ Failed"
    fi
    echo
done

# Show all available fields
echo "Available template fields:"
echo ".ID .Image .Command .CreatedAt .RunningFor .Ports .Status .Size .Names .Labels .Mounts .Networks"
```

### Container Discovery Recovery
```bash
#!/bin/bash
# Comprehensive container discovery when ps shows nothing
echo "=== Container Discovery Recovery ==="

# Check if Docker is working at all
if ! docker version >/dev/null 2>&1; then
    echo "Docker not accessible - check daemon and permissions"
    exit 1
fi

echo "Docker system information:"
docker system info | grep -E "Containers|Running|Paused|Stopped"

echo
echo "All containers (including hidden):"
docker container ls -a --no-trunc

echo
echo "Recently created containers:"
docker ps -a --filter "since=24h" --format "table {{.Names}}\t{{.CreatedAt}}\t{{.Status}}"

echo
echo "Container events (last hour):"
docker system events --since=1h --until=now | grep container | tail -10

echo
echo "Images with containers:"
for image in $(docker images --format "{{.Repository}}:{{.Tag}}"); do
    count=$(docker ps -a --filter ancestor="$image" -q | wc -l)
    [ $count -gt 0 ] && echo "  $image: $count containers"
done
```

### Performance Issues Recovery
```bash
#!/bin/bash
# Address slow ps performance
echo "=== PS Performance Recovery ==="

# Check system resources
echo "System resources:"
echo "  Memory: $(free -h | awk 'NR==2{print $3"/"$2}')"
echo "  CPU Load: $(uptime | awk -F'load average:' '{print $2}')"
echo "  Disk: $(df -h /var/lib/docker 2>/dev/null | awk 'NR==2{print $5}' || echo "N/A")"

echo
echo "Docker resource usage:"
docker system df

echo
echo "Container count by status:"
for status in running paused exited dead; do
    count=$(docker ps -aq --filter status=$status | wc -l)
    echo "  $status: $count"
done

echo
echo "Large containers (by size):"
docker ps -as --format "table {{.Names}}\t{{.Size}}" | sort -k2 -hr | head -5

# Cleanup suggestions
echo
echo "Cleanup suggestions:"
exited_count=$(docker ps -aq --filter status=exited | wc -l)
if [ $exited_count -gt 10 ]; then
    echo "  Consider removing $exited_count exited containers: docker container prune"
fi

unused_images=$(docker images -f "dangling=true" -q | wc -l)
if [ $unused_images -gt 5 ]; then
    echo "  Consider removing $unused_images dangling images: docker image prune"
fi
```
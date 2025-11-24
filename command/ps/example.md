# Examples

## Basic Examples

### List Running Containers
```bash
# List only running containers (default)
docker ps

# List all containers (running and stopped)
docker ps -a

# List last 5 created containers
docker ps -n 5

# Show latest created container
docker ps -l

# Show only container IDs
docker ps -q

# Show all container IDs
docker ps -aq
```

### Container Information with Sizes
```bash
# Show containers with file sizes
docker ps -s

# Show all containers with sizes
docker ps -as

# Show running containers without truncated output
docker ps --no-trunc
```

## Filtering Examples

### Filter by Status
```bash
# Show only running containers
docker ps --filter status=running

# Show only stopped containers
docker ps --filter status=exited

# Show paused containers
docker ps --filter status=paused

# Show containers that have died
docker ps --filter status=dead

# Multiple status filters
docker ps --filter status=running --filter status=paused
```

### Filter by Name and ID
```bash
# Filter by exact name
docker ps --filter name=web-server

# Filter by name pattern
docker ps --filter name=web

# Filter by container ID
docker ps --filter id=1a2b3c4d5e6f

# Filter by partial ID
docker ps --filter id=1a2b
```

### Filter by Image
```bash
# Filter by exact image
docker ps --filter ancestor=nginx:alpine

# Filter by image name
docker ps --filter ancestor=nginx

# Filter by image ID
docker ps --filter ancestor=sha256:abcd1234
```

### Filter by Labels
```bash
# Filter by label key
docker ps --filter label=environment

# Filter by label key-value pair
docker ps --filter label=environment=production

# Filter by multiple labels
docker ps --filter label=app=web --filter label=version=1.0
```

### Filter by Network and Volume
```bash
# Filter by network
docker ps --filter network=bridge
docker ps --filter network=my-network

# Filter by volume
docker ps --filter volume=my-volume
docker ps --filter volume=/data
```

### Filter by Health Status
```bash
# Filter by health status
docker ps --filter health=healthy
docker ps --filter health=unhealthy
docker ps --filter health=starting

# Show containers without health checks
docker ps --filter health=none
```

### Time-based Filters
```bash
# Show containers created before a specific container
docker ps --filter before=web-server

# Show containers created after a specific container
docker ps --filter since=web-server

# Combine with other filters
docker ps --filter since=web-server --filter status=running
```

## Format Templates

### Custom Output Formats
```bash
# Show only names and status
docker ps --format "table {{.Names}}\t{{.Status}}"

# Show ID, name, and image
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Image}}"

# Show detailed information
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"

# Custom table headers
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.RunningFor}}\t{{.Status}}" \
    --format-header="NAME\tIMAGE\tAGE\tSTATUS"
```

### JSON Output Format
```bash
# JSON format for single line per container
docker ps --format "{{json .}}"

# Extract specific fields as JSON
docker ps --format '{{json .Names}}'
docker ps --format '{{json .Ports}}'

# Combine multiple fields
docker ps --format '{"name":"{{.Names}}","status":"{{.Status}}","image":"{{.Image}}"}'
```

### Template Variables
```bash
# Available template variables
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Command}}\t{{.CreatedAt}}\t{{.RunningFor}}\t{{.Ports}}\t{{.Status}}\t{{.Size}}\t{{.Names}}\t{{.Labels}}\t{{.Mounts}}\t{{.Networks}}"

# Time-related fields
docker ps --format "table {{.Names}}\t{{.CreatedAt}}\t{{.RunningFor}}"

# Network and port information
docker ps --format "table {{.Names}}\t{{.Ports}}\t{{.Networks}}"

# Resource information
docker ps --format "table {{.Names}}\t{{.Size}}\t{{.Labels}}"
```

## Advanced Examples

### Combine with Other Commands
```bash
# Stop all running containers
docker ps -q | xargs docker stop

# Remove all stopped containers
docker ps -aq --filter status=exited | xargs docker rm

# Get logs from all running containers
docker ps --format "{{.Names}}" | xargs -I {} sh -c 'echo "=== {} ===" && docker logs {} --tail 10'

# Inspect all containers
docker ps -aq | xargs docker inspect
```

### Monitoring and Statistics
```bash
# Monitor container count
watch 'docker ps | wc -l'

# Monitor specific containers
watch 'docker ps --filter label=environment=production'

# Check container uptime
docker ps --format "table {{.Names}}\t{{.RunningFor}}\t{{.Status}}"

# Monitor resource usage
watch 'docker ps -s --format "table {{.Names}}\t{{.Size}}"'
```

### Filtering Complex Scenarios
```bash
# Production containers that are running
docker ps --filter label=environment=production --filter status=running

# Web containers with port mappings
docker ps --filter ancestor=nginx --format "table {{.Names}}\t{{.Ports}}"

# Containers using specific volumes
docker ps -a --filter volume=data-volume --format "table {{.Names}}\t{{.Status}}\t{{.Mounts}}"

# Old containers (created more than a week ago)
docker ps -a --filter "since=$(date -d '1 week ago' '+%Y-%m-%d')"
```

## Common Use Cases

### System Health Check
```bash
#!/bin/bash
# Container health monitoring script
echo "=== Docker Container Health Report ==="
echo "Date: $(date)"
echo

echo "Running Containers:"
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
echo

echo "Container Count Summary:"
echo "  Total containers: $(docker ps -a -q | wc -l)"
echo "  Running: $(docker ps -q | wc -l)"
echo "  Stopped: $(docker ps -aq --filter status=exited | wc -l)"
echo "  Paused: $(docker ps -aq --filter status=paused | wc -l)"
echo

echo "Unhealthy Containers:"
docker ps --filter health=unhealthy --format "table {{.Names}}\t{{.Status}}"

echo "Resource Usage (Top 5 by size):"
docker ps -s --format "table {{.Names}}\t{{.Size}}" | head -6
```

### Environment-Specific Monitoring
```bash
#!/bin/bash
# Monitor containers by environment
for env in development staging production; do
    echo "=== $env Environment ==="
    count=$(docker ps --filter label=environment=$env -q | wc -l)
    echo "Running containers: $count"
    
    if [ $count -gt 0 ]; then
        docker ps --filter label=environment=$env \
            --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
    fi
    echo
done
```

### Port Mapping Overview
```bash
#!/bin/bash
# Generate port mapping report
echo "=== Docker Port Mapping Report ==="
echo "Date: $(date)"
echo

echo "Containers with Port Mappings:"
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}" | \
    grep -v "NAMES" | \
    awk '$3 != "" {print}'

echo
echo "Port Usage Summary:"
docker ps --format "{{.Ports}}" | \
    grep -o '[0-9]*->[0-9]*/tcp' | \
    sort | uniq -c | sort -nr
```

### Container Lifecycle Analysis
```bash
#!/bin/bash
# Analyze container lifecycle and patterns
echo "=== Container Lifecycle Analysis ==="

echo "Recently Created (last 24 hours):"
docker ps -a --filter "since=24h" --format "table {{.Names}}\t{{.CreatedAt}}\t{{.Status}}"

echo
echo "Long Running Containers (>7 days):"
docker ps --format "table {{.Names}}\t{{.RunningFor}}\t{{.Status}}" | \
    awk 'NR==1 || /[7-9][0-9]* days|[0-9][0-9][0-9]+ days|[1-9][0-9]* weeks|[1-9][0-9]* months/'

echo
echo "Frequently Restarted Containers:"
for container in $(docker ps -a --format "{{.Names}}"); do
    restarts=$(docker inspect --format='{{.RestartCount}}' $container 2>/dev/null)
    if [ "$restarts" -gt 5 ]; then
        echo "  $container: $restarts restarts"
    fi
done
```

### Resource Monitoring Dashboard
```bash
#!/bin/bash
# Simple resource monitoring dashboard
clear
while true; do
    echo "=== Docker Resource Dashboard ==="
    echo "Updated: $(date)"
    echo
    
    echo "System Overview:"
    echo "  CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)% used"
    echo "  Memory: $(free -h | awk 'NR==2{printf "%s/%s (%.1f%%)\n", $3,$2,$3*100/$2 }')"
    echo "  Disk: $(df -h / | awk 'NR==2{printf "%s/%s (%s used)\n", $3,$2,$5}')"
    echo
    
    echo "Container Summary:"
    echo "  Running: $(docker ps -q | wc -l)"
    echo "  Total: $(docker ps -aq | wc -l)"
    echo
    
    echo "Active Containers:"
    docker ps --format "table {{.Names}}\t{{.Image}}\t{{.RunningFor}}\t{{.Size}}" -s
    
    echo
    echo "Press Ctrl+C to exit"
    sleep 30
    clear
done
```

### Automated Container Cleanup
```bash
#!/bin/bash
# Automated cleanup of old containers
echo "=== Container Cleanup Script ==="
echo "Date: $(date)"

# Find containers exited more than 7 days ago
old_containers=$(docker ps -aq --filter status=exited --filter "until=168h")

if [ -n "$old_containers" ]; then
    echo "Found old exited containers:"
    docker ps -a --filter status=exited --filter "until=168h" \
        --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.CreatedAt}}"
    
    echo
    read -p "Remove these containers? (y/N): " confirm
    
    if [ "$confirm" = "y" ] || [ "$confirm" = "Y" ]; then
        echo "Removing old containers..."
        echo $old_containers | xargs docker rm
        echo "Cleanup completed"
    else
        echo "Cleanup cancelled"
    fi
else
    echo "No old containers found for cleanup"
fi

# Show current status
echo
echo "Current container status:"
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.CreatedAt}}"
```

## Automation and Scripting

### Container Discovery Script
```bash
#!/bin/bash
# Discover and categorize containers
echo "=== Container Discovery ==="

# Group containers by image
echo "Containers by Image:"
docker ps -a --format "{{.Image}}" | sort | uniq -c | sort -nr

echo
echo "Containers by Status:"
docker ps -a --format "{{.Status}}" | sed 's/^Up.*/Running/' | \
    sed 's/^Exited.*/Stopped/' | sort | uniq -c

echo
echo "Containers by Network:"
for network in $(docker network ls --format "{{.Name}}"); do
    count=$(docker ps --filter network=$network -q | wc -l)
    if [ $count -gt 0 ]; then
        echo "  $network: $count containers"
    fi
done
```

### Performance Monitoring
```bash
#!/bin/bash
# Monitor container performance metrics
echo "=== Container Performance Monitoring ==="

# Memory usage by container
echo "Memory Usage (containers with size data):"
docker ps -s --format "table {{.Names}}\t{{.Size}}" | \
    grep -v "NAMES" | sort -k2 -hr | head -10

echo
echo "Port Utilization:"
docker ps --format "{{.Names}} {{.Ports}}" | \
    grep -v "NAMES" | \
    awk '$2 != "" {ports[$1] = $2} END {for (c in ports) print c ": " ports[c]}'

echo
echo "Container Density by Image:"
docker ps --format "{{.Image}}" | sort | uniq -c | \
    awk '{printf "%-30s %d containers\n", $2, $1}' | sort -k3 -nr
```
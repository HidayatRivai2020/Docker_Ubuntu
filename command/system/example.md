# Examples

## Basic Examples

### docker system df
```bash
# Show disk usage summary
docker system df

# Verbose output with details
docker system df -v

# Format output as table
docker system df --format "table {{.Type}}\t{{.TotalCount}}\t{{.Size}}"

# Show only specific resource type
docker system df | grep Images
```

### docker system prune
```bash
# Remove all unused data (except volumes)
docker system prune

# Remove all unused data including volumes
docker system prune --volumes

# Remove all unused data without confirmation
docker system prune -f

# Remove all unused data including all images
docker system prune -a

# Combine options for aggressive cleanup
docker system prune -a --volumes -f
```

### docker system info
```bash
# Display full system information
docker system info

# Format as JSON
docker system info --format json

# Extract specific information
docker system info --format '{{.ServerVersion}}'

# Get Docker root directory
docker system info --format '{{.DockerRootDir}}'
```

### docker system events
```bash
# Watch all events in real-time
docker system events

# Filter events by type
docker system events --filter 'type=container'

# Watch events for specific container
docker system events --filter 'container=myapp'

# Show events from last hour
docker system events --since '1h'
```

## Disk Usage Monitoring

### Check Available Space
```bash
# View total disk usage
docker system df

# Check specific resource types
docker system df -v | grep Images
docker system df -v | grep Containers
docker system df -v | grep Volumes

# Get reclaimable space
docker system df --format "{{.Reclaimable}}"
```

### Detailed Resource Analysis
```bash
# Show all image details
docker system df -v | grep -A 100 "Images:"

# Show all container details
docker system df -v | grep -A 100 "Containers:"

# Show all volume details
docker system df -v | grep -A 100 "Local Volumes:"

# Export disk usage report
docker system df -v > disk-usage-$(date +%Y%m%d).txt
```

## Cleanup Operations

### Safe Cleanup
```bash
# Remove only stopped containers
docker container prune

# Remove only dangling images
docker image prune

# Remove only unused networks
docker network prune

# Remove only unused volumes (careful!)
docker volume prune
```

### Filtered Cleanup
```bash
# Remove resources older than 24 hours
docker system prune --filter "until=24h"

# Remove resources older than 7 days
docker system prune --filter "until=168h"

# Remove resources with specific label
docker system prune --filter "label=environment=test"

# Remove resources without specific label
docker system prune --filter "label!=production"

# Combine multiple filters
docker system prune --filter "until=72h" --filter "label!=keep"
```

### Targeted Cleanup by Type
```bash
# Clean only build cache
docker builder prune

# Clean build cache older than 7 days
docker builder prune --filter "until=168h"

# Remove all unused images (aggressive)
docker image prune -a

# Remove dangling volumes
docker volume prune

# Clean everything except volumes
docker system prune -a -f
```

## System Information Extraction

### Docker Version and Config
```bash
# Get Docker version
docker system info --format '{{.ServerVersion}}'

# Get API version
docker system info --format '{{.ClientAPIVersion}}'

# Get storage driver
docker system info --format '{{.Driver}}'

# Get logging driver
docker system info --format '{{.LoggingDriver}}'

# Get cgroup driver
docker system info --format '{{.CgroupDriver}}'
```

### Resource Information
```bash
# Get total containers
docker system info --format '{{.Containers}}'

# Get running containers
docker system info --format '{{.ContainersRunning}}'

# Get stopped containers
docker system info --format '{{.ContainersStopped}}'

# Get paused containers
docker system info --format '{{.ContainersPaused}}'

# Get total images
docker system info --format '{{.Images}}'
```

### System Resources
```bash
# Get CPU count
docker system info --format '{{.NCPU}}'

# Get total memory
docker system info --format '{{.MemTotal}}'

# Get operating system
docker system info --format '{{.OperatingSystem}}'

# Get kernel version
docker system info --format '{{.KernelVersion}}'

# Get architecture
docker system info --format '{{.Architecture}}'
```

### Advanced Info Queries
```bash
# Check if Swarm is active
docker system info --format '{{.Swarm.LocalNodeState}}'

# Get Docker root directory
docker system info --format '{{.DockerRootDir}}'

# Get registry address
docker system info --format '{{.IndexServerAddress}}'

# Get security options
docker system info --format '{{.SecurityOptions}}'

# Get available plugins
docker system info --format '{{.Plugins}}'
```

## Event Monitoring

### Real-Time Monitoring
```bash
# Watch all Docker events
docker system events

# Watch with timestamps
docker system events --format '{{.Time}} {{.Action}} {{.Type}}'

# Watch events as JSON
docker system events --format '{{json .}}'

# Pretty print JSON events
docker system events --format '{{json .}}' | jq
```

### Container Events
```bash
# Watch container lifecycle events
docker system events --filter 'type=container'

# Watch container start events
docker system events --filter 'type=container' --filter 'event=start'

# Watch container stop events
docker system events --filter 'type=container' --filter 'event=stop'

# Watch events for specific container
docker system events --filter 'container=nginx'

# Watch create and destroy events
docker system events --filter 'event=create' --filter 'event=destroy'
```

### Image Events
```bash
# Watch image events
docker system events --filter 'type=image'

# Watch image pull events
docker system events --filter 'type=image' --filter 'event=pull'

# Watch image push events
docker system events --filter 'type=image' --filter 'event=push'

# Watch events for specific image
docker system events --filter 'image=nginx:latest'

# Watch image deletion events
docker system events --filter 'type=image' --filter 'event=delete'
```

### Network and Volume Events
```bash
# Watch network events
docker system events --filter 'type=network'

# Watch volume events
docker system events --filter 'type=volume'

# Watch volume mount events
docker system events --filter 'type=volume' --filter 'event=mount'

# Watch network create/destroy
docker system events --filter 'type=network' --filter 'event=create'
```

### Time-Based Event Queries
```bash
# Show events from last hour
docker system events --since '1h'

# Show events from last 24 hours
docker system events --since '24h'

# Show events in specific time range
docker system events --since '2024-11-24T00:00:00' --until '2024-11-24T23:59:59'

# Show events between relative times
docker system events --since '2h' --until '1h'

# Show events from specific timestamp
docker system events --since '2024-11-24T12:00:00'
```

## Common Use Cases

### Automated Maintenance
```bash
# Daily cleanup script
#!/bin/bash
echo "Running Docker cleanup..."
docker system df
docker system prune -f --filter "until=168h"
docker system df
echo "Cleanup complete"

# Aggressive cleanup (use with caution)
docker system prune -a --volumes -f

# Cleanup with backup
docker save $(docker images -q) -o backup-$(date +%Y%m%d).tar
docker system prune -a -f
```

### Monitoring Scripts
```bash
# Check disk usage and alert
#!/bin/bash
USAGE=$(docker system df --format "{{.Type}}\t{{.Size}}" | grep Images | awk '{print $2}' | sed 's/GB//')
if (( $(echo "$USAGE > 50" | bc -l) )); then
    echo "WARNING: Docker images using ${USAGE}GB"
    docker system prune -f
fi

# Monitor container count
RUNNING=$(docker system info --format '{{.ContainersRunning}}')
echo "Running containers: $RUNNING"
```

### Development Workflows
```bash
# Clean development environment
docker system prune -a -f --volumes
docker system df

# Reset Docker environment
docker stop $(docker ps -aq)
docker system prune -a --volumes -f

# Check system health
docker system info
docker system df
docker ps -a
```

### CI/CD Integration
```bash
# Pre-build cleanup
docker system prune -f --filter "until=24h"

# Post-build cleanup
docker image prune -f
docker builder prune -f

# Generate system report
docker system df > build-report.txt
docker system info >> build-report.txt
```

## Monitoring and Logging

### Event Logging
```bash
# Log all events to file
docker system events > docker-events.log

# Log events with timestamps
docker system events --format '{{.Time}} {{.Action}} {{.Type}}' > events-$(date +%Y%m%d).log

# Log specific event types
docker system events --filter 'type=container' >> container-events.log

# Log events with rotation
docker system events | while read event; do
    echo "$(date): $event" >> docker-events-$(date +%Y%m%d).log
done
```

### System Health Monitoring
```bash
# Create health check script
#!/bin/bash
{
    echo "=== Docker System Health Report ==="
    echo "Date: $(date)"
    echo ""
    echo "=== Disk Usage ==="
    docker system df
    echo ""
    echo "=== System Info ==="
    docker system info
    echo ""
    echo "=== Running Containers ==="
    docker ps
} > health-report-$(date +%Y%m%d-%H%M%S).txt

# Monitor disk usage continuously
watch -n 60 'docker system df'
```

### Performance Monitoring
```bash
# Track resource usage over time
while true; do
    echo "$(date),$(docker system df --format '{{.Size}}' | sed -n '2p')" >> images-size.csv
    sleep 3600
done

# Monitor system events in background
docker system events --format '{{.Time}},{{.Type}},{{.Action}}' >> events.csv &

# Generate daily report
docker system df -v > daily-report-$(date +%Y%m%d).txt
```

## Troubleshooting

### Disk Space Issues
```bash
# Identify large images
docker system df -v | grep -A 100 "Images:" | sort -k7 -h

# Find reclaimable space
docker system df | grep -v "TYPE" | awk '{print $4}'

# Emergency cleanup
docker system prune -a --volumes -f

# Check after cleanup
docker system df
```

### System Diagnostics
```bash
# Full system check
docker system info

# Check for errors in events
docker system events --since '1h' --filter 'type=container'

# Verify Docker daemon
docker system info --format '{{.ServerVersion}}'

# Check storage driver
docker system info --format '{{.Driver}}'
```

### Performance Analysis
```bash
# Monitor build cache
docker system df | grep "Build Cache"

# Check container overhead
docker system df -v | grep -A 100 "Containers:"

# Analyze volume usage
docker system df -v | grep -A 100 "Local Volumes:"

# Export analysis report
docker system df -v > analysis-$(date +%Y%m%d).txt
```

## Security Considerations

### Audit and Compliance
```bash
# Monitor security events
docker system events --filter 'type=container' --format '{{.Time}} {{.Action}} {{.Actor.Attributes.name}}'

# Track image pulls
docker system events --filter 'type=image' --filter 'event=pull'

# Monitor privilege escalation
docker system events --filter 'event=exec_create'

# Export security audit log
docker system events --since '24h' > security-audit-$(date +%Y%m%d).log
```

### Safe Cleanup Practices
```bash
# Never prune production resources
docker system prune --filter "label!=production"

# Keep recent resources
docker system prune --filter "until=168h"

# Dry run simulation (check what would be removed)
docker system prune --filter "until=24h" --dry-run 2>&1 | grep "Total reclaimed"

# Backup before aggressive cleanup
docker save $(docker images --filter "label=important" -q) -o backup.tar
docker system prune -a --volumes -f
```

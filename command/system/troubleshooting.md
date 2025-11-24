# Troubleshooting

## Common Issues

### High Disk Usage
**Problem**: Docker consuming too much disk space
```bash
# Check disk usage breakdown
docker system df -v

# Identify what's using space
df -h /var/lib/docker

# Check reclaimable space
docker system df | grep -v TYPE | awk '{print $4}'

# Clean up safely
docker system prune -f

# Aggressive cleanup (use with caution)
docker system prune -a --volumes -f

# Check after cleanup
docker system df
```

### Cannot Remove Resources
**Problem**: Error when trying to remove images, containers, or volumes
```bash
# Check what's using the resource
docker ps -a --filter "ancestor=<image_name>"

# Stop all containers using the image
docker stop $(docker ps -a -q --filter "ancestor=<image_name>")

# Force remove container
docker rm -f <container_id>

# Force remove image
docker rmi -f <image_id>

# Remove volume (check dependencies first!)
docker volume rm <volume_name>

# Check for system errors
sudo journalctl -u docker.service --since "10 minutes ago"
```

### Prune Removed Important Resources
**Problem**: Accidentally removed production resources with prune
```bash
# Prevention: Always use filters
docker system prune --filter "label!=production"
docker system prune --filter "until=168h"

# If already removed, restore from backup
docker load -i backup.tar

# Check remaining images
docker images

# Rebuild if necessary
docker build -t <image_name> .

# Recovery: Pull from registry
docker pull <registry>/<image_name>:<tag>
```

### System Info Not Showing Correctly
**Problem**: Docker system info returns incomplete or incorrect information
```bash
# Restart Docker daemon
sudo systemctl restart docker

# Check daemon status
sudo systemctl status docker

# Verify Docker is running
docker version

# Check daemon logs
sudo journalctl -u docker.service --no-pager | tail -50

# Reload daemon configuration
sudo systemctl daemon-reload
sudo systemctl restart docker

# Verify with basic command
docker info --format '{{.ServerVersion}}'
```

### Events Not Displaying
**Problem**: docker system events not showing any output
```bash
# Check if Docker daemon is running
sudo systemctl status docker

# Verify Docker socket permissions
ls -l /var/run/docker.sock

# Test with basic filter
docker system events --filter 'type=container' --since '1m'

# Generate test event
docker run --rm alpine echo "test"

# Check daemon logs for errors
sudo journalctl -u docker.service --since "5 minutes ago"

# Verify user permissions
groups | grep docker
```

### Disk Full After Prune
**Problem**: Disk still full even after running prune
```bash
# Check what's actually using space
docker system df -v

# Check Docker data directory
du -sh /var/lib/docker/*
sudo du -sh /var/lib/docker/overlay2/* | sort -h | tail -20

# Clean build cache specifically
docker builder prune -a -f

# Remove unused volumes (careful!)
docker volume prune -f

# Check system-level disk usage
df -h
sudo du -sh /var/* | sort -h | tail -10

# Consider moving Docker data directory
# Edit /etc/docker/daemon.json:
# {"data-root": "/new/path/docker"}
```

### Slow System Commands
**Problem**: docker system commands taking too long to execute
```bash
# Check Docker daemon load
docker info | grep "Containers"

# Check system resources
top
free -h
iostat

# Stop unnecessary containers
docker stop $(docker ps -q)

# Prune to reduce load
docker system prune -f

# Check for daemon issues
sudo journalctl -u docker.service --since "1 hour ago" | grep error

# Restart daemon if needed
sudo systemctl restart docker

# Check daemon settings
cat /etc/docker/daemon.json
```

## Debugging Commands

### Disk Usage Analysis
```bash
# Detailed disk usage
docker system df -v

# Sort images by size
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -k3 -h

# Find large containers
docker ps -a --format "table {{.Names}}\t{{.Size}}"

# Check volume sizes
docker volume ls -q | xargs docker volume inspect --format '{{.Name}}: {{.Mountpoint}}' | while read vol; do
    sudo du -sh $(echo $vol | cut -d: -f2)
done

# Analyze build cache
docker builder du

# Export disk usage report
docker system df -v > disk-analysis-$(date +%Y%m%d).txt
```

### System Information Queries
```bash
# Check Docker version
docker system info --format '{{.ServerVersion}}'

# Check storage driver
docker system info --format '{{.Driver}}'

# Check daemon configuration
docker system info --format '{{json .}}' | jq .

# Verify running state
docker system info --format '{{.ContainersRunning}} running / {{.Containers}} total'

# Check for warnings
docker system info 2>&1 | grep -i warning

# Export system info
docker system info > system-info-$(date +%Y%m%d).txt
```

### Event Monitoring and Logging
```bash
# Monitor events in real-time
docker system events

# Filter specific event types
docker system events --filter 'type=container' --filter 'event=start'

# Log events to file
docker system events --format '{{.Time}} {{.Type}} {{.Action}}' >> docker-events.log

# Check recent container events
docker system events --since '1h' --filter 'type=container'

# Monitor for errors
docker system events --filter 'type=container' | grep -i error

# Debug specific container
docker system events --filter "container=<container_name>"
```

### Resource Cleanup Verification
```bash
# Before cleanup snapshot
docker system df > before-cleanup.txt

# Perform cleanup
docker system prune -f

# After cleanup snapshot
docker system df > after-cleanup.txt

# Compare results
diff before-cleanup.txt after-cleanup.txt

# Verify reclaimable space reduced
docker system df | grep -v TYPE | awk '{print "Before:", $4}'

# Check specific resource types
docker images --filter "dangling=true"
docker ps -a --filter "status=exited"
docker volume ls --filter "dangling=true"
```

### Performance Diagnostics
```bash
# Monitor system resources during operation
docker system df & docker stats --no-stream

# Check daemon resource usage
ps aux | grep dockerd

# Monitor I/O
iostat -x 2 5

# Check for bottlenecks
vmstat 2 5

# Analyze slow commands
time docker system df -v
time docker system info

# Check overlay2 usage
sudo df -h /var/lib/docker/overlay2
```

## Error Messages and Solutions

### "No space left on device"
```bash
# Check available space
df -h /var/lib/docker

# Emergency cleanup
docker system prune -a --volumes -f

# Remove specific large images
docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}" | sort -k2 -h | tail -5
docker rmi <large_image>

# Clean build cache
docker builder prune -a -f

# Check for log files
sudo du -sh /var/lib/docker/containers/*/
sudo truncate -s 0 /var/lib/docker/containers/*/*.log
```

### "Cannot connect to Docker daemon"
```bash
# Check if daemon is running
sudo systemctl status docker

# Start daemon if stopped
sudo systemctl start docker

# Check socket permissions
ls -l /var/run/docker.sock
sudo chmod 666 /var/run/docker.sock

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify daemon is accessible
docker version
```

### "Error response from daemon: conflict"
```bash
# Resource name already in use
docker ps -a | grep <name>
docker rm <conflicting_container>

# Port already allocated
docker ps --filter "publish=8080"
docker stop <container_using_port>

# Network already exists
docker network ls | grep <network_name>
docker network rm <network_name>

# Volume already in use
docker ps -a --filter "volume=<volume_name>"
docker rm <container_using_volume>
```

### "permission denied" during prune
```bash
# Check user permissions
groups | grep docker

# Run with sudo (temporary)
sudo docker system prune -f

# Fix permissions permanently
sudo usermod -aG docker $USER
newgrp docker

# Verify permissions
docker system df
```

### "context deadline exceeded"
```bash
# Docker daemon timeout
sudo systemctl restart docker

# Increase timeout in client config
# ~/.docker/config.json
# Add: "timeout": 300

# Check network connectivity
ping 8.8.8.8

# Check daemon logs
sudo journalctl -u docker.service --since "5 minutes ago"

# Kill stuck processes
sudo pkill -9 dockerd
sudo systemctl restart docker
```

## Recovery Procedures

### Recover from Failed Cleanup
```bash
# Step 1: Check current state
docker system df -v

# Step 2: Verify what's missing
docker images
docker ps -a
docker volume ls

# Step 3: Restore from backup if available
docker load -i backup.tar

# Step 4: Pull missing images
docker pull <missing_image>

# Step 5: Recreate containers
docker run -d --name <container> <image>

# Step 6: Verify recovery
docker ps
docker system df
```

### Fix Corrupted Docker Data
```bash
# Backup current state
tar -czf docker-backup-$(date +%Y%m%d).tar.gz /var/lib/docker/volumes

# Stop Docker daemon
sudo systemctl stop docker

# Check filesystem
sudo fsck /dev/<docker_partition>

# Clear temporary files
sudo rm -rf /var/lib/docker/tmp/*

# Restart daemon
sudo systemctl start docker

# Verify integrity
docker system info
docker images
docker ps -a
```

### Reset Docker System
```bash
# WARNING: This removes ALL Docker data
# Backup first!
docker save $(docker images -q) -o full-backup.tar

# Stop all containers
docker stop $(docker ps -aq)

# Remove all containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Remove all volumes
docker volume rm $(docker volume ls -q)

# Remove all networks (except defaults)
docker network rm $(docker network ls -q)

# Prune everything
docker system prune -a --volumes -f

# Verify clean state
docker system df
```

### Automated System Health Check
```bash
#!/bin/bash
# Docker system health check script

echo "=== Docker System Health Check ==="
echo "Timestamp: $(date)"
echo

# Check daemon status
echo "=== Daemon Status ==="
systemctl is-active docker
echo

# Check disk usage
echo "=== Disk Usage ==="
docker system df
echo

# Check for warnings
echo "=== System Warnings ==="
docker system info 2>&1 | grep -i warning || echo "No warnings"
echo

# Check resource limits
echo "=== Resource Status ==="
echo "Running containers: $(docker ps -q | wc -l)"
echo "Total containers: $(docker ps -aq | wc -l)"
echo "Images: $(docker images -q | wc -l)"
echo "Volumes: $(docker volume ls -q | wc -l)"
echo

# Check reclaimable space
echo "=== Reclaimable Space ==="
docker system df | grep -v TYPE | awk '{print $1 ": " $4}'
echo

# Recommend cleanup if needed
RECLAIMABLE=$(docker system df --format "{{.Reclaimable}}" | grep "GB" | head -1 | sed 's/GB.*//' | cut -d'.' -f1)
if [ ! -z "$RECLAIMABLE" ] && [ "$RECLAIMABLE" -gt 10 ]; then
    echo "WARNING: Over ${RECLAIMABLE}GB reclaimable space detected"
    echo "Recommendation: Run 'docker system prune -f'"
fi
```

### Monitoring Script for Alerts
```bash
#!/bin/bash
# Docker system monitoring with alerts

THRESHOLD_GB=50
EMAIL="admin@example.com"

# Check disk usage
USAGE=$(docker system df --format "{{.Size}}" | sed -n '2p' | sed 's/GB//' | cut -d'.' -f1)

if [ "$USAGE" -gt "$THRESHOLD_GB" ]; then
    echo "ALERT: Docker images using ${USAGE}GB (threshold: ${THRESHOLD_GB}GB)" | \
        mail -s "Docker Disk Usage Alert" $EMAIL
    
    # Auto-cleanup old resources
    docker system prune -f --filter "until=168h"
    
    # Log the action
    echo "$(date): Auto-cleanup performed. Usage was ${USAGE}GB" >> /var/log/docker-cleanup.log
fi

# Check daemon health
if ! systemctl is-active --quiet docker; then
    echo "ALERT: Docker daemon is not running" | \
        mail -s "Docker Daemon Alert" $EMAIL
    
    systemctl start docker
fi

# Check for errors in recent events
ERRORS=$(docker system events --since '5m' --until '0m' 2>&1 | grep -i error | wc -l)
if [ "$ERRORS" -gt 0 ]; then
    echo "ALERT: ${ERRORS} errors detected in Docker events" | \
        mail -s "Docker Error Alert" $EMAIL
fi
```

## Best Practices for Prevention

### Regular Maintenance
```bash
# Daily cleanup (add to crontab)
0 2 * * * /usr/bin/docker system prune -f --filter "until=168h"

# Weekly aggressive cleanup
0 3 * * 0 /usr/bin/docker system prune -a -f --filter "until=336h"

# Monthly volume cleanup (careful!)
0 4 1 * * /usr/bin/docker volume prune -f --filter "label!=important"

# Continuous monitoring
*/15 * * * * /usr/local/bin/docker-health-check.sh
```

### Preventive Measures
```bash
# Set up disk usage alerts
# Add to /etc/docker/daemon.json:
{
  "storage-driver": "overlay2",
  "data-root": "/mnt/docker-data",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "builder": {
    "gc": {
      "enabled": true,
      "policy": [
        {"keepStorage": "20GB", "all": true}
      ]
    }
  }
}

# Reload daemon
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### Monitoring Setup
```bash
# Install monitoring tools
apt-get install sysstat

# Enable resource monitoring
systemctl enable sysstat
systemctl start sysstat

# Create custom dashboard script
cat > /usr/local/bin/docker-dashboard.sh <<'EOF'
#!/bin/bash
watch -n 5 'echo "=== Docker System Status ===" && \
docker system df && echo && \
echo "=== Running Containers ===" && \
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Size}}" && echo && \
echo "=== System Resources ===" && \
free -h && echo && \
df -h /var/lib/docker'
EOF

chmod +x /usr/local/bin/docker-dashboard.sh
```

## See Also
- [example.md](example.md) - System command examples
- [README.md](README.md) - System command documentation
- [images](../images/README.md) - Image management
- [ps](../ps/README.md) - Container management

# Troubleshooting

## Common Issues

### Image Not Found
**Problem**: Docker cannot find the specified image
```bash
# Check if image exists locally
docker images | grep <image_name>

# List all images
docker images

# Pull image from registry
docker pull <image_name>:<tag>

# Check image with full name
docker history <registry>/<image_name>:<tag>

# Verify image ID
docker images --no-trunc | grep <image_name>
```

### Truncated Output
**Problem**: Command output is cut off or abbreviated
```bash
# View full command text
docker history --no-trunc <image_name>

# Export to file for easier viewing
docker history --no-trunc <image_name> > history.txt

# Use format to show specific fields
docker history --format "{{.CreatedBy}}" --no-trunc <image_name>

# Pipe to less for scrolling
docker history --no-trunc <image_name> | less
```

### Missing Layer Information
**Problem**: Some layers show as `<missing>`
```bash
# This is normal for layers from parent images
# <missing> indicates layers pulled from registry, not built locally

# To see all layers including parent image
docker history <image_name>

# Compare with base image
docker history ubuntu:latest  # Check base image layers

# Verify image integrity
docker inspect <image_name> --format='{{.RootFS.Layers}}'

# Pull image again if suspected corruption
docker pull <image_name>
```

### Size Calculation Discrepancy
**Problem**: Layer sizes don't match total image size
```bash
# Check actual image size
docker images <image_name>

# View individual layer sizes
docker history <image_name> --format "table {{.Size}}\t{{.CreatedBy}}"

# Use docker system df for accurate sizes
docker system df -v | grep <image_name>

# Inspect image details
docker inspect <image_name> --format='{{.Size}}'

# Note: Some layers share space (deduplicated)
# Virtual size includes shared layers
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.VirtualSize}}"
```

### Permission Denied
**Problem**: Cannot access Docker socket or commands
```bash
# Check Docker permissions
ls -la /var/run/docker.sock

# Add user to docker group
sudo usermod -aG docker $USER

# Logout and login again, or use
newgrp docker

# Temporarily use sudo
sudo docker history <image_name>

# Check Docker service status
sudo systemctl status docker
```

### Format Errors
**Problem**: Invalid format string or unexpected output
```bash
# Use valid format placeholders
docker history --format "{{.ID}}\t{{.CreatedBy}}\t{{.Size}}" <image_name>

# Valid placeholders:
# {{.ID}}, {{.CreatedBy}}, {{.CreatedAt}}, {{.CreatedSince}}, {{.Size}}, {{.Comment}}

# Use table format for headers
docker history --format "table {{.ID}}\t{{.Size}}" <image_name>

# Use json format for structured output
docker history --format json <image_name>

# Escape special characters in format strings
docker history --format '{{.CreatedBy}}' <image_name>
```

### Command Hangs or Slow Response
**Problem**: Command takes too long to execute
```bash
# Check Docker daemon status
sudo systemctl status docker

# Check system resources
docker system df
df -h /var/lib/docker

# Restart Docker daemon if needed
sudo systemctl restart docker

# Clean up unused data
docker system prune

# Check for large images
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -k3 -h
```

## Debugging Commands

### Basic Inspection
```bash
# View complete history
docker history <image_name>

# View with full command text
docker history --no-trunc <image_name>

# View only image IDs
docker history -q <image_name>

# View with human-readable sizes
docker history -H <image_name>

# Check if image exists
docker images | grep <image_name>
```

### Detailed Analysis
```bash
# Count total layers
docker history <image_name> | wc -l

# Find large layers
docker history <image_name> | grep MB | awk '$5+0 > 50 {print $0}'

# Check layer creation times
docker history --format "{{.CreatedAt}}\t{{.Size}}" <image_name>

# Export full history
docker history --no-trunc <image_name> > history-$(date +%Y%m%d).txt

# Compare with image inspection
docker inspect <image_name> --format='{{.RootFS.Layers}}'
```

### Format Debugging
```bash
# Test different format options
docker history --format "{{.ID}}" <image_name>
docker history --format "{{.CreatedBy}}" <image_name>
docker history --format "{{.Size}}" <image_name>

# Use table format with multiple columns
docker history --format "table {{.ID}}\t{{.Size}}\t{{.CreatedSince}}" <image_name>

# Output as JSON for parsing
docker history --format json <image_name> | jq '.'

# Custom format with labels
docker history --format "ID: {{.ID}}, Size: {{.Size}}" <image_name>
```

### Image Verification
```bash
# Check image details
docker inspect <image_name>

# Verify image layers
docker inspect <image_name> --format='{{json .RootFS.Layers}}' | jq

# Check image configuration
docker inspect <image_name> --format='{{json .Config}}' | jq

# Verify image size
docker images <image_name> --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Check image creation date
docker inspect <image_name> --format='{{.Created}}'
```

### Layer Investigation
```bash
# Find base image layers
docker history <image_name> | tail -10

# Identify RUN commands
docker history --no-trunc <image_name> | grep "RUN"

# Find COPY/ADD commands
docker history --no-trunc <image_name> | grep -E "COPY|ADD"

# Check ENV variables
docker history --no-trunc <image_name> | grep "ENV"

# Find CMD/ENTRYPOINT
docker history --no-trunc <image_name> | grep -E "CMD|ENTRYPOINT"
```

### Size Analysis
```bash
# Show only non-zero layers
docker history <image_name> | grep -v "0B"

# Calculate total size from MB layers
docker history <image_name> --format "{{.Size}}" | grep MB | \
    sed 's/MB//' | awk '{sum+=$1} END {print sum " MB"}'

# Find largest layer
docker history <image_name> --format "table {{.Size}}\t{{.CreatedBy}}" | \
    grep MB | sort -rh | head -1

# Compare with docker images size
echo "History layers:" && docker history <image_name> | grep MB
echo "Image size:" && docker images <image_name> --format "{{.Size}}"
```

## Exit Codes and Meanings

### Standard Exit Codes
- **0** - Successful execution
- **1** - General error (image not found, invalid format, etc.)
- **125** - Docker daemon error
- **126** - Command not executable
- **127** - Command not found

### Checking Exit Codes
```bash
# Run command and check exit code
docker history <image_name>
echo $?

# Check for specific errors
docker history nonexistent:image 2>&1
if [ $? -ne 0 ]; then
    echo "Command failed"
fi

# Capture error output
ERROR=$(docker history badimage 2>&1)
if [ $? -ne 0 ]; then
    echo "Error: $ERROR"
fi
```

## Recovery Procedures

### Image Not Available
```bash
# Step 1: Check local images
docker images

# Step 2: Search for image
docker search <image_name>

# Step 3: Pull from registry
docker pull <image_name>:<tag>

# Step 4: Verify pull was successful
docker images | grep <image_name>

# Step 5: View history
docker history <image_name>:<tag>
```

### Corrupted Image Data
```bash
# Check image integrity
docker inspect <image_name>

# Try pulling fresh copy
docker rmi <image_name>
docker pull <image_name>

# Verify layers
docker inspect <image_name> --format='{{.RootFS.Layers}}' | jq

# Check Docker storage
docker system df

# Prune if needed
docker system prune
```

### Output Parsing Issues
```bash
# Use JSON format for reliable parsing
docker history --format json <image_name> > history.json

# Parse with jq
cat history.json | jq '.[] | {size: .Size, command: .CreatedBy}'

# Use table format for human reading
docker history --format "table {{.ID}}\t{{.Size}}\t{{.CreatedSince}}" <image_name>

# Export to CSV
docker history --format "{{.ID}},{{.Size}},{{.CreatedAt}}" <image_name> > history.csv
```

### Automated Debugging Script
```bash
#!/bin/bash
# Comprehensive history debugging script

IMAGE="$1"

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image_name>"
    exit 1
fi

echo "=== Docker History Debug Report ==="
echo "Image: $IMAGE"
echo "Timestamp: $(date)"
echo

echo "=== Checking Image Existence ==="
if docker images | grep -q "$(echo $IMAGE | cut -d: -f1)"; then
    echo "✓ Image found locally"
    docker images | grep "$(echo $IMAGE | cut -d: -f1)"
else
    echo "✗ Image not found locally"
    exit 1
fi
echo

echo "=== Basic History ==="
docker history "$IMAGE" 2>&1
if [ $? -eq 0 ]; then
    echo "✓ History retrieved successfully"
else
    echo "✗ Failed to retrieve history"
    exit 1
fi
echo

echo "=== Layer Count ==="
LAYERS=$(docker history "$IMAGE" 2>/dev/null | tail -n +2 | wc -l)
echo "Total layers: $LAYERS"
echo

echo "=== Size Analysis ==="
echo "Image size:"
docker images "$IMAGE" --format "{{.Size}}"
echo
echo "Large layers (>10MB):"
docker history "$IMAGE" | grep MB | awk '$5+0 > 10 {print $0}'
echo

echo "=== Layer Breakdown ==="
echo "Zero-size layers (metadata): $(docker history "$IMAGE" | grep -c "0B")"
echo "Non-zero layers: $(docker history "$IMAGE" | grep -cv "0B")"
echo "Missing layers: $(docker history "$IMAGE" | grep -c "missing")"
echo

echo "=== Full Command History ==="
docker history --no-trunc "$IMAGE" > "history-$IMAGE-$(date +%Y%m%d-%H%M%S).txt"
echo "Full history saved to history-$IMAGE-$(date +%Y%m%d-%H%M%S).txt"
```

### Size Investigation Script
```bash
#!/bin/bash
# Investigate image size issues

IMAGE="$1"

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image_name>"
    exit 1
fi

echo "=== Size Investigation for $IMAGE ==="
echo

echo "1. Image Size:"
docker images "$IMAGE" --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
echo

echo "2. Layer Count:"
docker history "$IMAGE" | tail -n +2 | wc -l
echo

echo "3. Top 5 Largest Layers:"
docker history --format "table {{.Size}}\t{{.CreatedBy}}" "$IMAGE" | \
    grep MB | sort -rh | head -5
echo

echo "4. Total Size from Layers:"
TOTAL=$(docker history "$IMAGE" --format "{{.Size}}" | grep MB | \
    sed 's/MB//' | awk '{sum+=$1} END {print sum}')
echo "${TOTAL} MB (Note: May differ from image size due to shared layers)"
echo

echo "5. Zero-size Layers:"
docker history "$IMAGE" | grep -c "0B"
echo

echo "6. Storage Driver Info:"
docker info | grep "Storage Driver"
echo

echo "7. Detailed Layer Information:"
docker inspect "$IMAGE" --format='{{json .RootFS.Layers}}' | jq
```

### Comparison Script
```bash
#!/bin/bash
# Compare history of two images

IMAGE1="$1"
IMAGE2="$2"

if [ -z "$IMAGE1" ] || [ -z "$IMAGE2" ]; then
    echo "Usage: $0 <image1> <image2>"
    exit 1
fi

echo "=== Comparing $IMAGE1 vs $IMAGE2 ==="
echo

echo "Layer Counts:"
echo "$IMAGE1: $(docker history "$IMAGE1" 2>/dev/null | tail -n +2 | wc -l) layers"
echo "$IMAGE2: $(docker history "$IMAGE2" 2>/dev/null | tail -n +2 | wc -l) layers"
echo

echo "Image Sizes:"
echo "$IMAGE1: $(docker images "$IMAGE1" --format "{{.Size}}")"
echo "$IMAGE2: $(docker images "$IMAGE2" --format "{{.Size}}")"
echo

echo "Exporting histories for comparison..."
docker history --no-trunc "$IMAGE1" > /tmp/history1.txt
docker history --no-trunc "$IMAGE2" > /tmp/history2.txt

echo "Differences:"
diff -u /tmp/history1.txt /tmp/history2.txt || true
echo

echo "Layer commands saved to:"
echo "  /tmp/history1.txt"
echo "  /tmp/history2.txt"
```

### Health Check Script
```bash
#!/bin/bash
# Check Docker and image health

echo "=== Docker Health Check ==="
echo

echo "1. Docker Version:"
docker version --format '{{.Server.Version}}'
echo

echo "2. Docker Daemon Status:"
if systemctl is-active --quiet docker; then
    echo "✓ Running"
else
    echo "✗ Not running"
    sudo systemctl start docker
fi
echo

echo "3. Disk Space:"
docker system df
echo

echo "4. Storage Driver:"
docker info | grep "Storage Driver"
echo

echo "5. Testing History Command:"
TEST_IMAGE="alpine:latest"
echo "Testing with $TEST_IMAGE..."
docker pull "$TEST_IMAGE" > /dev/null 2>&1
if docker history "$TEST_IMAGE" > /dev/null 2>&1; then
    echo "✓ History command working"
else
    echo "✗ History command failed"
fi
echo

echo "6. Recent Docker Logs:"
sudo journalctl -u docker.service --since "5 minutes ago" --no-pager | tail -20
```

## Best Practices for Troubleshooting

### Before Running History
```bash
# 1. Verify image exists
docker images | grep <image_name>

# 2. Check Docker daemon
systemctl status docker

# 3. Verify disk space
df -h /var/lib/docker

# 4. Pull image if needed
docker pull <image_name>:<tag>
```

### When History Fails
```bash
# 1. Check error message
docker history <image_name> 2>&1 | tee error.log

# 2. Verify image integrity
docker inspect <image_name>

# 3. Try with image ID instead
docker history <image_id>

# 4. Check Docker logs
sudo journalctl -u docker.service -n 50
```

### For Large Images
```bash
# 1. Use pagination
docker history <image_name> | head -20
docker history <image_name> | tail -20

# 2. Export to file
docker history --no-trunc <image_name> > history.txt

# 3. Use format to reduce output
docker history --format "{{.ID}}\t{{.Size}}" <image_name>

# 4. Filter specific information
docker history <image_name> | grep MB
```

### Regular Maintenance
```bash
# Clean up unused data
docker system prune -a

# Remove old images
docker image prune -a

# Check storage usage
docker system df -v

# Verify Docker health
docker info

# Update Docker
sudo apt-get update && sudo apt-get upgrade docker-ce
```

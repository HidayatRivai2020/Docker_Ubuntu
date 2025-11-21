# Docker images Examples

## Basic Examples

### List All Images
```bash
# List all local images
docker images

# Alternative syntax
docker image ls

# Show all images including intermediate layers
docker images -a
```

### Show Image IDs Only
```bash
# Show only image IDs (useful for scripting)
docker images -q

# Show all image IDs including intermediate layers
docker images -aq

# Count total images
docker images -q | wc -l
```

### Detailed Information
```bash
# Show full image IDs (not truncated)
docker images --no-trunc

# Show image digests
docker images --digests

# Show both full IDs and digests
docker images --no-trunc --digests
```

## Filtering Examples

### Filter by Repository
```bash
# Show only ubuntu images
docker images ubuntu

# Show specific tag
docker images ubuntu:20.04

# Show all tags for a repository
docker images nginx
```

### Filter by Conditions
```bash
# Show dangling images (untagged)
docker images --filter "dangling=true"

# Show images created before specific image
docker images --filter "before=ubuntu:20.04"

# Show images created after specific image
docker images --filter "since=ubuntu:18.04"

# Show images by label
docker images --filter "label=maintainer=nginx"

# Show images created in last 24 hours
docker images --filter "since=24h"
```

### Multiple Filters
```bash
# Combine multiple filters
docker images --filter "dangling=false" --filter "before=ubuntu:latest"

# Filter by reference pattern
docker images --filter "reference=*/ubuntu"
docker images --filter "reference=*:latest"
```

## Custom Formatting Examples

### Table Format
```bash
# Custom table with specific columns
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Include creation date
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.CreatedSince}}\t{{.Size}}"

# Show ID and repository
docker images --format "table {{.ID}}\t{{.Repository}}:{{.Tag}}"
```

### JSON Format
```bash
# Output as JSON
docker images --format "{{json .}}"

# Pretty print JSON with jq
docker images --format "{{json .}}" | jq .

# Extract specific fields with jq
docker images --format "{{json .}}" | jq -r '.Repository + ":" + .Tag'
```

### Custom Templates
```bash
# Show size in human readable format
docker images --format "{{.Repository}}:{{.Tag}} - {{.Size}}"

# Show creation time
docker images --format "{{.Repository}}:{{.Tag}} ({{.CreatedAt}})"

# Show virtual size
docker images --format "{{.Repository}}:{{.Tag}} [Virtual: {{.VirtualSize}}]"
```

## Advanced Examples

### Size Analysis
```bash
# Sort images by size (largest first)
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -k3 -hr

# Find images larger than 1GB
docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}" | grep -E "[0-9]+\.[0-9]+GB"

# Total disk usage by images
docker system df
```

### Date-based Filtering
```bash
# Images created in last week
docker images --filter "since=$(date -d '1 week ago' '+%Y-%m-%d')"

# Images older than 30 days
docker images --filter "before=$(date -d '30 days ago' '+%Y-%m-%d')"

# Images created today
docker images --filter "since=$(date '+%Y-%m-%d')"
```

### Cleanup Operations
```bash
# List dangling images for cleanup
docker images --filter "dangling=true" -q

# Remove all dangling images
docker rmi $(docker images --filter "dangling=true" -q)

# List images without running containers
docker images --filter "dangling=false" --format "{{.Repository}}:{{.Tag}}"
```

## Real-World Use Cases

### 1. Development Environment Management
```bash
# Check what development images are available
docker images --filter "reference=*/dev" --filter "reference=*:dev"

# List all node.js related images
docker images | grep node

# Find latest versions
docker images --filter "reference=*:latest"
```

### 2. Cleanup and Maintenance
```bash
# Find large images consuming disk space
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -k3 -hr | head -10

# Identify old unused images
docker images --filter "before=$(date -d '1 month ago' '+%Y-%m-%d')"

# Check for duplicate images (same ID, different tags)
docker images --format "{{.ID}}\t{{.Repository}}:{{.Tag}}" | sort
```

### 3. CI/CD Pipeline Integration
```bash
# List images for specific project
docker images --filter "label=project=myapp"

# Get latest build image ID
docker images --filter "reference=myapp:*" --format "{{.ID}}" | head -1

# Check if specific image exists
if docker images --format "{{.Repository}}:{{.Tag}}" | grep -q "myapp:latest"; then
    echo "Image exists"
else
    echo "Image not found"
fi
```

### 4. Security and Compliance
```bash
# Find images without specific labels
docker images --filter "label!=security.scan=passed"

# List images by maintainer
docker images --filter "label=maintainer=security-team"

# Check image creation dates for compliance
docker images --format "{{.Repository}}:{{.Tag}} - {{.CreatedSince}}"
```

## Scripting Examples

### Automated Image Inventory
```bash
#!/bin/bash
# Generate image inventory report
echo "Docker Image Inventory Report - $(date)"
echo "==========================================="
echo

echo "Total Images: $(docker images -q | wc -l)"
echo "Dangling Images: $(docker images --filter 'dangling=true' -q | wc -l)"
echo "Total Disk Usage: $(docker system df --format '{{.TotalCount}} {{.Size}}' | awk '/Images/ {print $2}')"
echo

echo "Top 10 Largest Images:"
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -k3 -hr | head -11
echo

echo "Recently Created Images (last 7 days):"
docker images --filter "since=$(date -d '7 days ago' '+%Y-%m-%d')" --format "table {{.Repository}}\t{{.Tag}}\t{{.CreatedSince}}"
```

### Image Cleanup Script
```bash
#!/bin/bash
# Safe image cleanup script
echo "Starting image cleanup..."

# Remove dangling images
echo "Removing dangling images..."
docker image prune -f

# List old images (older than 30 days)
old_images=$(docker images --filter "before=$(date -d '30 days ago' '+%Y-%m-%d')" -q)

if [ -n "$old_images" ]; then
    echo "Found old images:"
    docker images --filter "before=$(date -d '30 days ago' '+%Y-%m-%d')"
    read -p "Remove these old images? (y/N): " confirm
    if [[ $confirm == [yY] ]]; then
        docker rmi $old_images
        echo "Old images removed"
    fi
else
    echo "No old images found"
fi

echo "Cleanup completed"
```

### Image Tag Management
```bash
#!/bin/bash
# List all tags for a repository
repository="$1"

if [ -z "$repository" ]; then
    echo "Usage: $0 <repository>"
    exit 1
fi

echo "All tags for $repository:"
docker images --format "{{.Tag}}" "$repository" | sort -V

echo
echo "Latest tag info:"
docker images "$repository:latest" --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.CreatedSince}}\t{{.Size}}"
```
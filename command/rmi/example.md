# Examples

## Basic Examples

### Remove Single Image
```bash
# Remove image by repository:tag
docker rmi ubuntu:20.04

# Remove image by ID
docker rmi 1a2b3c4d5e6f

# Remove image by partial ID
docker rmi 1a2b

# Remove image by digest
docker rmi ubuntu@sha256:abcd1234...
```

### Remove Multiple Images
```bash
# Remove multiple images by name
docker rmi ubuntu:18.04 ubuntu:20.04 ubuntu:22.04

# Remove multiple images by ID
docker rmi 1a2b3c4d 5e6f7g8h 9i0j1k2l

# Mix names and IDs
docker rmi ubuntu:20.04 1a2b3c4d nginx:latest
```

### Remove All Images with Pattern
```bash
# Remove all ubuntu images
docker rmi $(docker images ubuntu -q)

# Remove all images with 'test' in name
docker rmi $(docker images | grep test | awk '{print $3}')

# Remove all untagged images
docker rmi $(docker images --filter "dangling=true" -q)
```

## Force Removal Examples

### Force Remove Single Image
```bash
# Force remove image even if containers reference it
docker rmi -f ubuntu:20.04

# Force remove by ID
docker rmi -f 1a2b3c4d5e6f

# Force remove multiple images
docker rmi -f ubuntu:18.04 ubuntu:20.04
```

### Force Remove All Images
```bash
# Nuclear option - remove ALL images (dangerous!)
docker rmi -f $(docker images -aq)

# Remove all images for specific repository
docker rmi -f $(docker images ubuntu -q)

# Remove all dangling images forcefully
docker rmi -f $(docker images --filter "dangling=true" -q)
```

## Conditional Removal Examples

### Safe Removal with Checks
```bash
# Check if image is used by containers before removal
image_name="ubuntu:20.04"
if ! docker ps -a --format "{{.Image}}" | grep -q "$image_name"; then
    docker rmi "$image_name"
    echo "Image removed safely"
else
    echo "Image is still in use by containers"
fi
```

### Remove Old Images
```bash
# Remove images older than 30 days
docker images --filter "before=$(date -d '30 days ago' '+%Y-%m-%d')" -q | xargs -r docker rmi

# Remove images created before specific image
docker rmi $(docker images --filter "before=ubuntu:20.04" -q)

# Remove images created after specific date
docker rmi $(docker images --filter "since=2023-01-01" -q)
```

### Size-Based Removal
```bash
# Function to remove large images
remove_large_images() {
    local size_threshold="1GB"
    echo "Finding images larger than $size_threshold..."
    docker images --format "table {{.Repository}}:{{.Tag}}\t{{.Size}}" | grep GB | while read line; do
        image=$(echo "$line" | awk '{print $1}')
        size=$(echo "$line" | awk '{print $2}')
        echo "Large image found: $image ($size)"
        read -p "Remove $image? (y/N): " confirm
        if [[ $confirm == [yY] ]]; then
            docker rmi "$image"
        fi
    done
}
```

## Advanced Examples

### Cleanup Scripts
```bash
#!/bin/bash
# Comprehensive image cleanup script
echo "Starting Docker image cleanup..."

# Remove dangling images
echo "Removing dangling images..."
dangling=$(docker images --filter "dangling=true" -q)
if [ -n "$dangling" ]; then
    docker rmi $dangling
    echo "Dangling images removed"
else
    echo "No dangling images found"
fi

# Remove unused images
echo "Removing unused images..."
docker image prune -f

# Optional: Remove old images
read -p "Remove images older than 30 days? (y/N): " confirm
if [[ $confirm == [yY] ]]; then
    old_images=$(docker images --filter "before=$(date -d '30 days ago' '+%Y-%m-%d')" -q)
    if [ -n "$old_images" ]; then
        docker rmi $old_images
        echo "Old images removed"
    else
        echo "No old images found"
    fi
fi

echo "Cleanup completed"
```

### Selective Repository Cleanup
```bash
#!/bin/bash
# Remove all but latest N versions of each repository
keep_versions=3

for repo in $(docker images --format "{{.Repository}}" | sort | uniq); do
    if [ "$repo" != "<none>" ]; then
        echo "Processing repository: $repo"
        
        # Get all tags for this repository, sorted by creation date
        tags=$(docker images "$repo" --format "{{.Tag}}\t{{.CreatedAt}}" | sort -k2 -r | tail -n +$((keep_versions + 1)) | awk '{print $1}')
        
        if [ -n "$tags" ]; then
            echo "Removing old versions: $tags"
            for tag in $tags; do
                if [ "$tag" != "<none>" ]; then
                    docker rmi "$repo:$tag" 2>/dev/null || echo "Failed to remove $repo:$tag"
                fi
            done
        fi
    fi
done
```

## Common Use Cases

### Development Environment Cleanup
```bash
# Clean up test and development images
echo "Cleaning development images..."

# Remove all test images
docker rmi $(docker images | grep "test" | awk '{print $3}') 2>/dev/null || echo "No test images found"

# Remove all dev images
docker rmi $(docker images | grep "dev" | awk '{print $3}') 2>/dev/null || echo "No dev images found"

# Remove all images tagged as latest except important ones
important_images=("ubuntu:latest" "nginx:latest" "postgres:latest")
for image in $(docker images --filter "reference=*:latest" --format "{{.Repository}}:{{.Tag}}"); do
    if [[ ! " ${important_images[@]} " =~ " ${image} " ]]; then
        echo "Removing $image"
        docker rmi "$image" 2>/dev/null || echo "Failed to remove $image"
    fi
done
```

### CI/CD Pipeline Cleanup
```bash
# CI/CD cleanup script
#!/bin/bash
echo "CI/CD Image Cleanup"

# Remove build artifacts
build_images=$(docker images --filter "label=build.stage=intermediate" -q)
if [ -n "$build_images" ]; then
    echo "Removing build artifacts..."
    docker rmi $build_images
fi

# Remove failed build images
failed_builds=$(docker images --filter "label=build.status=failed" -q)
if [ -n "$failed_builds" ]; then
    echo "Removing failed builds..."
    docker rmi -f $failed_builds
fi

# Keep only last 5 builds
project="myapp"
for tag in $(docker images "$project" --format "{{.Tag}}" | grep -E '^[0-9]+$' | sort -nr | tail -n +6); do
    echo "Removing old build: $project:$tag"
    docker rmi "$project:$tag"
done
```

### Maintenance and Monitoring
```bash
# Weekly maintenance script
#!/bin/bash
echo "=== Weekly Docker Image Maintenance ==="
echo "Date: $(date)"

# Show current disk usage
echo "Current disk usage:"
docker system df
echo

# Remove dangling images
echo "Removing dangling images..."
docker image prune -f

# Remove unused images older than a week
echo "Removing unused images older than 7 days..."
docker image prune -a -f --filter "until=168h"

# Show largest remaining images
echo "Largest remaining images:"
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -k3 -hr | head -10

# Final disk usage
echo "Final disk usage:"
docker system df
```

## Error Handling Examples

### Robust Removal Script
```bash
#!/bin/bash
# Safe image removal with error handling
remove_images_safely() {
    local images=("$@")
    local failed=()
    local success=()
    
    for image in "${images[@]}"; do
        echo "Attempting to remove: $image"
        if docker rmi "$image" 2>/dev/null; then
            echo "✓ Successfully removed: $image"
            success+=("$image")
        else
            echo "✗ Failed to remove: $image"
            failed+=("$image")
            
            # Try to understand why it failed
            if docker ps -a --format "{{.Image}}" | grep -q "$image"; then
                echo "  Reason: Image is used by containers"
            elif ! docker images --format "{{.Repository}}:{{.Tag}}" | grep -q "$image"; then
                echo "  Reason: Image not found"
            else
                echo "  Reason: Unknown error"
            fi
        fi
    done
    
    echo ""
    echo "Summary:"
    echo "Successfully removed: ${#success[@]} images"
    echo "Failed to remove: ${#failed[@]} images"
    
    if [ ${#failed[@]} -gt 0 ]; then
        echo "Failed images: ${failed[*]}"
        return 1
    fi
    return 0
}

# Usage
remove_images_safely "ubuntu:18.04" "nginx:old" "myapp:test"
```

### Batch Processing with Verification
```bash
#!/bin/bash
# Verify image removal and handle edge cases
verify_and_remove() {
    local image="$1"
    
    # Check if image exists
    if ! docker images --format "{{.Repository}}:{{.Tag}}" | grep -q "^$image$"; then
        echo "Image $image does not exist"
        return 1
    fi
    
    # Check if image is in use
    if docker ps -a --format "{{.Image}}" | grep -q "$image"; then
        echo "Image $image is used by containers:"
        docker ps -a --filter "ancestor=$image" --format "table {{.Names}}\t{{.Status}}"
        read -p "Force remove anyway? (y/N): " force
        if [[ $force == [yY] ]]; then
            docker rmi -f "$image"
        else
            echo "Skipping $image"
            return 1
        fi
    else
        docker rmi "$image"
    fi
    
    # Verify removal
    if docker images --format "{{.Repository}}:{{.Tag}}" | grep -q "^$image$"; then
        echo "Failed to remove $image"
        return 1
    else
        echo "Successfully removed $image"
        return 0
    fi
}
```
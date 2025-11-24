# rmi

## Overview
- Removes one or more Docker images from the local system
- Stands for "remove image" - counterpart to `docker rm` for containers
- Frees up disk space by deleting image layers and metadata
- Cannot remove images that are being used by containers
- Requires image to be untagged or explicitly forced for removal

## Basic Syntax
```bash
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

## Common Options

### Basic Options
- `-f, --force` - Force removal of image even if referenced by containers
- `--no-prune` - Do not delete untagged parent images
- `--prune` - Remove untagged parent images (default behavior)

### Image Specification
- Can specify by image ID (full or partial)
- Can specify by repository:tag format
- Can specify multiple images in single command
- Supports image digest format for precise targeting

## How It Works

### Image Removal Process
1. **Reference Check** - Verifies no containers are using the image
2. **Tag Removal** - Removes specified tags from image
3. **Layer Analysis** - Identifies which layers can be safely removed
4. **Dependency Check** - Ensures no other images depend on layers
5. **Physical Deletion** - Removes image layers and metadata from disk
6. **Space Reclaim** - Frees up disk space used by removed layers

### Image References
- **Tagged Images** - Images with repository:tag labels
- **Untagged Images** - Images without tags (dangling)
- **Intermediate Layers** - Build cache layers
- **Parent Images** - Base images used by other images

### Removal Scenarios
- **Single Tag** - Removes only specified tag, image may remain
- **Last Tag** - Removes image entirely when last tag is removed
- **Force Removal** - Removes image even if containers reference it
- **Cascade Removal** - May trigger removal of unused parent layers

## Best Practices

- **Stop containers first** - Ensure no running containers use the image
- **Check dependencies** - Verify removal won't break other images
- **Use specific tags** - Be explicit about which version to remove
- **Backup important images** - Save critical images before removal
- **Regular cleanup** - Remove unused images to free disk space
- **Avoid force removal** - Only use `--force` when absolutely necessary
- **Monitor disk space** - Check available space after bulk removals
- **Use image prune** - Prefer `docker image prune` for bulk cleanup
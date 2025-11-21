# images

## Overview
- Lists Docker images stored locally on the system
- Shows repository names, tags, image IDs, creation dates, and sizes
- Provides filtering and formatting options for better management
- Essential tool for managing local Docker image inventory
- Helps identify disk space usage and cleanup opportunities

## Basic Syntax
```bash
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

## Common Options

### Basic Options
- `-a, --all` - Show all images (including intermediate layers)
- `-q, --quiet` - Only show image IDs
- `--no-trunc` - Don't truncate output (show full image IDs)
- `--digests` - Show image digests
- `-f, --filter` - Filter output based on conditions

### Formatting Options
- `--format` - Pretty-print images using a custom template
- `--tree` - Show image tree (if supported)

### Repository and Tag Filtering
- Can specify repository name to filter results
- Can specify repository:tag for specific image versions
- Supports wildcards and pattern matching

## How It Works

### Image Listing Process
1. **Repository Scan** - Scans local Docker image repository
2. **Metadata Retrieval** - Collects image metadata and layer information
3. **Filter Application** - Applies any specified filters
4. **Format Processing** - Formats output according to specified template
5. **Display Generation** - Presents formatted list to user

### Image Information
- **Repository** - Image repository name
- **Tag** - Image version or tag
- **Image ID** - Unique identifier (SHA256 hash)
- **Created** - When the image was created
- **Size** - Compressed size on disk

### Image Layers
- **Base Layers** - Shared layers between images
- **Intermediate Layers** - Build process layers (hidden by default)
- **Top Layer** - Final image layer
- **Virtual Size** - Total uncompressed size including shared layers

## Best Practices

1. **Regular cleanup** - Remove unused images to free disk space
2. **Use specific tags** - Avoid relying on 'latest' tag for production
3. **Monitor image sizes** - Keep track of large images that consume disk space
4. **Filter effectively** - Use filters to find specific images quickly
5. **Check for updates** - Regularly check for newer image versions
6. **Remove dangling images** - Clean up <none>:<none> images periodically
7. **Use multi-stage builds** - Reduce final image sizes in Dockerfiles
# pull

## Overview
- Downloads Docker images from a registry to local system
- Essential command for obtaining images before running containers
- Supports pulling specific tags, latest versions, and all tags
- Automatically verifies image integrity and authenticity
- Updates local images with newer versions from registry

## Basic Syntax
```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

## Common Options

### Basic Options
- `-a, --all-tags` - Download all tagged images in the repository
- `--disable-content-trust` - Skip image verification (not recommended)
- `--platform` - Set platform if server is multi-platform capable
- `-q, --quiet` - Suppress verbose output

### Authentication Options
- Authentication handled through `docker login`
- Private registry credentials stored securely
- Can specify custom registry in image name

### Registry Options
- Default registry is Docker Hub
- Can specify alternative registries
- Supports private and self-hosted registries

## How It Works

### Pull Process
1. **Registry Connection** - Connects to specified registry (Docker Hub by default)
2. **Authentication Check** - Verifies credentials for private repositories
3. **Manifest Retrieval** - Downloads image manifest and layer information
4. **Layer Download** - Downloads missing or updated image layers
5. **Layer Verification** - Verifies integrity using checksums
6. **Image Assembly** - Assembles layers into complete image
7. **Tagging** - Tags image with specified name in local repository

### Image Layers
- **Base Layers** - Shared foundation layers (cached for efficiency)
- **Application Layers** - Specific application code and dependencies
- **Configuration Layer** - Final layer with image metadata
- **Delta Downloads** - Only downloads changed layers

### Registry Interaction
- **Docker Hub** - Default public registry
- **Private Registries** - Organization-specific repositories
- **Alternative Registries** - ECR, GCR, Azure Container Registry
- **Self-hosted** - Harbor, Nexus, local registries

## Best Practices

1. **Use specific tags** - Avoid `:latest` for production deployments
2. **Verify image sources** - Only pull from trusted registries
3. **Keep images updated** - Regularly pull security updates
4. **Check image sizes** - Monitor bandwidth and storage usage
5. **Use multi-stage builds** - Minimize image sizes
6. **Enable content trust** - Verify image signatures when possible
7. **Cache efficiently** - Leverage layer caching for faster pulls
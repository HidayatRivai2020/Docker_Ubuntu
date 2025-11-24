# tag

## Overview
- Creates a new tag (reference) for an existing Docker image
- Does not duplicate the image data - creates an alias pointing to the same image
- Useful for versioning and organizing images before pushing to registries
- Required for pushing images to Docker Hub or private registries
- Can create multiple tags for the same image
- Tags are metadata that reference the same image layers

## Basic Syntax
```bash
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

## Common Options

### Basic Options
- No additional options required - the command uses positional arguments
- `SOURCE_IMAGE` - The existing image name or ID to tag
- `TARGET_IMAGE` - The new tag/name to create
- Both source and target can include repository paths and tags

## How It Works

### Tagging Process
1. **Image Verification** - Checks if source image exists locally
2. **Reference Creation** - Creates new tag pointing to same image
3. **Metadata Update** - Updates Docker's image database
4. **No Data Duplication** - Both tags share the same image layers

### Process Flow
```
docker tag → Source Exists? → No → Error: Image not found
                   ↓ Yes
          Create New Reference
                   ↓
          Update Image Metadata
                   ↓
          Both Tags Point to Same Image
```

### Tag Structure
```
[REGISTRY_HOST[:PORT]/][USERNAME/]REPOSITORY[:TAG]

Examples:
- myapp:latest
- myapp:v1.0.0
- docker.io/username/myapp:latest
- localhost:5000/myapp:dev
```

## Best Practices

- **Use semantic versioning** for version tags (e.g., v1.0.0, v1.1.0)
- **Tag before pushing** to registries to organize your images properly
- **Create multiple tags** for different contexts (latest, version, commit hash)
- **Include registry URL** when tagging for private registries
- **Use descriptive tags** that indicate environment or purpose (prod, dev, staging)
- **Maintain latest tag** alongside version tags for convenience
- **Follow naming conventions** consistent with your team or organization
- **Tag before deletion** if you need to preserve an image before cleanup

# Troubleshooting

## Common Issues

### Image Not Found
**Problem**: Source image does not exist or cannot be found
```bash
# Check if image exists locally
docker images | grep <image_name>

# List all images
docker images

# Check with full image name
docker images <repository>/<image_name>:<tag>

# Pull image if not available
docker pull <image_name>:<tag>

# Verify image ID exists
docker images --no-trunc | grep <image_id>
```

### Invalid Tag Format
**Problem**: Tag name contains invalid characters or format
```bash
# Valid tag format: [a-zA-Z0-9_.-]
# Tags must not start with . or -

# Invalid examples:
docker tag myapp my app:v1.0       # Spaces not allowed
docker tag myapp myapp:.latest     # Cannot start with .
docker tag myapp myapp:-dev        # Cannot start with -
docker tag myapp myapp:V1.0/test   # / only for repository path

# Valid examples:
docker tag myapp myapp:v1.0
docker tag myapp myapp:dev-latest
docker tag myapp myapp:2025.11.24
docker tag myapp username/myapp:v1.0
```

### Permission Denied
**Problem**: Insufficient permissions to tag or push images
```bash
# Check Docker permissions
ls -la /var/run/docker.sock

# Add user to docker group
sudo usermod -aG docker $USER

# Logout and login again, or use
newgrp docker

# Temporarily use sudo
sudo docker tag <source> <target>

# For registry push issues, login first
docker login
docker login registry.example.com
```

### Tag Already Exists
**Problem**: Target tag already exists with different image
```bash
# Check existing tags
docker images <image_name>

# Remove existing tag before retagging
docker rmi <image_name>:<tag>

# Or force tag (overwrites existing)
docker tag <source> <image_name>:<tag>

# Check which image the tag points to
docker inspect <image_name>:<tag> --format='{{.Id}}'
```

### Registry Authentication Failed
**Problem**: Cannot tag for or push to private registry
```bash
# Login to Docker Hub
docker login

# Login to private registry
docker login registry.example.com

# Login with username
docker login -u username

# Check login status
cat ~/.docker/config.json

# Logout and login again
docker logout registry.example.com
docker login registry.example.com
```

### Out of Disk Space
**Problem**: No space left when tagging (rare, but can happen with metadata)
```bash
# Check disk usage
df -h
docker system df

# Clean up unused images
docker image prune -a

# Remove dangling images
docker image prune

# Clean up all unused data
docker system prune -a

# Check specific image sizes
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### Network Issues During Push
**Problem**: Cannot push tagged image to registry
```bash
# Check network connectivity
ping registry.hub.docker.com
ping registry.example.com

# Verify DNS resolution
nslookup registry.hub.docker.com

# Check proxy settings
echo $HTTP_PROXY
echo $HTTPS_PROXY

# Configure Docker proxy if needed
sudo systemctl edit docker.service

# Test with curl
curl -I https://registry.hub.docker.com
```

## Debugging Commands

### Image Verification
```bash
# List all tags for an image
docker images <image_name>

# Check if specific tag exists
docker images <image_name>:<tag>

# Show image IDs
docker images --no-trunc | grep <image_name>

# Check image details
docker inspect <image_name>:<tag>

# Verify image ID matches
docker inspect <image1>:<tag1> --format='{{.Id}}'
docker inspect <image2>:<tag2> --format='{{.Id}}'
```

### Tag Inspection
```bash
# List all tags with same image ID
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}" | grep <image_id>

# Check tag creation time
docker inspect <image_name>:<tag> --format='{{.Created}}'

# View tag metadata
docker inspect <image_name>:<tag> --format='{{json .RepoTags}}' | jq

# Check image size
docker images <image_name>:<tag> --format "{{.Size}}"

# Verify repository and tag
docker inspect <image_name>:<tag> --format='Repository: {{index .RepoTags 0}}'
```

### Registry Verification
```bash
# Check registry in tag
docker images --format "{{.Repository}}" | grep registry

# Verify registry authentication
docker login registry.example.com
echo $?

# Check what's authenticated
cat ~/.docker/config.json | jq

# Test registry connectivity
curl -I https://registry.example.com/v2/

# Check if image exists in registry
docker pull registry.example.com/image:tag
```

### Tag Conflicts
```bash
# Find duplicate tags
docker images --format "{{.Repository}}:{{.Tag}}" | sort | uniq -d

# Find tags pointing to same image
IMAGE_ID=$(docker inspect myapp:latest --format='{{.Id}}')
docker images --format "{{.Repository}}:{{.Tag}}\t{{.ID}}" | grep $IMAGE_ID

# List all tags for repository
docker images myapp --format "{{.Tag}}"

# Compare two tags
diff <(docker inspect image1:tag1) <(docker inspect image2:tag2)
```

### Authentication Debugging
```bash
# Check current authentication
docker info | grep Username

# Verify credentials file
cat ~/.docker/config.json

# Test authentication
docker login -u username --password-stdin < password.txt

# Check credential helper
cat ~/.docker/config.json | jq '.credsStore'

# Debug authentication
docker --debug login registry.example.com
```

### Image and Tag Analysis
```bash
# Count tags for an image
docker images myapp | tail -n +2 | wc -l

# Check total size of all tags
docker images myapp --format "{{.Size}}"

# Find oldest tag
docker images myapp --format "table {{.Tag}}\t{{.CreatedAt}}" | tail -1

# Find newest tag
docker images myapp --format "table {{.Tag}}\t{{.CreatedAt}}" | head -2

# Export tag list
docker images myapp --format "{{.Repository}}:{{.Tag}}" > tags.txt
```

## Exit Codes and Meanings

### Standard Exit Codes
- **0** - Successful execution
- **1** - General error (image not found, invalid format, permission denied)
- **125** - Docker daemon error
- **126** - Command not executable
- **127** - Command not found

### Common Error Messages
- **"No such image"** - Source image doesn't exist
- **"invalid reference format"** - Tag format is incorrect
- **"denied: requested access to the resource is denied"** - Authentication or permission issue
- **"repository name must be lowercase"** - Tag contains uppercase in repository name
- **"tag name is too long"** - Tag exceeds maximum length (128 characters)

### Checking Exit Codes
```bash
# Tag and check result
docker tag myapp:latest myapp:v1.0
echo $?

# Check for errors
docker tag nonexistent:image new:tag 2>&1
if [ $? -ne 0 ]; then
    echo "Tag operation failed"
fi

# Capture error output
ERROR=$(docker tag badimage new:tag 2>&1)
if [ $? -ne 0 ]; then
    echo "Error: $ERROR"
fi
```

## Recovery Procedures

### Fix Missing Source Image
```bash
# Step 1: Verify image doesn't exist
docker images | grep <image_name>

# Step 2: Search for correct image name
docker images

# Step 3: Pull image if needed
docker pull <correct_image_name>:<tag>

# Step 4: Tag the pulled image
docker tag <correct_image_name>:<tag> <target_name>:<tag>

# Step 5: Verify tag was created
docker images <target_name>
```

### Fix Invalid Tag Format
```bash
# Step 1: Check current tag attempt
# Invalid: docker tag myapp my app:v1.0

# Step 2: Fix format (remove spaces, special chars)
docker tag myapp myapp:v1.0

# Step 3: Follow naming conventions
# Use lowercase for repository
docker tag myapp username/myapp:v1.0

# Step 4: Use valid characters only [a-zA-Z0-9_.-]
docker tag myapp myapp:dev-v1.0-build123
```

### Resolve Authentication Issues
```bash
# Step 1: Logout from registry
docker logout registry.example.com

# Step 2: Login again with correct credentials
docker login registry.example.com

# Step 3: Verify login
cat ~/.docker/config.json | jq

# Step 4: Tag with registry prefix
docker tag myapp:latest registry.example.com/myapp:latest

# Step 5: Test push
docker push registry.example.com/myapp:latest
```

### Clean Up Conflicting Tags
```bash
#!/bin/bash
# Remove duplicate or conflicting tags

IMAGE=$1
TAG=$2

if [ -z "$IMAGE" ] || [ -z "$TAG" ]; then
    echo "Usage: $0 <image> <tag>"
    exit 1
fi

# Check if tag exists
if docker images | grep -q "$IMAGE.*$TAG"; then
    echo "Removing existing tag: $IMAGE:$TAG"
    docker rmi $IMAGE:$TAG
else
    echo "Tag $IMAGE:$TAG does not exist"
fi

echo "You can now create the tag again"
```

### Automated Debugging Script
```bash
#!/bin/bash
# Comprehensive tag debugging script

SOURCE=$1
TARGET=$2

if [ -z "$SOURCE" ] || [ -z "$TARGET" ]; then
    echo "Usage: $0 <source_image> <target_image>"
    echo "Example: $0 myapp:latest myapp:v1.0"
    exit 1
fi

echo "=== Docker Tag Debug Report ==="
echo "Source: $SOURCE"
echo "Target: $TARGET"
echo "Timestamp: $(date)"
echo

echo "=== Checking Source Image ==="
if docker images | grep -q "$(echo $SOURCE | cut -d: -f1)"; then
    echo "✓ Source image found"
    docker images | grep "$(echo $SOURCE | cut -d: -f1)"
else
    echo "✗ Source image not found"
    echo "Available images:"
    docker images
    exit 1
fi
echo

echo "=== Validating Target Format ==="
if [[ "$TARGET" =~ ^[a-z0-9._/-]+:[a-zA-Z0-9._-]+$ ]]; then
    echo "✓ Target format is valid"
else
    echo "⚠ Target format may be invalid"
    echo "Valid format: repository:tag"
    echo "Repository can contain: a-z 0-9 . _ / -"
    echo "Tag can contain: a-z A-Z 0-9 . _ -"
fi
echo

echo "=== Checking Target Conflicts ==="
if docker images | grep -q "$(echo $TARGET | cut -d: -f1).*$(echo $TARGET | cut -d: -f2)"; then
    echo "⚠ Target tag already exists"
    docker images | grep "$(echo $TARGET | cut -d: -f1).*$(echo $TARGET | cut -d: -f2)"
    echo "This will be overwritten"
else
    echo "✓ No conflicts detected"
fi
echo

echo "=== Attempting Tag Operation ==="
if docker tag "$SOURCE" "$TARGET"; then
    echo "✓ Tag operation successful"
    docker images "$TARGET"
else
    echo "✗ Tag operation failed"
    exit 1
fi
```

### Registry Push Troubleshooting Script
```bash
#!/bin/bash
# Debug registry push issues

IMAGE=$1
REGISTRY=${2:-docker.io}

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image> [registry]"
    echo "Example: $0 myapp:v1.0 registry.example.com"
    exit 1
fi

echo "=== Registry Push Troubleshooting ==="
echo "Image: $IMAGE"
echo "Registry: $REGISTRY"
echo

echo "1. Checking Network Connectivity:"
if ping -c 3 $REGISTRY > /dev/null 2>&1; then
    echo "✓ Registry is reachable"
else
    echo "✗ Cannot reach registry"
fi
echo

echo "2. Checking DNS Resolution:"
nslookup $REGISTRY || echo "⚠ DNS resolution issue"
echo

echo "3. Checking Authentication:"
if grep -q $REGISTRY ~/.docker/config.json; then
    echo "✓ Authentication configured"
else
    echo "⚠ Not authenticated to registry"
    echo "Run: docker login $REGISTRY"
fi
echo

echo "4. Checking Image Exists:"
if docker images | grep -q "$IMAGE"; then
    echo "✓ Image exists locally"
    docker images | grep "$IMAGE"
else
    echo "✗ Image not found locally"
fi
echo

echo "5. Checking Tag Format:"
FULL_TAG="$REGISTRY/$IMAGE"
echo "Full tag will be: $FULL_TAG"
if [[ "$FULL_TAG" =~ ^[a-z0-9._/-]+:[a-zA-Z0-9._-]+$ ]]; then
    echo "✓ Format is valid"
else
    echo "⚠ Format may be invalid"
fi
```

### Tag Validation Script
```bash
#!/bin/bash
# Validate tag format before applying

validate_tag() {
    local tag=$1
    
    # Check if tag is empty
    if [ -z "$tag" ]; then
        echo "Error: Tag cannot be empty"
        return 1
    fi
    
    # Check tag length (max 128 characters)
    if [ ${#tag} -gt 128 ]; then
        echo "Error: Tag is too long (max 128 characters)"
        return 1
    fi
    
    # Check for invalid characters
    if [[ ! "$tag" =~ ^[a-zA-Z0-9][a-zA-Z0-9._-]*$ ]]; then
        echo "Error: Tag contains invalid characters"
        echo "Valid characters: a-zA-Z0-9._-"
        echo "Must not start with . or -"
        return 1
    fi
    
    # Check for uppercase in repository part (if present)
    if [[ "$tag" =~ [A-Z] ]] && [[ "$tag" =~ / ]]; then
        local repo=$(echo "$tag" | cut -d: -f1)
        if [[ "$repo" =~ [A-Z] ]]; then
            echo "Error: Repository name must be lowercase"
            return 1
        fi
    fi
    
    echo "✓ Tag format is valid"
    return 0
}

# Test the function
TAG="$1"
if [ -z "$TAG" ]; then
    echo "Usage: $0 <tag>"
    echo "Example: $0 myapp:v1.0.0"
    exit 1
fi

validate_tag "$TAG"
```

### Batch Tag Recovery Script
```bash
#!/bin/bash
# Recover or recreate multiple tags

SOURCE_IMAGE=$1
TAG_FILE=$2

if [ -z "$SOURCE_IMAGE" ] || [ -z "$TAG_FILE" ]; then
    echo "Usage: $0 <source_image> <tag_file>"
    echo "tag_file should contain one tag per line"
    exit 1
fi

echo "=== Batch Tag Recovery ==="
echo "Source: $SOURCE_IMAGE"
echo "Tag file: $TAG_FILE"
echo

# Check source image exists
if ! docker images | grep -q "$(echo $SOURCE_IMAGE | cut -d: -f1)"; then
    echo "Error: Source image not found"
    exit 1
fi

# Read tags and create them
while IFS= read -r tag; do
    # Skip empty lines and comments
    [[ -z "$tag" || "$tag" =~ ^# ]] && continue
    
    echo "Creating tag: $tag"
    if docker tag "$SOURCE_IMAGE" "$tag"; then
        echo "✓ Success: $tag"
    else
        echo "✗ Failed: $tag"
    fi
done < "$TAG_FILE"

echo
echo "=== Summary ==="
echo "Tags created:"
docker images --format "table {{.Repository}}\t{{.Tag}}" | grep -f "$TAG_FILE" || echo "None"
```

## Best Practices for Troubleshooting

### Before Tagging
```bash
# 1. Verify source image exists
docker images | grep <source_image>

# 2. Check tag format
echo <target_tag> | grep -E '^[a-zA-Z0-9][a-zA-Z0-9._-]*$'

# 3. Check for conflicts
docker images <target_image>

# 4. Verify disk space
df -h
docker system df
```

### When Tag Fails
```bash
# 1. Check error message carefully
docker tag source:tag target:tag 2>&1 | tee error.log

# 2. Verify source exists
docker inspect source:tag

# 3. Try with image ID
docker tag <image_id> target:tag

# 4. Check permissions
ls -la /var/run/docker.sock
```

### For Registry Tags
```bash
# 1. Authenticate first
docker login registry.example.com

# 2. Use full registry path
docker tag myapp:latest registry.example.com/username/myapp:latest

# 3. Test connectivity
ping registry.example.com

# 4. Verify authentication
cat ~/.docker/config.json | jq
```

### Regular Maintenance
```bash
# Clean up old tags
docker images | grep '<none>' | awk '{print $3}' | xargs docker rmi

# Remove unused images
docker image prune -a

# Check disk usage
docker system df

# Verify tag consistency
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}"

# Audit tags
docker images --format "{{.Repository}}:{{.Tag}}" > current-tags.txt
```

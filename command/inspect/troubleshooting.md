# Docker inspect Troubleshooting

## Common Issues

### Object Not Found
**Problem**: Cannot find the specified container, image, or other object
```
Error: No such object: container_name
```

**Solutions**:
```bash
# Check if container exists
docker ps -a | grep container_name

# List all containers
docker ps -a

# Check if image exists
docker images | grep image_name

# Use partial IDs or exact names
docker inspect 1a2b3c4d
docker inspect full_container_name

# Inspect by full ID
docker inspect sha256:abcd1234567890...
```

### Invalid Format Template
**Problem**: Go template syntax errors in format string
```
template: :1:15: executing template: map has no entry for key "InvalidKey"
```

**Solutions**:
```bash
# Check available fields first
docker inspect container_name | jq 'keys'

# Use proper Go template syntax
docker inspect --format='{{.State.Status}}' container_name

# Handle missing fields with conditional logic
docker inspect --format='{{if .State.Health}}{{.State.Health.Status}}{{else}}No health check{{end}}' container_name

# Test templates with simple fields first
docker inspect --format='{{.Name}}' container_name
```

### Permission Denied
**Problem**: Insufficient permissions to inspect Docker objects
```
Got permission denied while trying to connect to the Docker daemon socket
```

**Solutions**:
```bash
# Use sudo (temporary fix)
sudo docker inspect container_name

# Add user to docker group
sudo usermod -aG docker $USER
# Log out and log back in

# Check Docker daemon status
sudo systemctl status docker

# Verify user permissions
groups $USER | grep docker
```

### Large JSON Output
**Problem**: Overwhelming amount of data in JSON output
```
Output is too verbose and hard to read
```

**Solutions**:
```bash
# Use format templates to extract specific information
docker inspect --format='{{.State.Status}}' container_name

# Use jq to filter and format output
docker inspect container_name | jq '.[] | {Name: .Name, Status: .State.Status}'

# Get only specific sections
docker inspect container_name | jq '.[].NetworkSettings'

# Combine with grep for quick searches
docker inspect container_name | grep -i "ipaddress"
```

### Type Ambiguity
**Problem**: Object name matches multiple types (container and image with same name)
```
Error response from daemon: ambiguous reference
```

**Solutions**:
```bash
# Specify object type explicitly
docker inspect --type=container container_name
docker inspect --type=image image_name

# Use full IDs to avoid ambiguity
docker inspect sha256:1234567890abcdef

# List objects of specific type
docker container ls -a | grep name
docker image ls | grep name
```

### Format Output Issues
**Problem**: Formatted output not displaying correctly
```
Template output is empty or malformed
```

**Solutions**:
```bash
# Check if field exists
docker inspect container_name | jq 'has("FieldName")'

# Use range for arrays
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name

# Debug template step by step
docker inspect --format='{{.NetworkSettings}}' container_name
docker inspect --format='{{.NetworkSettings.Networks}}' container_name

# Use json format for complex structures
docker inspect --format='{{json .NetworkSettings.Networks}}' container_name | jq .
```

## Debugging Commands

### Object Verification
```bash
# Verify object exists and get basic info
docker ps -a --filter name=container_name
docker images --filter reference=image_name

# Check object type
docker inspect container_name | jq '.[].Type' 2>/dev/null || echo "Object found"

# Get object ID
docker inspect --format='{{.Id}}' object_name

# List all objects
docker ps -a
docker images -a
docker volume ls
docker network ls
```

### Template Testing
```bash
# Test template syntax gradually
docker inspect --format='{{.}}' container_name | head -20
docker inspect --format='{{.State}}' container_name
docker inspect --format='{{.State.Status}}' container_name

# Check available fields
docker inspect container_name | jq 'keys_unsorted'
docker inspect container_name | jq '.[].State | keys'

# Validate template with simple field
docker inspect --format='{{.Name}}' container_name
```

### JSON Structure Analysis
```bash
# Pretty print full JSON
docker inspect container_name | jq .

# Show only top-level keys
docker inspect container_name | jq '.[0] | keys'

# Explore nested structures
docker inspect container_name | jq '.[0].Config | keys'
docker inspect container_name | jq '.[0].NetworkSettings | keys'

# Find specific values
docker inspect container_name | jq '.. | select(type == "string") | select(test("pattern"))'
```

### System Information
```bash
# Check Docker daemon status
docker version
docker info

# Verify Docker daemon connectivity
docker ps > /dev/null && echo "Docker daemon accessible" || echo "Cannot connect to Docker daemon"

# Check system resources
df -h $(docker info --format '{{.DockerRootDir}}')

# Monitor Docker daemon logs
sudo journalctl -u docker.service --since "5 minutes ago"
```

### Network and Storage Analysis
```bash
# Network troubleshooting
docker network ls
docker network inspect bridge

# Volume troubleshooting
docker volume ls
docker volume inspect volume_name

# Check container network configuration
docker inspect container_name | jq '.[].NetworkSettings'

# Check mount information
docker inspect container_name | jq '.[].Mounts'
```

## Error Codes and Meanings

### Standard Exit Codes
- **0** - Successful inspection
- **1** - General error
- **125** - Docker daemon error
- **404** - Object not found

### Docker-Specific Errors
- **No such object** - Container, image, or resource doesn't exist
- **Permission denied** - Insufficient Docker daemon access
- **Template error** - Invalid Go template syntax
- **Ambiguous reference** - Multiple objects match the same name
- **Invalid type** - Specified type doesn't match object

### Checking Inspection Status
```bash
# Verify successful inspection
if docker inspect container_name > /dev/null 2>&1; then
    echo "Object exists and accessible"
else
    echo "Cannot inspect object"
fi

# Check exit code
docker inspect container_name
echo "Exit code: $?"

# Capture and analyze errors
docker inspect invalid_name 2>&1 | grep -q "No such" && echo "Object not found"
```

## Recovery Procedures

### Object Discovery Recovery
```bash
#!/bin/bash
# Find objects when exact name is unknown
search_term="$1"

if [ -z "$search_term" ]; then
    echo "Usage: $0 <search_term>"
    exit 1
fi

echo "Searching for Docker objects matching: $search_term"

# Search containers
echo "Containers:"
docker ps -a --filter name="$search_term" --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

# Search images
echo "Images:"
docker images --filter reference="*$search_term*" --format "table {{.Repository}}\t{{.Tag}}\t{{.ID}}"

# Search volumes
echo "Volumes:"
docker volume ls --filter name="$search_term"

# Search networks
echo "Networks:"
docker network ls --filter name="$search_term"
```

### Template Debugging Recovery
```bash
#!/bin/bash
# Debug and fix template issues
object="$1"
template="$2"

if [ -z "$object" ] || [ -z "$template" ]; then
    echo "Usage: $0 <object_name> <template>"
    echo "Example: $0 mycontainer '{{.State.Status}}'"
    exit 1
fi

echo "Debugging template: $template"
echo "Object: $object"
echo

# Step 1: Verify object exists
if ! docker inspect "$object" > /dev/null 2>&1; then
    echo "Error: Object '$object' not found"
    exit 1
fi

echo "Step 1: Object exists ✓"

# Step 2: Test basic template
echo "Step 2: Testing basic access..."
if docker inspect --format='{{.Name}}' "$object" > /dev/null 2>&1; then
    echo "Basic template access works ✓"
else
    echo "Basic template access failed ✗"
    exit 1
fi

# Step 3: Test user template
echo "Step 3: Testing user template..."
if docker inspect --format="$template" "$object" 2>/dev/null; then
    echo "User template works ✓"
else
    echo "User template failed ✗"
    echo "Available top-level fields:"
    docker inspect "$object" | jq '.[0] | keys' 2>/dev/null || echo "Install jq for better debugging"
fi
```

### Permission Recovery
```bash
#!/bin/bash
# Recover from permission issues
echo "Diagnosing Docker permission issues..."

# Check if Docker daemon is running
if ! systemctl is-active docker > /dev/null 2>&1; then
    echo "Docker daemon is not running"
    echo "Starting Docker daemon..."
    sudo systemctl start docker
fi

# Check user groups
if groups $USER | grep -q docker; then
    echo "User is in docker group ✓"
else
    echo "User not in docker group, adding..."
    sudo usermod -aG docker $USER
    echo "Please log out and log back in for group changes to take effect"
fi

# Check Docker socket permissions
echo "Docker socket permissions:"
ls -la /var/run/docker.sock

# Test basic Docker access
if docker version > /dev/null 2>&1; then
    echo "Docker access working ✓"
else
    echo "Docker access still failing ✗"
    echo "Try: sudo docker inspect <object>"
fi
```

### Comprehensive Object Analysis
```bash
#!/bin/bash
# Comprehensive analysis when inspect fails
object="$1"

if [ -z "$object" ]; then
    echo "Usage: $0 <object_name_or_id>"
    exit 1
fi

echo "=== Comprehensive Docker Object Analysis ==="
echo "Object: $object"
echo "Date: $(date)"
echo

# Check if it's a container
echo "Checking containers..."
if docker ps -a --format "{{.Names}}" | grep -q "^$object$"; then
    echo "Found as container name: $object"
    docker inspect "$object" | jq '.[0] | {Name, Status: .State.Status, Image: .Config.Image}'
elif docker ps -a --format "{{.ID}}" | grep -q "^$object"; then
    echo "Found as container ID: $object"
    docker inspect "$object" | jq '.[0] | {Name, Status: .State.Status, Image: .Config.Image}'
else
    echo "Not found as container"
fi

echo

# Check if it's an image
echo "Checking images..."
if docker images --format "{{.Repository}}:{{.Tag}}" | grep -q "$object"; then
    echo "Found as image: $object"
    docker inspect "$object" | jq '.[0] | {Id, Created, Size}'
elif docker images --format "{{.ID}}" | grep -q "^$object"; then
    echo "Found as image ID: $object"
    docker inspect "$object" | jq '.[0] | {Id, Created, Size}'
else
    echo "Not found as image"
fi

echo

# Check volumes and networks
echo "Checking volumes..."
docker volume ls --format "{{.Name}}" | grep -q "^$object$" && echo "Found as volume: $object" || echo "Not found as volume"

echo "Checking networks..."
docker network ls --format "{{.Name}}" | grep -q "^$object$" && echo "Found as network: $object" || echo "Not found as network"

echo
echo "Analysis complete"
```
# Examples

## Basic Examples

### Inspect Containers
```bash
# Inspect a container by name
docker inspect mycontainer

# Inspect a container by ID
docker inspect 1a2b3c4d5e6f

# Inspect multiple containers
docker inspect web db cache

# Inspect with size information
docker inspect -s mycontainer
```

### Inspect Images
```bash
# Inspect an image
docker inspect ubuntu:20.04

# Inspect multiple images
docker inspect ubuntu:20.04 nginx:alpine python:3.9

# Inspect image by ID
docker inspect sha256:abcd1234...
```

### Inspect Networks and Volumes
```bash
# Inspect a network
docker inspect bridge
docker inspect my_custom_network

# Inspect a volume
docker inspect my_volume
docker inspect $(docker volume ls -q)
```

## Format Templates

### Container Information
```bash
# Get container IP address
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mycontainer

# Get container status
docker inspect --format='{{.State.Status}}' mycontainer

# Get container exit code
docker inspect --format='{{.State.ExitCode}}' mycontainer

# Get container start time
docker inspect --format='{{.State.StartedAt}}' mycontainer
```

### Network Configuration
```bash
# Get all IP addresses
docker inspect --format='{{json .NetworkSettings.Networks}}' mycontainer

# Get specific network IP
docker inspect --format='{{.NetworkSettings.Networks.bridge.IPAddress}}' mycontainer

# Get port mappings
docker inspect --format='{{json .NetworkSettings.Ports}}' mycontainer

# Get gateway address
docker inspect --format='{{.NetworkSettings.Networks.bridge.Gateway}}' mycontainer
```

### Resource Information
```bash
# Get memory limit
docker inspect --format='{{.HostConfig.Memory}}' mycontainer

# Get CPU limit
docker inspect --format='{{.HostConfig.CpuShares}}' mycontainer

# Get restart policy
docker inspect --format='{{.HostConfig.RestartPolicy.Name}}' mycontainer

# Get environment variables
docker inspect --format='{{json .Config.Env}}' mycontainer
```

### Volume and Mount Information
```bash
# Get all mounts
docker inspect --format='{{json .Mounts}}' mycontainer

# Get volume binds
docker inspect --format='{{range .Mounts}}{{.Source}}:{{.Destination}} {{end}}' mycontainer

# Get working directory
docker inspect --format='{{.Config.WorkingDir}}' mycontainer

# Get image name
docker inspect --format='{{.Config.Image}}' mycontainer
```

## Advanced Examples

### JSON Processing with jq
```bash
# Pretty print JSON output
docker inspect mycontainer | jq .

# Extract specific fields
docker inspect mycontainer | jq '.[] | {Name: .Name, Status: .State.Status, IP: .NetworkSettings.IPAddress}'

# Get environment variables as key-value pairs
docker inspect mycontainer | jq '.[] | .Config.Env | .[]' | grep -E '^[A-Z_]+=' | sort

# Get mount information
docker inspect mycontainer | jq '.[] | .Mounts | .[] | {Source: .Source, Destination: .Destination, Type: .Type}'
```

### Batch Processing
```bash
# Inspect all running containers
docker inspect $(docker ps -q)

# Inspect all containers (running and stopped)
docker inspect $(docker ps -aq)

# Get status of all containers
docker inspect $(docker ps -aq) --format='{{.Name}}: {{.State.Status}}'

# Get IP addresses of all running containers
docker ps --format="{{.Names}}" | xargs -I {} docker inspect --format='{}: {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' {}
```

### Container State Analysis
```bash
# Check if container is running
status=$(docker inspect --format='{{.State.Status}}' mycontainer)
if [ "$status" = "running" ]; then
    echo "Container is running"
else
    echo "Container is not running: $status"
fi

# Get container uptime
start_time=$(docker inspect --format='{{.State.StartedAt}}' mycontainer)
echo "Container started at: $start_time"

# Check container health
docker inspect --format='{{if .State.Health}}{{.State.Health.Status}}{{else}}No health check{{end}}' mycontainer
```

## Common Use Cases

### Debugging Network Issues
```bash
#!/bin/bash
# Network debugging script
container="$1"

echo "=== Network Analysis for $container ==="

# Basic network info
echo "Container IP:"
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' "$container"

echo "Port mappings:"
docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}}{{$p}} -> {{(index $conf 0).HostPort}}{{end}}' "$container"

echo "Network mode:"
docker inspect --format='{{.HostConfig.NetworkMode}}' "$container"

echo "DNS settings:"
docker inspect --format='{{json .HostConfig.Dns}}' "$container" | jq .
```

### Configuration Audit
```bash
#!/bin/bash
# Container configuration audit
container="$1"

echo "=== Configuration Audit for $container ==="

echo "Image: $(docker inspect --format='{{.Config.Image}}' "$container")"
echo "Command: $(docker inspect --format='{{json .Config.Cmd}}' "$container" | jq -r '.[]')"
echo "Working Directory: $(docker inspect --format='{{.Config.WorkingDir}}' "$container")"

echo "Environment Variables:"
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' "$container" | sort

echo "Volumes:"
docker inspect --format='{{range .Mounts}}{{.Source}} -> {{.Destination}} ({{.Type}}){{println}}{{end}}' "$container"

echo "Resource Limits:"
echo "  Memory: $(docker inspect --format='{{.HostConfig.Memory}}' "$container") bytes"
echo "  CPU Shares: $(docker inspect --format='{{.HostConfig.CpuShares}}' "$container")"
```

### Health Check Monitoring
```bash
#!/bin/bash
# Health check status monitoring
containers=$(docker ps --format "{{.Names}}")

echo "=== Container Health Status ==="
printf "%-20s %-15s %-15s %s\n" "Container" "Status" "Health" "Started"
printf "%-20s %-15s %-15s %s\n" "----------" "-------" "--------" "-------"

for container in $containers; do
    status=$(docker inspect --format='{{.State.Status}}' "$container")
    health=$(docker inspect --format='{{if .State.Health}}{{.State.Health.Status}}{{else}}N/A{{end}}' "$container")
    started=$(docker inspect --format='{{.State.StartedAt}}' "$container" | cut -d'T' -f1)
    
    printf "%-20s %-15s %-15s %s\n" "$container" "$status" "$health" "$started"
done
```

### Resource Usage Report
```bash
#!/bin/bash
# Container resource usage report
echo "=== Container Resource Report ==="
echo

containers=$(docker ps -a --format "{{.Names}}")

for container in $containers; do
    echo "Container: $container"
    
    # Get basic info
    status=$(docker inspect --format='{{.State.Status}}' "$container")
    image=$(docker inspect --format='{{.Config.Image}}' "$container")
    
    echo "  Status: $status"
    echo "  Image: $image"
    
    if [ "$status" = "running" ]; then
        # Resource limits
        memory=$(docker inspect --format='{{.HostConfig.Memory}}' "$container")
        cpu_shares=$(docker inspect --format='{{.HostConfig.CpuShares}}' "$container")
        
        echo "  Memory Limit: $(if [ "$memory" -eq 0 ]; then echo "Unlimited"; else echo "$memory bytes"; fi)"
        echo "  CPU Shares: $(if [ "$cpu_shares" -eq 0 ]; then echo "Default (1024)"; else echo "$cpu_shares"; fi)"
        
        # Network info
        ip=$(docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' "$container")
        echo "  IP Address: $ip"
    fi
    
    echo
done
```

### Image Analysis
```bash
#!/bin/bash
# Docker image analysis
image="$1"

if [ -z "$image" ]; then
    echo "Usage: $0 <image_name>"
    exit 1
fi

echo "=== Image Analysis: $image ==="

# Basic image info
echo "Image ID: $(docker inspect --format='{{.Id}}' "$image")"
echo "Created: $(docker inspect --format='{{.Created}}' "$image")"
echo "Architecture: $(docker inspect --format='{{.Architecture}}' "$image")"
echo "OS: $(docker inspect --format='{{.Os}}' "$image")"

# Size information
echo "Size: $(docker inspect --format='{{.Size}}' "$image") bytes"
echo "Virtual Size: $(docker inspect --format='{{.VirtualSize}}' "$image") bytes"

# Configuration
echo "Default Command: $(docker inspect --format='{{json .Config.Cmd}}' "$image" | jq -r 'if . then join(" ") else "None" end')"
echo "Entrypoint: $(docker inspect --format='{{json .Config.Entrypoint}}' "$image" | jq -r 'if . then join(" ") else "None" end')"
echo "Working Directory: $(docker inspect --format='{{.Config.WorkingDir}}' "$image")"
echo "User: $(docker inspect --format='{{.Config.User}}' "$image")"

# Exposed ports
echo "Exposed Ports:"
docker inspect --format='{{range $port, $config := .Config.ExposedPorts}}{{$port}}{{println}}{{end}}' "$image" || echo "  None"

# Environment variables
echo "Environment Variables:"
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' "$image" | sort

# Labels
echo "Labels:"
docker inspect --format='{{range $key, $value := .Config.Labels}}{{$key}}={{$value}}{{println}}{{end}}' "$image" || echo "  None"
```

## Automation and Scripting

### Container Inventory Script
```bash
#!/bin/bash
# Complete container inventory
echo "Docker Container Inventory - $(date)"
echo "=========================================="
echo

all_containers=$(docker ps -a --format "{{.Names}}")

if [ -z "$all_containers" ]; then
    echo "No containers found."
    exit 0
fi

for container in $all_containers; do
    echo "Container: $container"
    echo "  ID: $(docker inspect --format='{{.Id}}' "$container" | cut -c1-12)"
    echo "  Image: $(docker inspect --format='{{.Config.Image}}' "$container")"
    echo "  Status: $(docker inspect --format='{{.State.Status}}' "$container")"
    echo "  Created: $(docker inspect --format='{{.Created}}' "$container" | cut -d'T' -f1)"
    
    # Network information
    networks=$(docker inspect --format='{{range $net, $conf := .NetworkSettings.Networks}}{{$net}}={{.IPAddress}} {{end}}' "$container")
    echo "  Networks: $networks"
    
    # Port mappings
    ports=$(docker inspect --format='{{range $p, $conf := .NetworkSettings.Ports}}{{if $conf}}{{$p}}->{{(index $conf 0).HostPort}} {{end}}{{end}}' "$container")
    if [ -n "$ports" ]; then
        echo "  Port Mappings: $ports"
    fi
    
    # Volume mounts
    mounts=$(docker inspect --format='{{len .Mounts}}' "$container")
    if [ "$mounts" -gt 0 ]; then
        echo "  Volumes:"
        docker inspect --format='{{range .Mounts}}    {{.Source}} -> {{.Destination}} ({{.Type}}){{println}}{{end}}' "$container"
    fi
    
    echo
done
```

### Configuration Comparison
```bash
#!/bin/bash
# Compare configurations of two containers
container1="$1"
container2="$2"

if [ -z "$container1" ] || [ -z "$container2" ]; then
    echo "Usage: $0 <container1> <container2>"
    exit 1
fi

echo "Configuration Comparison: $container1 vs $container2"
echo "======================================================"

# Compare basic settings
echo "Images:"
echo "  $container1: $(docker inspect --format='{{.Config.Image}}' "$container1")"
echo "  $container2: $(docker inspect --format='{{.Config.Image}}' "$container2")"
echo

echo "Commands:"
echo "  $container1: $(docker inspect --format='{{json .Config.Cmd}}' "$container1" | jq -r 'join(" ")')"
echo "  $container2: $(docker inspect --format='{{json .Config.Cmd}}' "$container2" | jq -r 'join(" ")')"
echo

echo "Working Directories:"
echo "  $container1: $(docker inspect --format='{{.Config.WorkingDir}}' "$container1")"
echo "  $container2: $(docker inspect --format='{{.Config.WorkingDir}}' "$container2")"
echo

echo "Restart Policies:"
echo "  $container1: $(docker inspect --format='{{.HostConfig.RestartPolicy.Name}}' "$container1")"
echo "  $container2: $(docker inspect --format='{{.HostConfig.RestartPolicy.Name}}' "$container2")"
echo

# Compare environment variables
echo "Environment Variables:"
echo "  Differences:"
diff -u <(docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' "$container1" | sort) \
        <(docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' "$container2" | sort) || echo "  No differences found"
```
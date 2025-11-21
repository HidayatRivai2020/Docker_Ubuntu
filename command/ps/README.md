# ps

## Overview
- Lists Docker containers running on the system
- Displays container information in tabular format
- Provides filtering and formatting options for container management
- Essential tool for monitoring and troubleshooting container status
- Supports both running and stopped containers with appropriate flags

## Basic Syntax
```bash
docker ps [OPTIONS]
```

## Common Options

### Container Selection
- `-a, --all` - Show all containers (default shows only running)
- `-f, --filter` - Filter output based on conditions
- `-n, --last` - Show n last created containers (includes all states)
- `-l, --latest` - Show the latest created container (includes all states)
- `-q, --quiet` - Only display container IDs

### Output Formatting
- `--format` - Format output using Go template syntax
- `--no-trunc` - Don't truncate output (show full information)
- `-s, --size` - Display total file sizes

### Common Filters
- `status=running|paused|exited|dead` - Filter by container status
- `name=container_name` - Filter by container name
- `id=container_id` - Filter by container ID
- `label=key=value` - Filter by labels
- `ancestor=image_name` - Filter by image used to create container
- `before=container` - Containers created before specified container
- `since=container` - Containers created after specified container
- `health=starting|healthy|unhealthy|none` - Filter by health status
- `network=network_name` - Filter by connected network
- `volume=volume_name` - Filter by mounted volume

## How It Works

### Container Discovery Process
1. **Daemon Query** - Connects to Docker daemon to retrieve container information
2. **Status Collection** - Gathers current state, resource usage, and metadata
3. **Filter Application** - Applies any specified filters to container list
4. **Format Processing** - Formats output according to specified template or default table
5. **Display Rendering** - Presents filtered and formatted container information

### Information Categories
- **Identity** - Container ID, name, image source
- **Status** - Current state, uptime, exit codes
- **Network** - Port mappings, network connections
- **Resource** - CPU usage, memory consumption, storage
- **Configuration** - Command, creation time, labels

### Default Output Columns
- **CONTAINER ID** - Short container identifier (first 12 characters)
- **IMAGE** - Docker image used to create the container
- **COMMAND** - Command running inside the container
- **CREATED** - Time since container was created
- **STATUS** - Current container status and uptime
- **PORTS** - Published port mappings
- **NAMES** - Container name(s)

## Best Practices

1. **Use filters effectively** - Narrow down results with specific filters
2. **Regular monitoring** - Check container status periodically
3. **Format for automation** - Use `--format` and `-q` for scripts
4. **Monitor resource usage** - Use `-s` flag to track storage consumption
5. **Check all containers** - Use `-a` to see stopped containers
6. **Combine with other commands** - Pipe output to grep, awk, or other tools
7. **Use meaningful names** - Name containers for easier identification
8. **Monitor health status** - Filter by health status for health-checked containers
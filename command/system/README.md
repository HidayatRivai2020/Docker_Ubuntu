# system

## Overview
- Manages Docker system-wide operations including disk usage monitoring, resource cleanup, and system information retrieval
- Provides subcommands for maintaining Docker environments and managing system resources
- Essential for monitoring Docker health, cleaning up unused resources, and troubleshooting system-level issues
- Helps prevent disk space exhaustion and optimize Docker performance

## Basic Syntax
```bash
docker system COMMAND [OPTIONS]
```

## Subcommands

### docker system df
- Display disk usage information for Docker resources

#### Basic Syntax
```bash
docker system df [OPTIONS]
```

#### Common Options
- `-v, --verbose` - Show detailed information
- `--format` - Format output using Go template

### docker system prune
- Remove unused Docker data including containers, networks, images, and optionally volumes

#### Basic Syntax
```bash
docker system prune [OPTIONS]
```

#### Common Options
- `-a, --all` - Remove all unused images, not just dangling ones
- `-f, --force` - Do not prompt for confirmation
- `--volumes` - Prune volumes as well
- `--filter` - Filter based on conditions (until, label)

### docker system info
- Display system-wide information about Docker installation

#### Basic Syntax
```bash
docker system info [OPTIONS]
docker info [OPTIONS]  # Shorthand
```

#### Common Options
- `-f, --format` - Format output using Go template

### docker system events
- Get real-time events from the Docker daemon

#### Basic Syntax
```bash
docker system events [OPTIONS]
docker events [OPTIONS]  # Shorthand
```

#### Common Options
- `--since` - Show events created since timestamp
- `--until` - Show events created until timestamp
- `--filter` - Filter output based on conditions
- `--format` - Format output using Go template

## How It Works

### System Management Flow
```
docker system → Subcommand → Resource Analysis/Action
                    ↓
            df/prune/info/events
                    ↓
            Display/Modify System State
```

### Resource Tracking
1. **Monitor** - Check disk usage with `df`
2. **Analyze** - Identify unused resources
3. **Clean** - Remove unnecessary data with `prune`
4. **Verify** - Confirm cleanup results

## Best Practices

- **Monitor disk usage regularly** - Check `docker system df` frequently to prevent space exhaustion
- **Schedule automated cleanup** - Use cron or systemd timers for regular maintenance
- **Use filters for safe cleanup** - Protect important resources with label or time filters
- **Backup critical images** - Export important images before running aggressive prune operations
- **Use specific prune commands** - Prefer targeted cleanup (container, image, volume) over `system prune -a`
- **Review before pruning** - Understand what will be removed before executing
- **Use verbose mode** - Check `docker system df -v` for detailed resource information
- **Monitor system events** - Track Docker daemon events for troubleshooting

## Understanding System Resources

### Images
- **Dangling**: Untagged images (`<none>:<none>`)
- **Unused**: Images not referenced by any container
- **Active**: Images used by running or stopped containers

### Containers
- **Running**: Currently active containers
- **Stopped**: Exited containers still on disk
- **Paused**: Paused containers (still active)

### Volumes
- **Anonymous**: Created without explicit names
- **Named**: Created with specific names
- **Unused**: Not mounted by any container

### Build Cache
- **Local**: Build cache from local builds
- **Inactive**: Cache not used in recent builds
- **Active**: Cache used by recent builds

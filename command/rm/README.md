# rm

## Overview
- Removes one or more containers from the system
- Only works on stopped containers (use `docker stop` first)
- Permanently deletes container and its filesystem layers
- Cannot be undone once executed
- Frees up disk space used by container

## Basic Syntax
```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

## Common Options

### Basic Options
- `-f, --force` - Force removal of running container (sends SIGKILL)
- `-v, --volumes` - Remove anonymous volumes associated with container
- `-l, --link` - Remove the specified link

### Multiple Containers
- Can specify multiple container names or IDs
- Removes all specified containers in sequence
- Stops on first error unless using `--force`

## How It Works

### Removal Process
1. **Container Check** - Verifies container exists and is stopped
2. **Volume Handling** - Removes anonymous volumes if `-v` specified
3. **Filesystem Cleanup** - Removes container's filesystem layers
4. **Metadata Removal** - Removes container from Docker's database
5. **Resource Cleanup** - Frees up associated system resources

### Container States
- **Running** - Must be stopped first (or use `--force`)
- **Exited** - Ready for removal
- **Paused** - Must be unpaused and stopped first
- **Dead** - Can be removed directly

## Best Practices

- **Stop containers gracefully** before removing (avoid `--force` when possible)
- **Use `--volumes`** to clean up anonymous volumes and free disk space
- **Backup important data** before removing containers
- **Use container names** instead of IDs for better readability
- **Remove containers regularly** to prevent disk space accumulation
- **Check dependencies** before removing linked containers
- **Use `docker container prune`** to remove all stopped containers at once
- **Verify container is stopped** before attempting removal without force
# stop

## Overview
- Stops one or more running containers gracefully
- Sends SIGTERM signal first, then SIGKILL after timeout
- Does not remove the container (use `docker rm` to delete)
- Container can be restarted later with `docker start`

## Basic Syntax
```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

## Common Options

### Basic Options
- `-t, --time` - Seconds to wait for stop before killing (default 10)

### Multiple Containers
- Can specify multiple container names or IDs
- Stops all specified containers in parallel

## How It Works

### Stop Process
1. **SIGTERM Signal** - Docker sends SIGTERM to main process
2. **Grace Period** - Waits for specified timeout (default 10 seconds)
3. **SIGKILL Signal** - If process doesn't exit, sends SIGKILL
4. **Container Stopped** - Container enters "exited" state

## Best Practices

1. **Use appropriate timeouts** for different application types
2. **Graceful shutdown** - Allow applications time to cleanup
3. **Database containers** - Use longer timeouts for data consistency
4. **Batch operations** - Stop multiple containers efficiently
5. **Health checks** - Verify services stopped properly
6. **Logging** - Check logs before stopping for debugging
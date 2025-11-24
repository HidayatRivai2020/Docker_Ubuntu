# stop

## Overview
- Stops one or more running containers gracefully
- Sends SIGTERM signal first, then SIGKILL after timeout
- Does not remove the container (use `docker rm` to delete)
- Container can be restarted later with `docker start`
- Essential for graceful application shutdown and maintenance
- Preserves container state and data for future use

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
- Processes containers independently

## How It Works

### Stop Process
1. **Signal Dispatch** - Docker sends SIGTERM to main container process
2. **Grace Period** - Waits for specified timeout (default 10 seconds)
3. **Force Termination** - If process doesn't exit, sends SIGKILL
4. **State Update** - Container enters "exited" state
5. **Resource Cleanup** - Releases network and temporary resources

### Process Flow
```
docker stop → Send SIGTERM → Wait Timeout → Still Running? → Yes → Send SIGKILL
                    ↓              ↓              ↓ No              ↓
            Process Receives   Grace Period   Container       Container
               Signal            Elapses        Stops           Stops
```

## Best Practices

- **Use appropriate timeouts** for different application types
- **Graceful shutdown** - Allow applications time to cleanup
- **Database containers** - Use longer timeouts for data consistency
- **Batch operations** - Stop multiple containers efficiently
- **Health checks** - Verify services stopped properly
- **Logging** - Check logs before stopping for debugging
- **Save state** - Commit changes if needed before stopping
- **Signal handling** - Ensure applications handle SIGTERM properly
# run

## Overview
- Creates and starts a new container from a Docker image
- Combines the functionality of `docker create` and `docker start` in a single command
- If image does not exist on host, it will pull the image from Docker Hub
- If the image exists on host, it will re-use the existing image
- If the tag is not provided, it will use the latest tag
- Run in a non interactive mode, Docker container does not listen to standard input


## Basic Syntax
```bash
docker run [OPTIONS] IMAGE:[TAG] [COMMAND] [ARG...]
```

## Common Options

### Basic Options
- `-d, --detach` - Run container in background and print container ID
- `-i, --interactive` - Keep STDIN open even if not attached
- `-t, --tty` - Allocate a pseudo-TTY
- `--name` - Assign a name to the container
- `--rm` - Automatically remove the container when it exits

### Port and Network Options
- `-p, --publish` - Publish container's port(s) to the host
- `--network` - Connect container to a network
- `--expose` - Expose a port or range of ports

### Volume and Mount Options
- `-v, --volume` - Bind mount a volume
- `--mount` - Attach a filesystem mount to the container

### Environment and Variables
- `-e, --env` - Set environment variables
- `--env-file` - Read environment variables from file

### Resource Limits
- `-m, --memory` - Memory limit
- `--cpus` - Number of CPUs
- `--cpu-shares` - CPU shares (relative weight)

## How It Works

### Container Lifecycle
1. **Image Check** - Verifies if image exists locally
2. **Image Pull** - Downloads image from registry if needed
3. **Container Creation** - Creates new container from image
4. **Container Start** - Starts the container process
5. **Command Execution** - Runs specified command or default entrypoint

### Process Flow
```
docker run → Image Available? → No → Pull Image
                    ↓ Yes           ↓
            Create Container ← Image Downloaded
                    ↓
            Start Container
                    ↓
            Execute Command
```

## Best Practices

- **Use specific image tags** instead of `latest` for reproducibility
- **Use `--rm` for temporary containers** to avoid accumulating stopped containers
- **Use meaningful names** with `--name` for long-running containers
- **Set resource limits** for production containers to prevent resource exhaustion
- **Use volumes** for persistent data storage
- **Avoid running as root** when possible (use `--user` option)
- **Use environment files** for complex configurations
- **Implement health checks** for production containers
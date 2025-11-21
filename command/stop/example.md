# Examples

## Basic Examples
```bash
# Stop a single container by name
docker stop my-nginx

# Stop a container by ID
docker stop a1b2c3d4e5f6

# Stop container with custom timeout
docker stop -t 30 my-app

# Stop container immediately (0 timeout)
docker stop -t 0 my-app
```

## Multiple Container Examples
```bash
# Stop multiple containers by name
docker stop web-server database redis-cache

# Stop multiple containers by ID
docker stop abc123 def456 ghi789

# Stop all running containers
docker stop $(docker ps -q)

# Stop containers with specific image
docker stop $(docker ps -q --filter ancestor=nginx)
```

## Conditional Examples
```bash
# Stop containers matching a pattern
docker stop $(docker ps --format "{{.Names}}" | grep web)

# Stop containers with specific status
docker stop $(docker ps -q --filter status=running)

# Stop containers created from specific image
docker stop $(docker ps -q --filter ancestor=myapp:latest)
```

## Common Use Cases

### Service Management
```bash
# Stop web server for maintenance
docker stop nginx-server

# Stop database before backup
docker stop mysql-db
```

### Development Workflow
```bash
# Stop development environment
docker stop $(docker ps -q --filter name=dev)

# Stop and restart service
docker stop myapp && docker start myapp
```

### Resource Management
```bash
# Stop all containers to free resources
docker stop $(docker ps -q)

# Stop memory-intensive containers
docker stop $(docker ps -q --filter ancestor=heavy-app)
```

### Deployment and Updates
```bash
# Stop old version before deploying new
docker stop app-v1

# Rolling restart of services
docker stop web-1 && docker start web-1
docker stop web-2 && docker start web-2
```

### Timeout Behavior
```bash
# Default timeout (10 seconds)
docker stop myapp

# Custom timeout (30 seconds)
docker stop -t 30 myapp

# No timeout (immediate SIGKILL)
docker stop -t 0 myapp

# Long timeout for graceful shutdown
docker stop -t 60 database
```

### Debugging Commands
```bash
# Check if container is still running
docker ps --filter name=myapp

# View container processes before stopping
docker top myapp

# Check container logs for shutdown messages
docker logs myapp

# Inspect container state
docker inspect myapp --format='{{.State.Status}}'

# Force kill if stop doesn't work
docker kill myapp
```
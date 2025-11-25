# Container Management

Docker container management encompasses the complete lifecycle of containers from creation to removal, including configuration, monitoring, and troubleshooting.

## Container Lifecycle

### States
- **Created**: Container exists but hasn't been started
- **Running**: Container is actively executing
- **Paused**: Container processes are suspended
- **Restarting**: Container is restarting after stop
- **Exited**: Container has stopped (successfully or with error)
- **Dead**: Container failed to stop properly

### State Transitions
```
Created → Running → Paused → Running → Exited
   ↓         ↓         ↓         ↓        ↓
   └─────────┴─────────┴─────────┴────────┘
                    Removed
```

## Creating Containers

### Basic Creation
```bash
# Create from image
docker create nginx

# Create with name
docker create --name web nginx

# Create with environment variables
docker create -e MYSQL_ROOT_PASSWORD=secret mysql

# Create with port mapping
docker create -p 8080:80 nginx

# Create with volume
docker create -v mydata:/data alpine

# Create with multiple options
docker create --name app \
  -p 8080:80 \
  -e NODE_ENV=production \
  -v /host/data:/app/data \
  --restart unless-stopped \
  myapp:latest
```

### Creation Options
- **--name**: Assign custom name to container
- **-p, --publish**: Publish container ports to host
- **-e, --env**: Set environment variables
- **-v, --volume**: Mount volumes or bind mounts
- **--network**: Connect to specific network
- **--restart**: Set restart policy
- **--user**: Run as specific user
- **--workdir**: Set working directory
- **--entrypoint**: Override default entrypoint
- **--memory**: Memory limit
- **--cpus**: CPU limit
- **--label**: Add metadata labels

## Running Containers

### Starting Containers
```bash
# Create and start in one command
docker run nginx

# Run in detached mode (background)
docker run -d nginx

# Run interactively with terminal
docker run -it ubuntu bash

# Run with automatic removal after exit
docker run --rm alpine echo "hello"

# Run with resource limits
docker run --memory="512m" --cpus="1.5" nginx

# Run with restart policy
docker run --restart=always nginx

# Run with health check
docker run --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  nginx
```

### Run Options
- **-d, --detach**: Run in background
- **-i, --interactive**: Keep STDIN open
- **-t, --tty**: Allocate pseudo-TTY
- **--rm**: Automatically remove container on exit
- **--name**: Container name
- **-p**: Port mapping (host:container)
- **-v**: Volume mount
- **-e**: Environment variable
- **--network**: Network to connect
- **--link**: Link to another container (deprecated)
- **--restart**: Restart policy (no, on-failure, always, unless-stopped)

### Interactive Mode
```bash
# Start bash shell
docker run -it ubuntu bash

# Run single command
docker run ubuntu echo "Hello World"

# Start with specific shell
docker run -it alpine /bin/sh

# Override entrypoint
docker run -it --entrypoint /bin/bash nginx
```

## Managing Running Containers

### Listing Containers
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List with custom format
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# List with size
docker ps -s

# List specific containers
docker ps --filter "name=web"
docker ps --filter "status=running"
docker ps --filter "label=env=production"

# Show latest created container
docker ps -l

# Show n last created containers
docker ps -n 5
```

### Container Control
```bash
# Start stopped container
docker start <container>

# Start multiple containers
docker start container1 container2

# Start and attach to container
docker start -a <container>

# Stop running container (SIGTERM)
docker stop <container>

# Stop with timeout
docker stop -t 30 <container>

# Force stop (SIGKILL)
docker kill <container>

# Restart container
docker restart <container>

# Pause container processes
docker pause <container>

# Unpause container
docker unpause <container>

# Wait for container to stop
docker wait <container>
```

### Container Inspection
```bash
# Detailed container information
docker inspect <container>

# Get specific field
docker inspect --format='{{.State.Status}}' <container>

# Get IP address
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container>

# Get all ports
docker inspect --format='{{.NetworkSettings.Ports}}' <container>

# Get environment variables
docker inspect --format='{{.Config.Env}}' <container>

# Get volumes
docker inspect --format='{{.Mounts}}' <container>
```

## Interacting with Containers

### Execute Commands
```bash
# Run command in running container
docker exec <container> ls /app

# Interactive shell
docker exec -it <container> bash

# Run as specific user
docker exec -u root <container> apt-get update

# Run in specific directory
docker exec -w /app <container> npm test

# Run with environment variable
docker exec -e VAR=value <container> env

# Run detached command
docker exec -d <container> service nginx reload
```

### Attach to Container
```bash
# Attach to container's console
docker attach <container>

# Attach with no stdin
docker attach --no-stdin <container>

# Attach and detach with Ctrl+P then Ctrl+Q
docker attach --sig-proxy=false <container>
```

### Copy Files
```bash
# Copy from container to host
docker cp <container>:/path/in/container /path/on/host

# Copy from host to container
docker cp /path/on/host <container>:/path/in/container

# Copy directory
docker cp <container>:/app/logs ./logs

# Follow symbolic links
docker cp -L <container>:/app/link ./file

# Archive mode (preserve permissions)
docker cp -a <container>:/data ./backup
```

## Container Logs

### Viewing Logs
```bash
# View logs
docker logs <container>

# Follow logs (tail -f)
docker logs -f <container>

# Show last N lines
docker logs --tail 100 <container>

# Show logs since timestamp
docker logs --since 2024-01-01T00:00:00 <container>

# Show logs until timestamp
docker logs --until 2024-01-02T00:00:00 <container>

# Show timestamps
docker logs -t <container>

# Follow with tail
docker logs -f --tail 50 <container>
```

### Log Drivers
- **json-file**: Default, JSON formatted logs
- **syslog**: Send to syslog daemon
- **journald**: Send to systemd journal
- **gelf**: Graylog Extended Log Format
- **fluentd**: Forward to Fluentd
- **awslogs**: AWS CloudWatch Logs
- **splunk**: Splunk logging
- **gcplogs**: Google Cloud Logging
- **none**: Disable logging

```bash
# Run with specific log driver
docker run --log-driver=syslog nginx

# Run with log options
docker run --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx
```

## Resource Management

### CPU Limits
```bash
# Limit CPU shares (relative weight)
docker run --cpu-shares=512 nginx

# Limit CPU cores
docker run --cpus=1.5 nginx

# Specific CPU cores
docker run --cpuset-cpus="0,1" nginx

# CPU quota and period
docker run --cpu-quota=50000 --cpu-period=100000 nginx
```

### Memory Limits
```bash
# Memory limit
docker run --memory=512m nginx

# Memory + swap limit
docker run --memory=512m --memory-swap=1g nginx

# Memory reservation (soft limit)
docker run --memory-reservation=256m nginx

# Disable OOM killer
docker run --oom-kill-disable nginx

# OOM score adjustment
docker run --oom-score-adj=500 nginx
```

### Storage Limits
```bash
# Storage driver options (overlay2)
docker run --storage-opt size=10G nginx

# Device read/write limits
docker run --device-read-bps=/dev/sda:10mb nginx
docker run --device-write-bps=/dev/sda:10mb nginx

# Device read/write IOPS
docker run --device-read-iops=/dev/sda:100 nginx
docker run --device-write-iops=/dev/sda:100 nginx
```

### Network Limits
```bash
# Not directly supported in Docker
# Use tc (traffic control) or third-party tools

# Example with network namespace
docker run --cap-add=NET_ADMIN nginx

# Limit bandwidth using tc inside container
docker exec <container> tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms
```

## Health Checks

### Defining Health Checks
```bash
# In Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

# At runtime
docker run --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  --health-start-period=5s \
  nginx

# Disable health check
docker run --no-healthcheck nginx
```

### Health Check Options
- **--interval**: Time between checks (default: 30s)
- **--timeout**: Maximum time for check (default: 30s)
- **--start-period**: Initialization time (default: 0s)
- **--retries**: Consecutive failures needed (default: 3)

### Checking Health Status
```bash
# View health status
docker inspect --format='{{.State.Health.Status}}' <container>

# View health log
docker inspect --format='{{json .State.Health}}' <container> | jq

# Filter unhealthy containers
docker ps --filter health=unhealthy

# Filter healthy containers
docker ps --filter health=healthy
```

## Restart Policies

### Policy Types
- **no**: Never restart (default)
- **on-failure[:max-retries]**: Restart only on failure
- **always**: Always restart regardless of exit status
- **unless-stopped**: Restart unless explicitly stopped

### Setting Restart Policies
```bash
# Never restart
docker run --restart=no nginx

# Restart on failure
docker run --restart=on-failure nginx

# Restart on failure with max retries
docker run --restart=on-failure:5 nginx

# Always restart
docker run --restart=always nginx

# Restart unless stopped
docker run --restart=unless-stopped nginx
```

### Updating Restart Policy
```bash
# Update existing container
docker update --restart=always <container>

# Update multiple containers
docker update --restart=always container1 container2
```

## Container Networking

### Network Modes
```bash
# Bridge network (default)
docker run --network=bridge nginx

# Host network
docker run --network=host nginx

# No network
docker run --network=none nginx

# Custom network
docker run --network=mynetwork nginx

# Container network
docker run --network=container:<container-name> nginx
```

### Port Mapping
```bash
# Map single port
docker run -p 8080:80 nginx

# Map to specific host IP
docker run -p 127.0.0.1:8080:80 nginx

# Map random host port
docker run -p 80 nginx

# Map multiple ports
docker run -p 8080:80 -p 8443:443 nginx

# Map UDP port
docker run -p 53:53/udp dns-server

# Publish all exposed ports
docker run -P nginx
```

### DNS Configuration
```bash
# Custom DNS server
docker run --dns=8.8.8.8 nginx

# Multiple DNS servers
docker run --dns=8.8.8.8 --dns=8.8.4.4 nginx

# DNS search domain
docker run --dns-search=example.com nginx

# Custom hostname
docker run --hostname=web01 nginx

# Add hosts entry
docker run --add-host=db:192.168.1.10 nginx
```

## Security and Isolation

### User and Permissions
```bash
# Run as specific user
docker run --user=1000:1000 nginx

# Run as user by name
docker run --user=nginx nginx

# Read-only root filesystem
docker run --read-only nginx

# Read-only with tmpfs for writable dirs
docker run --read-only --tmpfs /tmp --tmpfs /var/run nginx
```

### Capabilities
```bash
# Drop all capabilities
docker run --cap-drop=ALL nginx

# Add specific capability
docker run --cap-add=NET_ADMIN nginx

# Drop specific capability
docker run --cap-drop=CHOWN nginx

# Run privileged (all capabilities)
docker run --privileged nginx
```

### Security Options
```bash
# AppArmor profile
docker run --security-opt apparmor=docker-default nginx

# SELinux label
docker run --security-opt label=level:s0:c100,c200 nginx

# Seccomp profile
docker run --security-opt seccomp=profile.json nginx

# Disable seccomp
docker run --security-opt seccomp=unconfined nginx

# No new privileges
docker run --security-opt no-new-privileges nginx
```

### Process Isolation
```bash
# PID namespace mode
docker run --pid=host nginx        # Share host PID namespace
docker run --pid=container:<name> nginx  # Share another container's PID

# IPC namespace mode
docker run --ipc=host nginx        # Share host IPC
docker run --ipc=container:<name> nginx  # Share another container's IPC

# UTS namespace (hostname)
docker run --uts=host nginx        # Share host hostname
```

## Container Updates

### Updating Configuration
```bash
# Update restart policy
docker update --restart=always <container>

# Update memory limit
docker update --memory=1g <container>

# Update CPU limit
docker update --cpus=2 <container>

# Update multiple containers
docker update --memory=512m container1 container2

# Update all running containers
docker update --restart=unless-stopped $(docker ps -q)
```

### Updating Images
```bash
# Pull latest image
docker pull nginx:latest

# Stop container
docker stop web

# Remove container
docker rm web

# Run with new image
docker run -d --name web nginx:latest
```

## Cleaning Up

### Removing Containers
```bash
# Remove stopped container
docker rm <container>

# Force remove running container
docker rm -f <container>

# Remove multiple containers
docker rm container1 container2

# Remove with volumes
docker rm -v <container>

# Remove all stopped containers
docker container prune

# Remove all containers (including running)
docker rm -f $(docker ps -aq)
```

### Pruning Resources
```bash
# Remove stopped containers
docker container prune

# Remove containers stopped more than 24h ago
docker container prune --filter "until=24h"

# Remove without confirmation
docker container prune -f

# Remove all stopped containers and their volumes
docker container prune --volumes
```

## Container Statistics

### Real-time Monitoring
```bash
# View all containers stats
docker stats

# View specific containers
docker stats web db cache

# No streaming (single output)
docker stats --no-stream

# Custom format
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Show only container names and IDs
docker stats --format "{{.Name}}: {{.ID}}"
```

### Process Information
```bash
# View processes in container
docker top <container>

# View with custom ps options
docker top <container> aux

# View with specific format
docker top <container> -eo pid,user,args
```

## Container Events

### Monitoring Events
```bash
# Stream all events
docker events

# Filter by container
docker events --filter container=web

# Filter by event type
docker events --filter event=start
docker events --filter event=stop
docker events --filter event=die

# Filter by time
docker events --since 2024-01-01
docker events --until 2024-01-02

# Filter multiple criteria
docker events --filter container=web --filter event=start

# Format output
docker events --format '{{.Time}}: {{.Action}} - {{.Actor.Attributes.name}}'
```

## Docker Compose Container Management

### Lifecycle Commands
```yaml
# docker-compose.yml example
version: '3.8'
services:
  web:
    image: nginx
    restart: unless-stopped
    ports:
      - "8080:80"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 3s
      retries: 3
```

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose stop

# Restart services
docker-compose restart

# Pause services
docker-compose pause

# Unpause services
docker-compose unpause

# Remove stopped containers
docker-compose rm

# Scale services
docker-compose up -d --scale web=3

# View logs
docker-compose logs -f

# Execute command
docker-compose exec web bash

# View processes
docker-compose top
```

## Best Practices

### Naming
- **Use Descriptive Names**: Name containers based on their function
- **Consistent Naming**: Follow naming conventions across environments
- **Avoid Generic Names**: Don't use names like "test" or "container1"
- **Include Environment**: Add env prefix for multi-environment setups

### Resource Management
- **Set Memory Limits**: Prevent containers from consuming all memory
- **Set CPU Limits**: Ensure fair CPU distribution
- **Use Health Checks**: Monitor application health
- **Configure Restart Policies**: Automatic recovery from failures

### Security
- **Run as Non-Root**: Use --user flag
- **Drop Capabilities**: Remove unnecessary privileges
- **Read-Only Filesystem**: When possible
- **Limit Resources**: Prevent DoS attacks
- **Regular Updates**: Keep images up to date

### Monitoring
- **Log Everything**: Enable proper logging
- **Monitor Resources**: Use docker stats
- **Set Up Alerts**: Monitor unhealthy containers
- **Track Events**: Log container lifecycle events

### Cleanup
- **Remove Unused Containers**: Regular pruning
- **Use --rm Flag**: For temporary containers
- **Clean Up Volumes**: Remove unused volumes
- **Automate Cleanup**: Schedule regular maintenance

## Troubleshooting

### Container Won't Start
```bash
# Check logs
docker logs <container>

# Inspect container
docker inspect <container>

# Check exit code
docker inspect --format='{{.State.ExitCode}}' <container>

# Try running interactively
docker run -it <image> bash

# Check image
docker history <image>
```

### Container Crashes
```bash
# View crash logs
docker logs <container>

# Check restart count
docker inspect --format='{{.RestartCount}}' <container>

# Check OOM killed
docker inspect --format='{{.State.OOMKilled}}' <container>

# Increase memory limit
docker run --memory=1g <image>

# Check health status
docker inspect --format='{{.State.Health.Status}}' <container>
```

### Performance Issues
```bash
# Monitor resources
docker stats <container>

# Check processes
docker top <container>

# Review resource limits
docker inspect --format='{{.HostConfig.Memory}}' <container>

# Check storage driver
docker info | grep "Storage Driver"

# Monitor I/O
docker stats --format "table {{.Container}}\t{{.BlockIO}}"
```

### Networking Issues
```bash
# Check IP address
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container>

# Check port mappings
docker port <container>

# Test connectivity
docker exec <container> ping google.com

# Check DNS
docker exec <container> nslookup google.com

# View network
docker network inspect <network>
```

### Access Issues
```bash
# Check running user
docker exec <container> whoami

# Check file permissions
docker exec <container> ls -la /path

# Enter as root
docker exec -u root -it <container> bash

# Check capabilities
docker exec <container> capsh --print
```

## Advanced Topics

### Container Commit
```bash
# Create image from container
docker commit <container> myimage:v1

# Commit with message and author
docker commit -m "Added config" -a "John Doe" <container> myimage:v1

# Commit with changes
docker commit --change='CMD ["nginx", "-g", "daemon off;"]' <container> mynginx
```

### Container Export/Import
```bash
# Export container filesystem
docker export <container> > container.tar

# Import as image
docker import container.tar myimage:imported

# Export to file
docker export -o container.tar <container>
```

### Container Checkpoints (Experimental)
```bash
# Create checkpoint
docker checkpoint create <container> checkpoint1

# List checkpoints
docker checkpoint ls <container>

# Start from checkpoint
docker start --checkpoint=checkpoint1 <container>

# Remove checkpoint
docker checkpoint rm <container> checkpoint1
```

### Init Process
```bash
# Use tini as init
docker run --init nginx

# Prevents zombie processes
# Properly handles signals
# Recommended for production containers
```

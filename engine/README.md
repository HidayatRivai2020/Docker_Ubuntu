# Docker Engine
- Docker Engine is an open source containerization technology for building and containerizing applications.
- **Client-Server Application**: Acts as a client-server architecture with multiple components
- **Containerization Technology**: Builds and runs containerized applications
- **Open Source**: Available under Apache License, Version 2.0
- **Core Runtime**: Foundation for Docker Desktop and standalone Docker installations

## Components

### Server (dockerd)
- **Long-running Daemon Process**: Background service that manages Docker objects
- **Creates and Manages**: Images, containers, networks, and volumes
- **Listens for API Requests**: Processes requests from Docker CLI and other clients
- **Resource Management**: Allocates and controls system resources for containers

### APIs
- **RESTful Interface**: HTTP-based API for programmatic access
- **Communication Layer**: Enables programs to interact with Docker daemon
- **Integration**: Allows third-party tools and applications to use Docker
- **Unix Socket**: Default communication via `/var/run/docker.sock`
- **TCP Socket**: Optional remote access configuration

### CLI Client (docker)
- **Command Line Interface**: Primary user interface for Docker
- **Uses Docker APIs**: Communicates with daemon through REST API
- **Scripting Support**: Automate Docker operations with shell scripts
- **Direct Commands**: Execute Docker operations interactively
- **Remote Capable**: Can connect to remote Docker daemons

## Key Features

### [Container Management](feature/container_management.md)
- **Lifecycle Control**: Create, start, stop, restart, pause, and remove containers
- **Process Isolation**: Separate process namespaces for security
- **Resource Limits**: CPU, memory, and I/O constraints
- **Health Checks**: Monitor application health inside containers
- **Auto-restart Policies**: Automatic recovery from failures

### Image Management
- **Build System**: Create images from Dockerfiles
- **Layer Caching**: Optimize build times with intelligent caching
- **Local Registry**: Store and manage images locally
- **Registry Integration**: Push/pull images from Docker Hub and private registries
- **Multi-architecture**: Support for different CPU architectures

### [Networking](feature/networking.md)
- **Bridge Networks**: Default isolated network per container
- **Host Networking**: Direct access to host network stack
- **Overlay Networks**: Multi-host networking for distributed applications
- **Custom Networks**: User-defined networks with built-in DNS
- **Port Mapping**: Expose container services to external networks
- **Network Drivers**: Pluggable network backend support

### [Storage](feature/storage.md)
- **Volume Management**: Persistent data storage independent of container lifecycle
- **Bind Mounts**: Direct host filesystem access
- **tmpfs Mounts**: In-memory storage for temporary data
- **Storage Drivers**: Pluggable backends (overlay2, btrfs, zfs)
- **Layer Management**: Efficient storage through image layering
- **Copy-on-Write**: Optimize disk usage and performance

### Security
- **Namespace Isolation**: Process, network, and filesystem isolation
- **Control Groups (cgroups)**: Resource limitation and accounting
- **Seccomp Profiles**: System call filtering
- **AppArmor/SELinux**: Mandatory access control
- **Capabilities**: Fine-grained privilege management
- **User Namespaces**: Map container root to non-privileged host users
- **Content Trust**: Image signature verification
- **Rootless Mode**: Run Docker daemon without root privileges

## Architecture

```
┌─────────────────────────────────────────────┐
│         Docker CLI (docker command)         │
│              User Interface                  │
└──────────────────┬──────────────────────────┘
                   │ REST API
                   ▼
┌─────────────────────────────────────────────┐
│          Docker Daemon (dockerd)            │
│                                             │
│  ┌─────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Images  │  │Containers│  │ Networks │  │
│  └─────────┘  └──────────┘  └──────────┘  │
│  ┌─────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Volumes │  │  Build   │  │ Plugins  │  │
│  └─────────┘  └──────────┘  └──────────┘  │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│              containerd                      │
│     High-level Container Runtime            │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│                 runc                         │
│      Low-level Container Runtime            │
│         (OCI Runtime Specification)         │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│         Linux Kernel Features               │
│   Namespaces • cgroups • Capabilities       │
└─────────────────────────────────────────────┘
```

## How Docker Engine Works

### Container Lifecycle

1. **Image Pull**: Daemon downloads image layers from registry if not cached locally
2. **Container Creation**: Creates container from image with specified configuration
3. **Namespace Setup**: Isolates processes, network, filesystem, and IPC
4. **Resource Allocation**: Assigns CPU shares, memory limits, I/O bandwidth
5. **Network Configuration**: Connects container to specified networks
6. **Volume Mounting**: Attaches persistent storage and bind mounts
7. **Process Start**: Launches container's main process via containerd and runc
8. **Monitoring**: Tracks container state, health, and resource usage
9. **Log Collection**: Captures stdout/stderr from container processes
10. **Container Stop**: Sends SIGTERM, then SIGKILL if necessary
11. **Cleanup**: Removes container layer and network connections
12. **Resource Release**: Frees allocated CPU, memory, and storage

### Build Process

1. **Parse Dockerfile**: Reads and validates build instructions
2. **Pull Base Image**: Downloads FROM image if not cached
3. **Execute Instructions**: Runs each command in temporary container
4. **Create Layers**: Commits filesystem changes as new image layers
5. **Cache Layers**: Stores layers for future build optimization
6. **Tag Image**: Assigns name and tag to final image
7. **Push (Optional)**: Uploads image to registry

## Docker Objects

### Images
- **Read-only Templates**: Blueprint for creating containers
- **Layered Filesystem**: Built from multiple stacked layers
- **Portable**: Can be shared via registries
- **Versioned**: Tagged with versions for tracking

### Containers
- **Runnable Instances**: Created from images
- **Isolated**: Own process space, network, and filesystem
- **Ephemeral**: Can be easily created and destroyed
- **Writable Layer**: Changes stored in thin writable layer

### Networks
- **Connectivity**: Enable container communication
- **Isolation**: Separate network namespaces
- **DNS Resolution**: Built-in service discovery
- **Drivers**: bridge, host, overlay, macvlan, none

### Volumes
- **Persistent Storage**: Data survives container deletion
- **Shared Data**: Multiple containers can access same volume
- **Managed by Docker**: Stored in Docker's storage area
- **Portable**: Can be backed up and migrated

## Configuration

### Daemon Configuration
- **Location**: `/etc/docker/daemon.json` (Linux), `%programdata%\docker\config\daemon.json` (Windows)
- **Format**: JSON configuration file
- **Settings**: Storage driver, log driver, registry mirrors, DNS, resource limits
- **Reload**: Restart daemon to apply changes

### Common Options
- **Storage Driver**: `"storage-driver": "overlay2"`
- **Log Driver**: `"log-driver": "json-file"`
- **Log Options**: `"log-opts": {"max-size": "10m", "max-file": "3"}`
- **Registry Mirrors**: `"registry-mirrors": ["https://mirror.example.com"]`
- **Insecure Registries**: `"insecure-registries": ["registry.local:5000"]`
- **Live Restore**: `"live-restore": true`
- **Userland Proxy**: `"userland-proxy": false`

## Managing Docker Engine

### Service Management (Linux)
```bash
# Start Docker daemon
sudo systemctl start docker

# Stop Docker daemon
sudo systemctl stop docker

# Restart Docker daemon
sudo systemctl restart docker

# Enable on boot
sudo systemctl enable docker

# Check status
sudo systemctl status docker

# View logs
sudo journalctl -u docker
```

### Daemon Information
```bash
# Docker version
docker version

# System-wide information
docker info

# Real-time events
docker events

# Disk usage
docker system df
```

## Performance Optimization

### Storage Optimization
- **Use overlay2**: Best performance on modern kernels
- **Regular Pruning**: Remove unused images and containers
- **Volume Drivers**: Choose appropriate driver for workload

### Network Optimization
- **Disable Userland Proxy**: Better performance with `iptables`
- **Use Host Network**: When isolation not required
- **Custom Networks**: Optimize DNS and routing

### Resource Management
- **Set Limits**: Prevent resource starvation
- **Use cgroups**: Fair resource distribution
- **Monitor Usage**: Track with `docker stats`

### Build Optimization
- **Layer Caching**: Order instructions by change frequency
- **Multi-stage Builds**: Reduce final image size
- **.dockerignore**: Exclude unnecessary files
- **Minimize Layers**: Combine related commands

## Monitoring and Logging

### Container Monitoring
- **docker stats**: Real-time resource usage
- **docker top**: Process list inside container
- **docker inspect**: Detailed container information
- **Health Checks**: Application-level monitoring

### Logging
- **Log Drivers**: json-file, syslog, journald, gelf, fluentd
- **Log Rotation**: Prevent disk space exhaustion
- **Centralized Logging**: Forward to logging systems
- **Debug Mode**: Enable with `"debug": true`

## Security Best Practices

### Daemon Security
- **Run as Non-root**: Use rootless mode when possible
- **Enable Content Trust**: Verify image signatures
- **Limit Socket Access**: Restrict `/var/run/docker.sock` permissions
- **Use TLS**: Encrypt remote API access

### Container Security
- **Non-root User**: Run processes as non-privileged user
- **Read-only Filesystem**: Use `--read-only` flag
- **Drop Capabilities**: Remove unnecessary privileges
- **Resource Limits**: Prevent DoS attacks
- **Security Scanning**: Check images for vulnerabilities

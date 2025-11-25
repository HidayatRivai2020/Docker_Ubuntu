# Storage

Docker storage manages how data is persisted, shared, and accessed by containers and the host system.

## Storage Types

### Volumes
- **Docker-Managed Storage**: Stored in Docker's storage directory (`/var/lib/docker/volumes/`)
- **Preferred Method**: Recommended for persistent data in production
- **Portable**: Can be easily backed up, migrated, and shared
- **Named or Anonymous**: Can have explicit names or auto-generated IDs
- **Driver Support**: Pluggable volume drivers for various backends
- **Container Independent**: Persist beyond container lifecycle
- **Sharing**: Multiple containers can mount the same volume
- **Pre-populated**: Volume initialized with container's directory contents

**Create and Use Volumes:**
```bash
# Create named volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Use volume in container
docker run -v mydata:/app/data myapp

# Remove volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune
```

**Volume Location:**
- Linux: `/var/lib/docker/volumes/<volume-name>/_data`
- Windows: `C:\ProgramData\Docker\volumes\<volume-name>\_data`

### Bind Mounts
- **Host Path Mapping**: Direct mapping of host filesystem path into container
- **Full Path Required**: Must specify absolute path on host
- **Development Use**: Ideal for development with live code changes
- **Performance**: Native filesystem performance
- **File Permissions**: Uses host file permissions and ownership
- **No Docker Management**: Files managed outside Docker
- **Bidirectional Sync**: Changes reflect immediately in both directions
- **Security Consideration**: Container can modify host files

**Use Bind Mounts:**
```bash
# Mount host directory to container
docker run -v /host/path:/container/path myapp

# Mount with read-only access
docker run -v /host/path:/container/path:ro myapp

# Using --mount syntax (more explicit)
docker run --mount type=bind,source=/host/path,target=/container/path myapp

# Bind mount with specific options
docker run --mount type=bind,source=/host/path,target=/container/path,readonly myapp
```

**Use Cases:**
- Sharing source code during development
- Sharing configuration files from host
- Sharing build artifacts between stages
- Accessing host sockets (e.g., `/var/run/docker.sock`)

### tmpfs Mounts
- **Memory Storage**: Data stored in host system's memory (RAM)
- **Temporary**: Data deleted when container stops
- **Fast Performance**: RAM is faster than disk I/O
- **Security**: Sensitive data never written to disk
- **Linux Only**: Not available on Windows containers
- **No Persistence**: Lost on container stop or host reboot
- **Size Limits**: Limited by available RAM

**Use tmpfs Mounts:**
```bash
# Create tmpfs mount
docker run --tmpfs /app/temp myapp

# Specify size limit
docker run --tmpfs /app/temp:size=100m myapp

# Using --mount syntax
docker run --mount type=tmpfs,target=/app/temp,tmpfs-size=100m myapp
```

**Use Cases:**
- Temporary processing of sensitive data
- Cache directories
- Session data
- Temporary build artifacts

## Storage Drivers

Storage drivers control how image layers and container layers are stored and managed on the host.

### overlay2
- **Default Driver**: Recommended for most use cases
- **Kernel Support**: Requires Linux kernel 4.0+
- **Performance**: Excellent read/write performance
- **Efficiency**: Efficient use of disk space and inodes
- **Stability**: Production-ready and well-tested
- **Copy-on-Write**: Fast CoW operations
- **Native Support**: Built into modern kernels

**Configuration:**
```json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

### aufs
- **Legacy Driver**: Deprecated, for backward compatibility only
- **Ubuntu Support**: Previously default on older Ubuntu versions
- **Layer Limit**: Limited number of layers (42 max)
- **Performance**: Good read performance, slower writes
- **Status**: No longer maintained for new kernels
- **Migration**: Should migrate to overlay2

### devicemapper
- **Legacy Driver**: Used on older RHEL/CentOS systems
- **Complex Setup**: Requires direct-lvm configuration
- **Performance**: Moderate performance
- **Production Use**: Requires careful configuration
- **Status**: Deprecated in favor of overlay2
- **Block Device**: Uses block devices for storage

### btrfs
- **Filesystem Required**: Host must use Btrfs filesystem
- **Advanced Features**: Snapshots, compression, checksums
- **Performance**: Good for many small writes
- **CoW Support**: Native copy-on-write
- **Complexity**: More complex management
- **Use Case**: Specific Btrfs requirements

### zfs
- **Filesystem Required**: Host must use ZFS filesystem
- **Enterprise Features**: Snapshots, replication, compression
- **Memory Usage**: High memory requirements
- **Performance**: Excellent for I/O intensive workloads
- **Complexity**: Complex configuration
- **Use Case**: Enterprise storage requirements

### vfs
- **No Optimization**: No copy-on-write or layering
- **Simple**: Direct filesystem copy
- **Performance**: Very slow, high disk usage
- **Use Case**: Testing only, not for production
- **Compatibility**: Works anywhere but inefficient

**Check Current Driver:**
```bash
docker info | grep "Storage Driver"
```

## Storage Architecture

### [Layered Architecture](storage/layered_architecture.md)
- **Layer Structure**: Images built from multiple read-only layers stacked on top of each other
- **Base Layer**: Starting point from base image (e.g., FROM ubuntu:22.04)
- **Instruction Layers**: Each Dockerfile RUN, COPY, ADD creates new layer
- **Layer Identification**: Each layer has unique SHA256 hash
- **Layer Sharing**: Multiple images share common base layers for efficiency
- **Layer Caching**: Docker reuses unchanged layers during builds
- **Size Optimization**: Chain commands in single RUN to reduce layers
- **Immutability**: Once created, layers never change
- **Multi-Stage Builds**: Create smaller final images by copying only needed files

### Image Layers
- **Read-Only Layers**: Each image consists of multiple immutable layers
- **Stacked Structure**: Layers stacked from bottom (base) to top (latest)
- **Shared Layers**: Common layers shared between images
- **Layer Reuse**: Reduces disk usage and speeds up pulls
- **Content Addressable**: Identified by SHA256 hash
- **Cacheable**: Build cache based on layer content
- **Viewing**: Use `docker history` and `docker image inspect` to analyze layers

### Container Layer
- **Writable Layer**: Thin layer added on top of image layers
- **Runtime Changes**: All container modifications stored here
- **Copy-on-Write**: Files copied from lower layers on first write
- **Ephemeral**: Deleted when container removed
- **Performance**: Slower than volumes for heavy I/O
- **Isolation**: Each container has its own independent layer
- **Not Shared**: Container layers unique to each container

### Union Filesystem
- **Unified View**: Presents multiple layers as single filesystem
- **Layer Merging**: Combines read-only and writable layers
- **Overlay**: Upper layers override lower layers
- **Transparent**: Application sees unified filesystem
- **Efficient**: No physical merging, just logical view

## Copy-on-Write (CoW)

### How CoW Works

1. **Read Operation**: File read from lowest layer containing it
2. **First Write**: File copied from image layer to container layer
3. **Subsequent Writes**: Modifications made to copied file
4. **Delete Operation**: Whiteout file created in container layer
5. **New Files**: Created directly in container layer

### CoW Benefits
- **Space Efficiency**: Multiple containers share image layers
- **Fast Container Start**: No need to copy entire image
- **Layer Sharing**: Common layers stored once
- **Memory Efficiency**: Shared pages in memory

### CoW Performance
- **Read Performance**: Fast, reads from original layer
- **First Write**: Slower, requires copy operation
- **Large Files**: Significant overhead on first write
- **Optimization**: Use volumes for heavy write workloads

## Storage Best Practices

### Data Persistence
- **Use Volumes**: For all persistent data in production
- **Named Volumes**: Easier to identify and manage
- **Volume Drivers**: Use appropriate driver for workload
- **Backup Volumes**: Regular backups of important data
- **Avoid Container Layer**: Don't store important data in container layer

### Performance
- **Volumes for I/O**: Use volumes for heavy write operations
- **overlay2 Driver**: Use overlay2 for best overall performance
- **tmpfs for Temp**: Use tmpfs for temporary high-speed data
- **Minimize Layers**: Combine Dockerfile commands to reduce layers
- **Order Layers**: Put frequently changing layers last

### Security
- **Read-Only Volumes**: Mount volumes as read-only when possible
- **Limited Bind Mounts**: Minimize host filesystem exposure
- **Volume Permissions**: Set appropriate file permissions
- **Sensitive Data**: Use tmpfs for sensitive temporary data
- **Regular Cleanup**: Remove unused volumes and images

### Management
- **Label Volumes**: Use labels for organization
- **Volume Pruning**: Regular cleanup of unused volumes
- **Monitor Usage**: Track disk space consumption
- **Backup Strategy**: Implement volume backup procedures
- **Documentation**: Document volume purposes and contents

## Storage Commands

### Volume Management
```bash
# Create volume
docker volume create <name>

# List volumes
docker volume ls

# Inspect volume
docker volume inspect <name>

# Remove volume
docker volume rm <name>

# Prune unused volumes
docker volume prune

# Remove all volumes
docker volume rm $(docker volume ls -q)
```

### Container Storage Options
```bash
# Volume mount
docker run -v myvolume:/data myapp

# Bind mount
docker run -v /host/path:/container/path myapp

# Read-only mount
docker run -v myvolume:/data:ro myapp

# tmpfs mount
docker run --tmpfs /tmp myapp

# Multiple mounts
docker run -v vol1:/data1 -v vol2:/data2 myapp
```

### Storage Information
```bash
# Disk usage
docker system df

# Detailed disk usage
docker system df -v

# Container sizes
docker ps -s

# Image sizes
docker images

# Volume location
docker volume inspect <name> --format '{{ .Mountpoint }}'
```

## Docker Compose Storage

### Volume Declaration
```yaml
version: '3.8'

services:
  app:
    image: myapp
    volumes:
      # Named volume
      - mydata:/app/data
      
      # Bind mount
      - ./config:/app/config
      
      # Read-only bind mount
      - ./secrets:/app/secrets:ro
      
      # tmpfs mount
      - type: tmpfs
        target: /app/temp
        tmpfs:
          size: 100m

volumes:
  # Define named volumes
  mydata:
    driver: local
    driver_opts:
      type: none
      device: /path/on/host
      o: bind
```

### Volume Options
```yaml
volumes:
  # External volume (already exists)
  existing_volume:
    external: true
  
  # Named with driver options
  custom_volume:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.1,rw
      device: ":/path/to/dir"
  
  # With labels
  labeled_volume:
    driver: local
    labels:
      com.example.description: "Database volume"
      com.example.team: "backend"
```

## Storage Troubleshooting

### Volume Issues

**Volume Not Found:**
```bash
# List all volumes
docker volume ls

# Inspect volume
docker volume inspect <name>

# Check if volume is external
docker volume ls --filter "dangling=false"
```

**Permission Denied:**
```bash
# Check volume permissions
docker volume inspect <name> --format '{{ .Mountpoint }}'
ls -la /var/lib/docker/volumes/<name>/_data

# Run container as specific user
docker run --user $(id -u):$(id -g) -v myvolume:/data myapp

# Fix permissions in volume
docker run --rm -v myvolume:/data alpine chown -R 1000:1000 /data
```

**Volume Full:**
```bash
# Check disk usage
docker system df
df -h /var/lib/docker

# Clean up unused volumes
docker volume prune

# Remove specific volume
docker volume rm <name>
```

### Storage Driver Issues

**Check Current Driver:**
```bash
docker info | grep "Storage Driver"
docker info | grep "Backing Filesystem"
```

**Driver Compatibility:**
```bash
# Check kernel version (for overlay2)
uname -r

# Check filesystem type
df -T /var/lib/docker

# Test driver change
sudo systemctl stop docker
# Edit /etc/docker/daemon.json
sudo systemctl start docker
```

**Storage Corruption:**
```bash
# Stop Docker
sudo systemctl stop docker

# Check filesystem
sudo fsck /dev/<device>

# Clear Docker storage (WARNING: removes all data)
sudo rm -rf /var/lib/docker

# Restart Docker
sudo systemctl start docker
```

### Performance Issues

**Slow I/O:**
- Use volumes instead of container layer for heavy writes
- Consider using tmpfs for temporary high-speed storage
- Check storage driver performance characteristics
- Monitor with `docker stats` and `iostat`

**High Disk Usage:**
```bash
# Check what's using space
docker system df -v

# Clean up
docker system prune -a --volumes

# Remove specific items
docker image prune -a
docker container prune
docker volume prune
```

## Advanced Storage

### Volume Plugins
- **Third-Party Drivers**: NFS, GlusterFS, Ceph, AWS EBS, Azure Disk
- **Installation**: Install plugin via `docker plugin install`
- **Configuration**: Specify driver in volume creation
- **Use Cases**: Shared storage, cloud storage, enterprise storage systems

### Storage Backends
- **Local**: Default, uses host filesystem
- **NFS**: Network File System for shared storage
- **Cloud**: AWS EFS, Azure Files, Google Cloud Filestore
- **Distributed**: GlusterFS, Ceph, Longhorn
- **Block Storage**: AWS EBS, Azure Disk, GCE Persistent Disk

### Data Migration
```bash
# Backup volume to tar
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine \
  tar czf /backup/myvolume-backup.tar.gz -C /data .

# Restore volume from tar
docker run --rm -v newvolume:/data -v $(pwd):/backup alpine \
  tar xzf /backup/myvolume-backup.tar.gz -C /data

# Copy between volumes
docker run --rm -v source:/source -v dest:/dest alpine \
  cp -av /source/. /dest/
```

## Storage Monitoring

### Metrics to Track
- **Disk Usage**: Total space used by Docker
- **Volume Count**: Number of volumes created
- **Layer Count**: Number of image layers
- **I/O Performance**: Read/write throughput
- **Inode Usage**: Available inodes on filesystem

### Monitoring Commands
```bash
# Overall disk usage
docker system df

# Detailed usage by type
docker system df -v

# Container sizes
docker ps -s

# Image layer sizes
docker history <image>

# Volume locations and sizes
docker volume ls
du -sh /var/lib/docker/volumes/*

# Real-time I/O stats
docker stats --no-stream

# Storage driver info
docker info | grep -A 10 "Storage Driver"
```

### Alerting Thresholds
- **Disk Usage**: Alert when >80% full
- **Inode Usage**: Alert when >80% used
- **Volume Growth**: Track unexpected size increases
- **Stale Volumes**: Identify unused volumes
- **Failed Mounts**: Monitor mount errors in logs

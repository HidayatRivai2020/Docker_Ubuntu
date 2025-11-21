# How to Check Docker Version on Ubuntu

## Check Docker Engine Version

### Basic Version Check
```bash
docker --version
```

### Detailed Version Information
```bash
docker version
```

This command shows:
- Client version information
- Server (Docker Engine) version information
- API version
- Go version used to build Docker
- Git commit information
- Build time
- OS/Architecture information

### System-wide Docker Information
```bash
docker info
```

This command provides comprehensive system information including:
- Docker version details
- Storage driver information
- Number of containers and images
- System resources (CPUs, memory)
- Docker daemon configuration
- Registry information
- Plugin information

## Check Docker Compose Version

### Docker Compose Plugin Version
```bash
docker compose version
```

### Legacy Docker Compose Version (if installed)
```bash
docker-compose --version
```

## Check Docker Buildx Version
```bash
docker buildx version
```

## Check All Docker Components
```bash
docker version && echo "" && docker compose version && echo "" && docker buildx version
```

## Check if Docker is Running
```bash
sudo systemctl status docker
```

Alternative method:
```bash
docker ps
```

## Troubleshooting

### Permission Denied Error
If you get permission denied when running Docker commands:
```bash
sudo docker --version
```

### Docker Not Found
If Docker is not found, verify installation:
```bash
which docker
```

### Check Docker Service Status
```bash
sudo systemctl is-active docker
sudo systemctl is-enabled docker
```
# Docker Compose
- Tool for defining and running multi-container Docker applications.
- Using a YAML file to configure your application's services, networks, and volumes
- Create and start all services with a single command.

## Key Benefits
- **Single Configuration File**: Define entire application stack in one place
- **Simple Commands**: Start/stop all services with one command
- **Service Dependencies**: Automatically manage container startup order
- **Network Isolation**: Automatic network creation for service communication
- **Reproducibility**: Consistent environments across different machines
- **Version Control**: Track infrastructure changes in source control

## Basic Syntax

### docker-compose.yml Structure

```yaml
version: '3.8'  # Compose file version (optional in newer versions)

services:       # Define containers
  service_name:
    image: image_name:tag
    # or
    build: ./path/to/dockerfile
    
networks:       # Define networks (optional)
  network_name:
    driver: bridge

volumes:        # Define volumes (optional)
  volume_name:
```

## Environment Variables
- Create `.env` file for storing configuration and secrets
- Use `${VARIABLE:-default}` syntax for defaults in docker-compose.yml
- Specify custom env file with `--env-file` flag
- Add `.env` to `.gitignore` to protect secrets

## Best Practices
- **One Service Per Container**: Follow single responsibility principle
- **Named Volumes**: Use named volumes for persistent data
- **Environment Files**: Store secrets in `.env` files (add to `.gitignore`)
- **Network Segmentation**: Use multiple networks for service isolation
- **Health Checks**: Define health checks for critical services

## Migration from docker-compose v1

```bash
# Old command (v1)
docker-compose up -d

# New command (v2)
docker compose up -d
```

**Key Differences:**
- Command changed from `docker-compose` (hyphen) to `docker compose` (space)
- Compose v2 is a Docker CLI plugin
- Faster performance and better integration
- Some minor syntax changes in compose file


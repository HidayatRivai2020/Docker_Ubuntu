# Examples

## Basic Examples
```bash
# Run a simple hello-world container
docker run hello-world

# Run Ubuntu container with interactive shell
docker run -it ubuntu bash

# Run container in background
docker run -d nginx

# Run container with custom name
docker run --name my-nginx nginx
```

## Port Mapping Examples
```bash
# Map container port 80 to host port 8080
docker run -p 8080:80 nginx

# Map container port 80 to random host port
docker run -P nginx

# Map multiple ports
docker run -p 8080:80 -p 8443:443 nginx
```

## Volume Examples
```bash
# Bind mount host directory to container
docker run -v /host/path:/container/path ubuntu

# Create and mount named volume
docker run -v my-volume:/data ubuntu

# Mount current directory
docker run -v $(pwd):/app ubuntu
```

## Environment Variables
```bash
# Set single environment variable
docker run -e MYSQL_ROOT_PASSWORD=secret mysql

# Set multiple environment variables
docker run -e VAR1=value1 -e VAR2=value2 ubuntu

# Load environment from file
docker run --env-file .env ubuntu
```

## Advanced Examples
```bash
# Run with resource limits
docker run -m 512m --cpus="1.5" ubuntu

# Run with restart policy
docker run --restart=always nginx

# Run with custom entrypoint
docker run --entrypoint="" ubuntu echo "Hello World"

# Run with working directory
docker run -w /app ubuntu pwd
```

## Common Use Cases

### Web Servers
```bash
# Run web server with port mapping
docker run -d -p 80:80 --name web-server nginx

# Run with custom configuration
docker run -d -p 80:80 -v /host/config:/etc/nginx nginx
```

### Database Containers
```bash
# Run MySQL with persistent data
docker run -d -e MYSQL_ROOT_PASSWORD=secret -v mysql-data:/var/lib/mysql mysql

# Run PostgreSQL with environment file
docker run -d --env-file db.env -v postgres-data:/var/lib/postgresql/data postgres
```

### Development Environment
```bash
# Interactive development container
docker run -it -v $(pwd):/workspace ubuntu bash

# Run development server
docker run -d -p 3000:3000 -v $(pwd):/app node:16 npm start
```

### One-time Tasks
```bash
# Run and auto-remove after completion
docker run --rm alpine echo "Hello World"

# Run backup script
docker run --rm -v /data:/backup ubuntu tar -czf /backup/backup.tar.gz /data
```

## Security Considerations

### User and Permissions
```bash
# Run as non-root user
docker run --user 1001:1001 ubuntu

# Run with read-only filesystem
docker run --read-only ubuntu

# Drop capabilities
docker run --cap-drop ALL ubuntu
```

### Network Security
```bash
# Run without network access
docker run --network none ubuntu

# Use custom network
docker run --network my-network ubuntu
```


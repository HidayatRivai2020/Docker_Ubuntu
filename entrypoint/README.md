# Entrypoint

An entrypoint is a Dockerfile instruction that configures a container to run as an executable. It defines the default command that will be executed when a container starts.

## Overview

- **Purpose**: Specifies the main process that runs when the container starts
- **Flexibility**: Allows containers to accept command-line arguments
- **Use Case**: Ideal for creating container images that function like standalone applications

## Key Characteristics

- **Always Executed**: The entrypoint command always runs when the container starts
- **Not Easily Overridden**: Unlike CMD, requires `--entrypoint` flag to override
- **Accepts Arguments**: Can receive additional arguments from `docker run` command
- **Combines with CMD**: CMD can provide default arguments to ENTRYPOINT

## Basic Syntax

### Dockerfile ENTRYPOINT Instruction

```dockerfile
# Shell form - runs in a shell (/bin/sh -c)
ENTRYPOINT command param1 param2

# Exec form - preferred, runs directly without shell
ENTRYPOINT ["executable", "param1", "param2"]
```

### Docker Run with Entrypoint

```bash
# Use default entrypoint from image
docker run image_name

# Override entrypoint
docker run --entrypoint /bin/bash image_name

# Pass arguments to entrypoint
docker run image_name arg1 arg2
```

## ENTRYPOINT vs CMD

| Feature | ENTRYPOINT | CMD |
|---------|-----------|-----|
| **Purpose** | Main executable | Default arguments or command |
| **Override** | Requires `--entrypoint` flag | Easily overridden by arguments |
| **Use Case** | Application containers | Flexible command execution |
| **Combination** | Uses CMD as default arguments | Can be overridden completely |

## Common Patterns

### Pattern 1: Entrypoint as Main Command

```dockerfile
FROM python:3.9
WORKDIR /app
COPY app.py .
ENTRYPOINT ["python", "app.py"]
```

```bash
# Runs: python app.py
docker run myapp

# Runs: python app.py --debug
docker run myapp --debug
```

### Pattern 2: Entrypoint + CMD for Defaults

```dockerfile
FROM nginx:alpine
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

```bash
# Runs: nginx -g daemon off;
docker run mynginx

# Runs: nginx -v
docker run mynginx -v
```

### Pattern 3: Shell Script as Entrypoint

```dockerfile
FROM ubuntu:22.04
COPY entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["default"]
```

**entrypoint.sh:**
```bash
#!/bin/bash
set -e

# Setup logic
echo "Initializing application..."

# Execute the main command
exec "$@"
```

## Examples

### Simple Python Application

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
ENTRYPOINT ["python", "main.py"]
```

```bash
# Run with default behavior
docker run myapp

# Pass arguments
docker run myapp --port 8080

# Override entrypoint
docker run --entrypoint /bin/bash myapp
```

### Database Initialization

```dockerfile
FROM postgres:15
COPY init-scripts/ /docker-entrypoint-initdb.d/
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["postgres"]
```

### Web Server with Configuration

```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY html/ /usr/share/nginx/html/
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

```bash
# Normal operation
docker run -p 80:80 mynginx

# Test configuration
docker run mynginx -t

# Debug mode
docker run mynginx -g "daemon off;" -e debug
```

### Multi-Stage Build with Entrypoint

```dockerfile
# Build stage
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Runtime stage
FROM alpine:latest
RUN apk add --no-cache ca-certificates
COPY --from=builder /app/myapp /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/myapp"]
CMD ["serve"]
```

## Best Practices

- **Use Exec Form**: Prefer `ENTRYPOINT ["executable", "param"]` over shell form for better signal handling
- **Make Executable**: Ensure scripts used as entrypoints are executable (`chmod +x`)
- **Use `exec`**: In shell scripts, use `exec` to replace the shell process with the main process
- **Combine with CMD**: Use CMD to provide default arguments that users can easily override
- **Handle Signals**: Ensure your entrypoint properly handles SIGTERM for graceful shutdown
- **Document Arguments**: Clearly document what arguments your entrypoint accepts
- **Keep Simple**: Avoid complex logic in entrypoints; use initialization scripts instead
- **Error Handling**: Implement proper error handling with `set -e` in shell scripts

## Advanced Usage

### Dynamic Configuration

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["npm", "start"]
```

**docker-entrypoint.sh:**
```bash
#!/bin/sh
set -e

# Generate configuration from environment variables
cat > config.json <<EOF
{
  "port": ${PORT:-3000},
  "host": "${HOST:-0.0.0.0}",
  "debug": ${DEBUG:-false}
}
EOF

# Execute the main command
exec "$@"
```

### Conditional Execution

```bash
#!/bin/bash
set -e

# Check if running as specific user
if [ "$1" = "app" ]; then
    # Run application
    exec python app.py
elif [ "$1" = "migrate" ]; then
    # Run migrations
    exec python manage.py migrate
else
    # Execute whatever was passed
    exec "$@"
fi
```

## Troubleshooting

### Entrypoint Not Executing

```bash
# Check if file exists and is executable
docker run --entrypoint ls myapp -la /usr/local/bin/

# Verify file permissions
docker run --entrypoint sh myapp -c "ls -l /entrypoint.sh"

# Test script manually
docker run -it --entrypoint /bin/bash myapp
> /entrypoint.sh
```

### Override for Debugging

```bash
# Start interactive shell
docker run -it --entrypoint /bin/bash myapp

# Run specific command
docker run --entrypoint /bin/sh myapp -c "ls -la"

# Bypass entrypoint completely
docker run -it --entrypoint "" myapp /bin/bash
```

### Signal Handling Issues

```dockerfile
# Use exec to ensure proper signal forwarding
ENTRYPOINT ["exec", "python", "app.py"]

# Or in shell script
#!/bin/bash
exec python app.py
```

## See Also

- [CMD vs ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Container Lifecycle](https://docs.docker.com/engine/reference/run/)

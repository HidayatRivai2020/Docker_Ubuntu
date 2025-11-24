# Dockerfile Instructions

## FROM - Base Image
Specifies the base image to build upon. Must be the first instruction (except ARG).

```dockerfile
# Use specific version
FROM ubuntu:22.04

# Use minimal base image
FROM alpine:3.18

# Use official language images
FROM node:18
FROM python:3.11
FROM golang:1.21

# Use scratch for minimal images
FROM scratch
```

## RUN - Execute Commands
Executes commands during image build. Creates a new layer.

```dockerfile
# Single command
RUN apt-get update

# Multiple commands (preferred - fewer layers)
RUN apt-get update && \
    apt-get install -y curl wget && \
    rm -rf /var/lib/apt/lists/*

# Using shell form
RUN echo "Building application"

# Using exec form
RUN ["apt-get", "update"]
```

## COPY - Copy Files
Copies files from build context to image.

```dockerfile
# Copy single file
COPY app.js /app/

# Copy multiple files
COPY package.json package-lock.json /app/

# Copy directory
COPY ./src /app/src

# Copy with wildcard
COPY *.conf /etc/

# Copy and change ownership
COPY --chown=user:group app.js /app/
```

## ADD - Copy and Extract
Similar to COPY but with additional features (URL support, auto-extraction).

```dockerfile
# Copy local file
ADD app.js /app/

# Download from URL
ADD https://example.com/file.tar.gz /tmp/

# Auto-extract tar files
ADD archive.tar.gz /app/

# Note: Prefer COPY over ADD for simple file copying
```

## WORKDIR - Set Working Directory
Sets the working directory for subsequent instructions.

```dockerfile
# Set working directory
WORKDIR /app

# Create nested directories
WORKDIR /app/data/logs

# Use with ENV
ENV APP_HOME=/app
WORKDIR ${APP_HOME}
```

## ENV - Environment Variables
Sets environment variables.

```dockerfile
# Single variable
ENV NODE_ENV=production

# Multiple variables
ENV APP_HOME=/app \
    LOG_LEVEL=info \
    PORT=8080

# Use in other instructions
ENV APP_DIR=/app
WORKDIR ${APP_DIR}
```

## ARG - Build Arguments
Defines build-time variables.

```dockerfile
# Define build argument
ARG VERSION=1.0
ARG BUILD_DATE

# Use in FROM
ARG BASE_IMAGE=ubuntu:22.04
FROM ${BASE_IMAGE}

# Use in RUN
ARG PYTHON_VERSION=3.11
RUN apt-get install -y python${PYTHON_VERSION}
```

## LABEL - Add Metadata
Adds metadata to image as key-value pairs.

```dockerfile
# Single label
LABEL version="1.0"

# Multiple labels
LABEL maintainer="user@example.com" \
      description="My application" \
      version="1.0.0"

# OpenContainer Initiative labels
LABEL org.opencontainers.image.title="My Application"
LABEL org.opencontainers.image.description="Application description"
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.authors="your-email@example.com"
LABEL org.opencontainers.image.source="https://github.com/user/repo"
```

## EXPOSE - Document Ports
Declares which ports the container listens on.

```dockerfile
# Single port
EXPOSE 8080

# Multiple ports
EXPOSE 8080 8443

# With protocol
EXPOSE 8080/tcp
EXPOSE 53/udp
```

## VOLUME - Define Mount Points
Creates mount points for external volumes.

```dockerfile
# Single volume
VOLUME /data

# Multiple volumes
VOLUME ["/var/log", "/var/db"]

# With variable
ENV DATA_DIR=/app/data
VOLUME ${DATA_DIR}
```

## USER - Set User
Sets the user for running subsequent instructions.

```dockerfile
# Switch to user
USER appuser

# Create and switch to user
RUN useradd -m -u 1000 appuser
USER appuser

# Switch back to root
USER root
```

## CMD - Default Command
Specifies the default command to run when container starts.

```dockerfile
# Exec form (preferred)
CMD ["python", "app.py"]

# Shell form
CMD python app.py

# With ENTRYPOINT
CMD ["--port", "8080"]
```

## ENTRYPOINT - Configure Container Executable
Configures container to run as an executable.

```dockerfile
# Exec form (preferred)
ENTRYPOINT ["python", "app.py"]

# Shell form
ENTRYPOINT python app.py

# With CMD for default arguments
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]
```

## HEALTHCHECK - Container Health
Defines how to test if container is healthy.

```dockerfile
# HTTP health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1

# Custom command
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD python health_check.py

# Disable health check
HEALTHCHECK NONE
```
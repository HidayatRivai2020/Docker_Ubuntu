# How to Create a Dockerfile

## Overview
- Text document that contains instructions for building a Docker image. 
- Each instruction creates a layer in the image
- Docker uses these instructions to automate the image building process.

## [Basic Structure](basic_structure.md)
- Base Image (Required)
- Metadata (Optional but recommended)
- Environment Setup
- Working Directory
- Dependencies Installation
- Copy Application Files
- Expose Ports
- Define Startup Command

## [Dockerfile Instructions](instruction.md)
- FROM : Specifies the base image to build upon
- RUN : Executes commands during image build
- COPY : Copies files from build context to image
- ADD : Similar to COPY with additional features (URL support, auto-extraction)
- WORKDIR : Sets the working directory for subsequent instructions
- ENV : Sets environment variables
- ARG : Defines build-time variables
- LABEL : Adds metadata to image
- EXPOSE : Declares which ports the container listens on
- VOLUME : Creates mount points for external volumes
- USER : Sets the user for running subsequent instructions
- CMD : Specifies the default command to run when container starts
- ENTRYPOINT : Configures container to run as an executable
- HEALTHCHECK : Defines how to test if container is healthy

### 5. Database with Initialization
```dockerfile
FROM postgres:15

# Set environment variables
ENV POSTGRES_DB=myapp
ENV POSTGRES_USER=appuser
ENV POSTGRES_PASSWORD=secret

# Copy initialization scripts
COPY ./init-scripts/ /docker-entrypoint-initdb.d/

# Copy custom configuration
COPY postgresql.conf /etc/postgresql/postgresql.conf

EXPOSE 5432

VOLUME /var/lib/postgresql/data
```

## Best Practices

### 1. Optimize Layer Caching
```dockerfile
# ❌ Bad - Copies everything first
COPY . /app
RUN npm install

# ✅ Good - Copies dependencies first
COPY package*.json /app/
RUN npm install
COPY . /app
```

### 2. Minimize Layers
```dockerfile
# ❌ Bad - Multiple RUN commands
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN rm -rf /var/lib/apt/lists/*

# ✅ Good - Single RUN command
RUN apt-get update && \
    apt-get install -y \
      curl \
      wget \
    && rm -rf /var/lib/apt/lists/*
```

### 3. Use Specific Base Image Tags
```dockerfile
# ❌ Bad - Using latest
FROM node:latest

# ✅ Good - Specific version
FROM node:18.17.0-alpine3.18
```

### 4. Use .dockerignore
Create a `.dockerignore` file to exclude unnecessary files:
```
node_modules
.git
.env
*.log
.DS_Store
README.md
.dockerignore
Dockerfile
```

### 5. Security Best Practices
```dockerfile
# Run as non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

# Don't store secrets in image
# Use build secrets instead
RUN --mount=type=secret,id=token \
    TOKEN=$(cat /run/secrets/token) && \
    curl -H "Authorization: Bearer $TOKEN" https://api.example.com

# Scan for vulnerabilities
# Use tools like: docker scan, trivy, snyk
```

### 6. Keep Images Small
```dockerfile
# Use alpine variants
FROM node:18-alpine

# Remove package manager cache
RUN apt-get update && apt-get install -y package \
    && rm -rf /var/lib/apt/lists/*

# Use multi-stage builds
# Copy only necessary files from builder
```

### 7. Add Metadata
```dockerfile
LABEL org.opencontainers.image.title="My Application"
LABEL org.opencontainers.image.description="Application description"
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.authors="your-email@example.com"
LABEL org.opencontainers.image.source="https://github.com/user/repo"
```

## Creating Your First Dockerfile

### Step 1: Create Project Directory
```bash
mkdir my-docker-app
cd my-docker-app
```

### Step 2: Create Application Files
```bash
# Create a simple Node.js application
cat > app.js <<EOF
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from Docker!\\n');
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
EOF

# Create package.json
cat > package.json <<EOF
{
  "name": "my-docker-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  }
}
EOF
```

### Step 3: Create Dockerfile
```bash
cat > Dockerfile <<EOF
FROM node:18-alpine

WORKDIR /app

COPY package.json .
COPY app.js .

EXPOSE 3000

CMD ["node", "app.js"]
EOF
```

### Step 4: Create .dockerignore
```bash
cat > .dockerignore <<EOF
node_modules
npm-debug.log
.git
.env
EOF
```

### Step 5: Build and Run
```bash
# Build the image
docker build -t my-docker-app:1.0 .

# Run the container
docker run -d -p 3000:3000 --name myapp my-docker-app:1.0

# Test the application
curl http://localhost:3000

# View logs
docker logs myapp

# Stop and remove
docker stop myapp
docker rm myapp
```

## Testing Your Dockerfile

### 1. Validate Dockerfile Syntax
```bash
# Using hadolint
docker run --rm -i hadolint/hadolint < Dockerfile

# Or install hadolint locally
hadolint Dockerfile
```

### 2. Build and Test
```bash
# Build with output
docker build --progress=plain -t myapp:test .

# Build without cache
docker build --no-cache -t myapp:test .

# Test the image
docker run --rm myapp:test --version
```

### 3. Inspect the Image
```bash
# View image layers
docker history myapp:test

# Inspect image details
docker inspect myapp:test

# Check image size
docker images myapp:test
```

## Troubleshooting

### Common Issues

**Problem**: Build context is too large
```bash
# Solution: Add .dockerignore file
echo "node_modules" >> .dockerignore
echo ".git" >> .dockerignore
```

**Problem**: Dependencies not found
```dockerfile
# Solution: Ensure correct WORKDIR and COPY order
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
```

**Problem**: Permission denied errors
```dockerfile
# Solution: Set correct permissions
RUN chown -R user:user /app
USER user
```

**Problem**: Large image size
```dockerfile
# Solution: Use multi-stage builds and alpine base images
FROM node:18-alpine AS builder
# ... build steps ...

FROM node:18-alpine
COPY --from=builder /app/dist .
```

## Additional Resources

### Dockerfile Reference
- [Official Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)

### Tools
- **hadolint**: Dockerfile linter
- **dive**: Image layer analyzer
- **docker-slim**: Image optimization tool

### Example Repositories
- Look for `Dockerfile` in open-source projects on GitHub
- Check official Docker images on Docker Hub

## Next Steps

After creating your Dockerfile:

1. **Build the image**: `docker build -t myapp:1.0 .`
2. **Test locally**: `docker run --rm -it myapp:1.0`
3. **Optimize**: Use multi-stage builds, minimize layers
4. **Scan for vulnerabilities**: `docker scan myapp:1.0`
5. **Push to registry**: `docker push myregistry.com/myapp:1.0`
6. **Document**: Add README with build and run instructions

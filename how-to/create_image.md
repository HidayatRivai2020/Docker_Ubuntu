# How to Create a New Image

## Using Dockerfile (Recommended)
- Create Project Structure
- Create Application
- Create Dockerfile
- Build the Image
- Test the Image

## From a Running Container
- Start a Base Container
- Make Changes in Container
- Commit Container to Image
- Test New Image
- Important Notes
    - `docker commit` is not recommended for production
    - Changes are not reproducible
    - Dockerfile method is preferred for version control


## Import from Tarball
- Export Container Filesystem
- Save and Load Complete Images

## Advanced Image Creation
- Multi-Stage Builds : Create optimized images with multiple build stages.
- Build with BuildKit : Enable advanced features and better performance.
- Build for Multiple Platforms

## Optimizing Image Size

### 1. Use Smaller Base Images
```dockerfile
# Instead of full Ubuntu (77MB)
FROM ubuntu:22.04

# Use Alpine (5MB)
FROM alpine:3.18

# Use slim variants (50MB)
FROM python:3.11-slim

# Use distroless (2-20MB)
FROM gcr.io/distroless/python3-debian11
```

### 2. Minimize Layers
```dockerfile
# ❌ Bad - Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget

# ✅ Good - Single layer
RUN apt-get update && \
    apt-get install -y curl wget && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Use .dockerignore
```
# .dockerignore
node_modules
.git
.env
*.log
.DS_Store
*.md
.gitignore
Dockerfile
docker-compose.yml
```

### 4. Multi-Stage Builds
```dockerfile
# Build stage (large)
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o main .

# Production stage (minimal)
FROM alpine:3.18
COPY --from=builder /app/main /app/main
CMD ["/app/main"]
```

### 5. Clean Up in Same Layer
```dockerfile
# ❌ Bad - Cache remains
RUN apt-get update
RUN apt-get install -y build-essential
RUN apt-get clean

# ✅ Good - Clean in same layer
RUN apt-get update && \
    apt-get install -y build-essential && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

## Image Versioning Best Practices

### Semantic Versioning
```bash
# Major.Minor.Patch
docker build -t myapp:1.0.0 .
docker build -t myapp:1.0 .
docker build -t myapp:1 .
docker build -t myapp:latest .
```

### Git-Based Versioning
```bash
# Use git tags
VERSION=$(git describe --tags --always)
docker build -t myapp:$VERSION .

# Use git commit hash
COMMIT=$(git rev-parse --short HEAD)
docker build -t myapp:$COMMIT .

# Use branch name
BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker build -t myapp:$BRANCH .
```

### Date-Based Versioning
```bash
# Use ISO date
DATE=$(date +%Y%m%d)
docker build -t myapp:$DATE .

# With time
DATETIME=$(date +%Y%m%d-%H%M%S)
docker build -t myapp:$DATETIME .
```

## Automated Image Builds

### Using GitHub Actions
```yaml
name: Build Docker Image

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: username/myapp
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Using Makefile
```makefile
IMAGE_NAME = myapp
VERSION = $(shell git describe --tags --always)
REGISTRY = username

.PHONY: build
build:
	docker build -t $(IMAGE_NAME):$(VERSION) .
	docker tag $(IMAGE_NAME):$(VERSION) $(IMAGE_NAME):latest

.PHONY: build-prod
build-prod:
	docker build -f Dockerfile.prod -t $(IMAGE_NAME):$(VERSION)-prod .

.PHONY: push
push:
	docker tag $(IMAGE_NAME):$(VERSION) $(REGISTRY)/$(IMAGE_NAME):$(VERSION)
	docker push $(REGISTRY)/$(IMAGE_NAME):$(VERSION)

.PHONY: test
test:
	docker run --rm $(IMAGE_NAME):$(VERSION) pytest

.PHONY: clean
clean:
	docker rmi $(IMAGE_NAME):$(VERSION) || true
```

Usage:
```bash
make build
make build-prod
make push
make test
make clean
```

## Validating Images

### 1. Lint Dockerfile
```bash
# Using hadolint
docker run --rm -i hadolint/hadolint < Dockerfile

# Or install locally
brew install hadolint
hadolint Dockerfile
```

### 2. Scan for Vulnerabilities
```bash
# Using Docker scan
docker scan myapp:latest

# Using Trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image myapp:latest

# Using Snyk
snyk container test myapp:latest
```

### 3. Inspect Image Layers
```bash
# View layer history
docker history myapp:latest

# View detailed information
docker inspect myapp:latest

# Using dive for analysis
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest myapp:latest
```

### 4. Test Image
```bash
# Run basic test
docker run --rm myapp:latest --version

# Run with shell
docker run --rm -it myapp:latest /bin/sh

# Check exposed ports
docker inspect --format='{{.Config.ExposedPorts}}' myapp:latest

# Check environment variables
docker inspect --format='{{.Config.Env}}' myapp:latest
```

## Troubleshooting Image Creation

### Build Fails

**Problem**: Dockerfile syntax error
```bash
# Validate syntax
docker build --check .

# Use linter
hadolint Dockerfile
```

**Problem**: Context too large
```bash
# Check context size
docker build --progress=plain -t myapp . 2>&1 | grep "sending build context"

# Add .dockerignore
echo "node_modules" >> .dockerignore
echo ".git" >> .dockerignore
```

**Problem**: Network issues during build
```bash
# Use host network
docker build --network=host -t myapp .

# Use specific DNS
docker build --dns 8.8.8.8 -t myapp .
```

### Image Too Large

```bash
# Check image size
docker images myapp:latest

# Check layer sizes
docker history myapp:latest --human --no-trunc

# Analyze with dive
dive myapp:latest
```

**Solutions**:
- Use smaller base images
- Use multi-stage builds
- Clean up in same RUN command
- Add .dockerignore file

### Build Cache Issues

```bash
# Build without cache
docker build --no-cache -t myapp .

# Pull latest base image
docker pull python:3.11-slim
docker build -t myapp .

# Clear build cache
docker builder prune
```

## Best Practices

1. **Use official base images**: Start with trusted images
2. **Pin versions**: Use specific tags, not `latest`
3. **Minimize layers**: Combine RUN commands
4. **Order instructions**: Put frequently changing instructions last
5. **Use .dockerignore**: Exclude unnecessary files
6. **Add metadata**: Use LABEL for documentation
7. **Run as non-root**: Create and use non-privileged user
8. **Scan for vulnerabilities**: Regularly scan images
9. **Use multi-stage builds**: Reduce final image size
10. **Document**: Add comments in Dockerfile

## Example: Complete Image Creation Workflow

### 1. Develop Application
```bash
# Create application
mkdir my-nodejs-app
cd my-nodejs-app
npm init -y
npm install express

# Create app
cat > index.js <<EOF
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Docker!' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
EOF
```

### 2. Create Dockerfile
```dockerfile
FROM node:18-alpine

LABEL maintainer="you@example.com"
LABEL version="1.0.0"
LABEL description="My Node.js application"

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s \
  CMD node -e "require('http').get('http://localhost:3000/health')" || exit 1

CMD ["node", "index.js"]
```

### 3. Create .dockerignore
```
node_modules
npm-debug.log
.git
.env
.DS_Store
*.md
Dockerfile
.dockerignore
```

### 4. Build and Tag
```bash
# Build image
docker build -t my-nodejs-app:1.0.0 .

# Tag versions
docker tag my-nodejs-app:1.0.0 my-nodejs-app:1.0
docker tag my-nodejs-app:1.0.0 my-nodejs-app:1
docker tag my-nodejs-app:1.0.0 my-nodejs-app:latest
```

### 5. Test Image
```bash
# Run container
docker run -d -p 3000:3000 --name test my-nodejs-app:1.0.0

# Test application
curl http://localhost:3000

# Check logs
docker logs test

# Check health
docker inspect --format='{{.State.Health.Status}}' test

# Clean up
docker stop test
docker rm test
```

### 6. Scan and Validate
```bash
# Scan for vulnerabilities
docker scan my-nodejs-app:1.0.0

# Lint Dockerfile
hadolint Dockerfile

# Analyze layers
docker history my-nodejs-app:1.0.0
```

### 7. Push to Registry
```bash
# Tag for registry
docker tag my-nodejs-app:1.0.0 username/my-nodejs-app:1.0.0

# Login and push
docker login
docker push username/my-nodejs-app:1.0.0
docker push username/my-nodejs-app:latest
```


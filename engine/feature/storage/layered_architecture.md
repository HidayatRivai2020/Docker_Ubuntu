# Layered Architecture

Docker uses a layered filesystem architecture where images are built from multiple read-only layers stacked on top of each other.

## Layer Structure

```
┌─────────────────────────────────────┐
│  Container Layer (writable)         │ ← Runtime changes
├─────────────────────────────────────┤
│  Layer 5: ENTRYPOINT ["app"]        │
├─────────────────────────────────────┤
│  Layer 4: COPY app.js /app/         │
├─────────────────────────────────────┤
│  Layer 3: RUN npm install           │
├─────────────────────────────────────┤
│  Layer 2: WORKDIR /app              │
├─────────────────────────────────────┤
│  Layer 1: FROM node:18              │ ← Base layer
└─────────────────────────────────────┘
```

## How Layers Work

### Layer Creation Process

1. **Base Image Layer**: Starting point from base image (e.g., `FROM ubuntu:22.04`)
2. **Instruction Layers**: Each Dockerfile instruction that modifies filesystem creates a new layer
3. **Read-Only Nature**: All image layers are immutable and never change
4. **Layer Stacking**: Layers stacked from bottom (base) to top (latest)
5. **Container Layer**: Thin writable layer added when container starts
6. **Unified View**: Union filesystem presents all layers as single filesystem

### Dockerfile Instructions and Layers

```dockerfile
FROM ubuntu:22.04          # Layer 1: Base image
RUN apt-get update         # Layer 2: Package index update
RUN apt-get install -y nginx  # Layer 3: Nginx installation
COPY index.html /var/www/html/  # Layer 4: Copy files
EXPOSE 80                  # Layer 5: Metadata (no filesystem change)
CMD ["nginx", "-g", "daemon off;"]  # Layer 6: Metadata
```

**Layer Types:**

- **Create Data Layers**: `RUN`, `COPY`, `ADD` (modify filesystem)
- **Metadata Only**: `ENV`, `EXPOSE`, `CMD`, `ENTRYPOINT`, `LABEL`, `WORKDIR` (no layer data)

**Note**: Metadata instructions still create layers but don't add filesystem data, resulting in 0B size.

## Layer Identification

Each layer has a unique SHA256 hash calculated from its content.

### Viewing Layer Information

```bash
# View layer history
docker history nginx:latest
```

**Example Output:**
```
IMAGE          CREATED        CREATED BY                              SIZE
ab56bba91343   2 weeks ago    CMD ["nginx" "-g" "daemon off;"]        0B
<missing>      2 weeks ago    EXPOSE 80                               0B
<missing>      2 weeks ago    COPY index.html /var/www/html/          2.1kB
<missing>      2 weeks ago    RUN apt-get install -y nginx            85MB
<missing>      2 weeks ago    RUN apt-get update                      23MB
<missing>      3 weeks ago    FROM ubuntu:22.04                       77MB
```

### Layer Commands

```bash
# View detailed layer information
docker image inspect <image>

# View layer IDs (SHA256 hashes)
docker image inspect <image> --format='{{.RootFS.Layers}}'

# Analyze layer sizes
docker history <image> --format "table {{.Size}}\t{{.CreatedBy}}" --no-trunc

# Check specific layer
docker image inspect <image> --format='{{json .RootFS}}' | jq
```

## Layer Sharing

Multiple images and containers share common base layers, providing significant efficiency benefits.

### Sharing Between Images

```
Image 1: node:18-alpine        Image 2: my-app:v1
┌────────────────────┐         ┌────────────────────┐
│ COPY app.js        │         │ ENTRYPOINT app     │
├────────────────────┤         ├────────────────────┤
│ RUN npm install    │         │ COPY app.js        │
├────────────────────┤         ├────────────────────┤
│ COPY package.json  │         │ RUN npm install    │
└────────────────────┘         ├────────────────────┤
         │                     │ COPY package.json  │
         │                     ├────────────────────┤
         └─────────────────────►│ FROM node:18       │ ← Shared base
                               └────────────────────┘
```

### Sharing Between Containers

```
┌────────────────────────────────┐
│  Container 1 (writable)        │ ← Independent changes
├────────────────────────────────┤
│  Container 2 (writable)        │ ← Independent changes
├────────────────────────────────┤
│  Container 3 (writable)        │ ← Independent changes
├────────────────────────────────┤
│  Image Layers (read-only)      │ ← Shared, never modified
└────────────────────────────────┘
```

### Benefits of Layer Sharing

- **Storage Efficiency**: Base layers stored once on disk, shared by all images using them
- **Faster Image Pulls**: Only new/missing layers downloaded from registry
- **Faster Builds**: Cached layers reused instead of rebuilding
- **Memory Efficiency**: Shared pages in RAM across containers
- **Reduced Bandwidth**: Less data transferred during push/pull operations

### Example: Layer Sharing in Practice

If you have 10 containers running from the same image:
- **Without layer sharing**: 10 × 500MB = 5GB disk usage
- **With layer sharing**: 500MB (image) + 10 × 10MB (container layers) = 600MB disk usage

## Layer Caching

Docker caches layers during image builds to optimize build time and efficiency.

### Cache Behavior

- **Cache Hit**: If instruction and context unchanged, Docker reuses existing layer
- **Cache Miss**: If instruction or context changes, Docker rebuilds layer and all subsequent layers
- **Cache Invalidation**: Changing one layer invalidates cache for all layers after it

### Efficient Layer Ordering

```dockerfile
# ✓ EFFICIENT: Dependencies cached separately
FROM node:18
WORKDIR /app

# These layers rarely change
COPY package*.json ./        # Layer cached until package.json changes
RUN npm install              # Layer cached if package.json unchanged

# This layer changes frequently
COPY . .                     # Only this rebuilds on source changes

CMD ["node", "app.js"]
```

```dockerfile
# ✗ INEFFICIENT: Cache invalidated on any source change
FROM node:18
WORKDIR /app

# This invalidates cache on ANY file change
COPY . .                     # Layer invalidated on any file change
RUN npm install              # Reinstalls dependencies every time

CMD ["node", "app.js"]
```

### Layer Ordering Best Practices

1. **Base Image First**: Start with `FROM` instruction
2. **System Dependencies**: Install system packages early
3. **Application Dependencies**: Copy dependency files (package.json, requirements.txt) before source code
4. **Install Dependencies**: Run package manager installs
5. **Application Code**: Copy application source code last
6. **Configuration**: Add configuration and metadata
7. **Entry Point**: Set CMD/ENTRYPOINT last

**Rule**: Order instructions from least frequently changed to most frequently changed.

### Build Cache Management

```bash
# Build without using cache
docker build --no-cache -t myapp .

# Build from specific layer (cache up to that point)
docker build --cache-from myapp:latest -t myapp:v2 .

# Pull cache from registry
docker pull myapp:latest
docker build --cache-from myapp:latest -t myapp:v2 .
```

## Layer Size Optimization

### Problem: Multiple RUN Commands

Each `RUN` command creates a separate layer, and deleted files in later layers don't reduce earlier layer sizes.

```dockerfile
# ✗ CREATES 3 SEPARATE LAYERS
RUN apt-get update              # Layer 1: 23MB
RUN apt-get install -y nginx    # Layer 2: 85MB  
RUN apt-get clean               # Layer 3: 0MB (but Layer 1 and 2 remain!)
# Total image size: 108MB
```

**Why this is inefficient:**
- Layer 1 contains package lists (23MB)
- Layer 2 contains installed packages (85MB)
- Layer 3 tries to clean up, but layers 1 and 2 are immutable
- Final image still contains all 108MB

### Solution: Chain Commands

Combine related commands in a single `RUN` instruction using `&&`:

```dockerfile
# ✓ CREATES 1 OPTIMIZED LAYER
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
# Total image size: 62MB
```

**Why this works:**
- All commands execute in same layer
- Cleanup happens before layer commit
- Temporary files never stored in layer
- Much smaller final image

### Additional Optimization Techniques

#### 1. Multi-line RUN with Cleanup

```dockerfile
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    package3 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

#### 2. Download and Remove in Same Layer

```dockerfile
RUN wget https://example.com/file.tar.gz && \
    tar -xzf file.tar.gz && \
    mv extracted /opt/app && \
    rm file.tar.gz
```

#### 3. Build Dependencies Cleanup

```dockerfile
RUN apk add --no-cache --virtual .build-deps \
    gcc \
    musl-dev \
    && pip install some-package \
    && apk del .build-deps
```

#### 4. Use Smaller Base Images

```dockerfile
# Instead of full image (200MB+)
FROM ubuntu:22.04

# Use slim variant (50-100MB)
FROM ubuntu:22.04-slim

# Or Alpine for smallest size (~5MB)
FROM alpine:3.18
```

## Viewing and Analyzing Layers

### Layer History

```bash
# Basic history
docker history <image>

# With full command details
docker history --no-trunc <image>

# Show only sizes
docker history --format "table {{.Size}}\t{{.CreatedBy}}" <image>

# Human-readable with dates
docker history --human=true <image>
```

### Layer Inspection

```bash
# Full image details
docker image inspect <image>

# Extract layer information
docker image inspect <image> --format='{{json .RootFS}}' | jq

# List all layer IDs
docker image inspect <image> --format='{{range .RootFS.Layers}}{{println .}}{{end}}'

# Get layer count
docker image inspect <image> --format='{{len .RootFS.Layers}}'
```

### Layer Storage Location

```bash
# Find storage driver
docker info | grep "Storage Driver"

# Layer storage location (overlay2)
ls -la /var/lib/docker/overlay2/

# View specific layer content
sudo ls -la /var/lib/docker/overlay2/<layer-id>/diff/
```

## Multi-Stage Builds

Multi-stage builds create multiple intermediate images but only keep the final stage, dramatically reducing image size.

### Basic Multi-Stage Build

```dockerfile
# Stage 1: Build environment
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production environment (only this creates final image layers)
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

**Result**: 
- Build stage creates large image with dev dependencies (500MB)
- Production stage only includes runtime files (100MB)
- Final image is 100MB (build stage discarded)

### Copying from Previous Stages

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Test stage
FROM builder AS tester
RUN go test ./...

# Production stage
FROM alpine:3.18
COPY --from=builder /app/myapp /usr/local/bin/
CMD ["myapp"]
```

### Copying from External Images

```dockerfile
FROM alpine:3.18
# Copy from specific image (not a build stage)
COPY --from=nginx:latest /etc/nginx/nginx.conf /etc/nginx/
COPY --from=busybox:latest /bin/busybox /bin/
```

## Layer Immutability

Once created, image layers are immutable and never change.

### Immutability Benefits

- **Consistency**: Same image always has same layers
- **Security**: Layers can't be tampered with after creation
- **Caching**: Safe to cache and reuse layers
- **Verification**: SHA256 hash verifies layer integrity
- **Sharing**: Safe to share layers between images and containers

### Container Writable Layer

```
┌─────────────────────────────────┐
│  Container Writable Layer       │ ← Changes stored here
│  - New files created            │
│  - Modified files (CoW)         │
│  - Deleted files (whiteout)     │
├─────────────────────────────────┤
│  Image Layer 3 (read-only)      │ ← Immutable
├─────────────────────────────────┤
│  Image Layer 2 (read-only)      │ ← Immutable
├─────────────────────────────────┤
│  Image Layer 1 (read-only)      │ ← Immutable
└─────────────────────────────────┘
```

**Container Layer Characteristics:**
- Thin writable layer on top of image layers
- All container changes stored here
- Deleted when container removed
- Not shared between containers
- Slower than volumes for heavy I/O

## Best Practices

### 1. Minimize Layer Count

```dockerfile
# ✗ BAD: 4 layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim
RUN apt-get clean

# ✓ GOOD: 1 layer
RUN apt-get update && \
    apt-get install -y curl vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 2. Order Layers by Change Frequency

```dockerfile
# Least frequently changed (cached longer)
FROM node:18
RUN apt-get update && apt-get install -y git

# Moderately changed
COPY package*.json ./
RUN npm install

# Most frequently changed
COPY . .
```

### 3. Use .dockerignore

```
# .dockerignore
node_modules
.git
.env
*.log
README.md
```

Prevents unnecessary files from invalidating cache and increasing layer size.

### 4. Leverage Build Cache

```bash
# Build with cache
docker build -t myapp:v1 .

# Rebuild quickly using cache
docker build -t myapp:v2 .

# Share cache via registry
docker push myapp:v1
docker pull myapp:v1
docker build --cache-from myapp:v1 -t myapp:v2 .
```

### 5. Use Multi-Stage Builds

- Separate build and runtime dependencies
- Keep final image small
- Improve security by excluding build tools

### 6. Clean Up in Same Layer

```dockerfile
# ✓ GOOD: Cleanup in same RUN
RUN wget file.tar.gz && \
    tar -xzf file.tar.gz && \
    rm file.tar.gz

# ✗ BAD: Cleanup in separate RUN
RUN wget file.tar.gz
RUN tar -xzf file.tar.gz  
RUN rm file.tar.gz  # file.tar.gz still in previous layer!
```

### 7. Use Specific Base Image Tags

```dockerfile
# ✓ GOOD: Specific version
FROM node:18.17-alpine3.18

# ✗ BAD: Floating tag
FROM node:latest
```

Ensures consistent base layers across builds.

## Troubleshooting Layers

### Large Image Size

```bash
# Identify large layers
docker history <image> --no-trunc

# Analyze each layer size
docker history <image> --format "table {{.Size}}\t{{.CreatedBy}}"

# Find what's in large layers
docker run --rm <image> du -sh /*
```

### Cache Not Working

```bash
# Rebuild without cache
docker build --no-cache -t myapp .

# Check if .dockerignore is excluding wrong files
cat .dockerignore

# Verify build context size
docker build --no-cache -t myapp . 2>&1 | grep "Sending build context"
```

### Layer Corruption

```bash
# Remove corrupted images
docker image prune -a

# Pull fresh image
docker pull <image>

# Rebuild from scratch
docker build --pull --no-cache -t myapp .
```

### Inspect Layer Content

```bash
# Export image
docker save <image> -o image.tar

# Extract and examine
tar -xf image.tar

# View layer contents
tar -tzf <layer-id>/layer.tar
```
# build

## Overview
- Creates Docker images from a Dockerfile and build context
- Automates the process of building custom container images
- Supports multi-stage builds, build arguments, and caching
- Essential for creating custom applications and deployments
- Integrates with Docker registries for image distribution

## Basic Syntax
```bash
docker build [OPTIONS] PATH | URL | -
```

## Common Options

### Image Naming and Tagging
- `-t, --tag` - Name and optionally tag the image (name:tag format)
- `--label` - Set metadata labels on the image
- `--iidfile` - Write image ID to a file

### Build Context
- `-f, --file` - Name of the Dockerfile (default is PATH/Dockerfile)
- `--build-arg` - Set build-time variables
- `--target` - Set target build stage for multi-stage builds

### Build Optimization
- `--no-cache` - Do not use cache when building
- `--pull` - Always attempt to pull newer version of base image
- `--cache-from` - Images to consider as cache sources
- `--rm` - Remove intermediate containers after successful build (default true)
- `--force-rm` - Always remove intermediate containers

### Network and Security
- `--network` - Set networking mode for RUN instructions
- `--add-host` - Add custom host-to-IP mapping
- `--ssh` - SSH agent socket or keys to expose
- `--secret` - Secret file to expose to build

### Platform and Architecture
- `--platform` - Set platform if server is multi-platform capable
- `--build-context` - Additional build contexts

### Output Control
- `-q, --quiet` - Suppress build output and print image ID on success
- `--progress` - Set type of progress output (auto, plain, tty)
- `--output` - Output destination (local directory, tar archive, registry)

### Resource Limits
- `--memory` - Memory limit for build
- `--memory-swap` - Swap limit equal to memory plus swap
- `--cpu-shares` - CPU shares (relative weight)
- `--cpuset-cpus` - CPUs in which to allow execution

## How It Works

### Build Process
1. **Context Preparation** - Sends build context (files and directories) to Docker daemon
2. **Dockerfile Parsing** - Reads and validates Dockerfile instructions
3. **Layer Creation** - Executes each instruction, creating intermediate layers
4. **Caching** - Reuses cached layers when possible for faster builds
5. **Image Creation** - Combines all layers into final image
6. **Tagging** - Applies specified tags to the built image

### Build Context
- Directory or URL containing files needed for the build
- Sent to Docker daemon before build starts
- Can be local path, Git repository, or tarball
- Files not needed should be excluded via `.dockerignore`

### Layer Caching
- Each Dockerfile instruction creates a layer
- Layers are cached and reused if unchanged
- Cache is invalidated if instruction or context changes
- Speeds up subsequent builds significantly

### Multi-Stage Builds
- Use multiple FROM statements in single Dockerfile
- Copy artifacts between stages
- Reduces final image size
- Separates build and runtime dependencies

## Best Practices

- **Use .dockerignore** - Exclude unnecessary files from build context
- **Order instructions wisely** - Place frequently changing instructions last
- **Minimize layers** - Combine related RUN commands with &&
- **Use multi-stage builds** - Reduce final image size
- **Leverage build cache** - Structure Dockerfile for optimal caching
- **Pin base image versions** - Use specific tags instead of `latest`
- **Use build arguments** - Make builds more flexible and reusable
- **Keep images small** - Remove unnecessary files and dependencies
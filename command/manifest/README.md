# manifest

## Overview
- Creates and manages Docker image manifest lists for multi-platform images
- Enables building and distributing images that support multiple architectures (amd64, arm64, armv7, etc.)
- Essential for supporting diverse hardware platforms with a single image tag
- Requires experimental features to be enabled in Docker CLI
- Modern alternative: Docker Buildx (recommended for new projects)

## Basic Syntax
```bash
docker manifest COMMAND [OPTIONS]
```

## Subcommands

### docker manifest create
- Create a local manifest list for multi-platform images

#### Basic Syntax
```bash
docker manifest create MANIFEST_LIST MANIFEST [MANIFEST...]
```

#### Common Options
- `-a, --amend` - Amend an existing manifest list
- `--insecure` - Allow communication with insecure registries
- `--help` - Display help information

### docker manifest annotate
- Add additional information to a local image manifest

#### Basic Syntax
```bash
docker manifest annotate MANIFEST_LIST MANIFEST [OPTIONS]
```

#### Common Options
- `--arch` - Set architecture (amd64, arm64, arm, etc.)
- `--os` - Set operating system (linux, windows, etc.)
- `--variant` - Set architecture variant (v6, v7, v8)
- `--os-version` - Set operating system version

### docker manifest inspect
- Display manifest details for an image

#### Basic Syntax
```bash
docker manifest inspect IMAGE_NAME[:TAG]
```

#### Common Options
- `--insecure` - Allow communication with insecure registries
- `--verbose` - Show additional information

### docker manifest push
- Push a manifest list to a registry

#### Basic Syntax
```bash
docker manifest push MANIFEST_LIST [OPTIONS]
```

#### Common Options
- `-p, --purge` - Remove local manifest list after push
- `--insecure` - Allow push to insecure registries

### docker manifest rm
- Remove one or more manifest lists from local storage

#### Basic Syntax
```bash
docker manifest rm MANIFEST_LIST [MANIFEST_LIST...]
```

## How It Works

### Multi-Platform Image Flow
```
Build Platform Images → Create Manifest List → Annotate Platforms → Push to Registry
        ↓                       ↓                      ↓                    ↓
   (amd64, arm64)      Combine References      Add Metadata         Single Tag
                                                                           ↓
                                          Pull → Auto-Select Platform Image
```

### Manifest List Structure
1. **Build** - Create platform-specific images (myapp:amd64, myapp:arm64)
2. **Create** - Combine them into manifest list (myapp:latest)
3. **Annotate** - Add platform metadata (architecture, OS, variant)
4. **Push** - Upload manifest list to registry
5. **Pull** - Docker automatically selects correct platform image

## Enable Experimental Features

### Temporary (Current Session)
```bash
export DOCKER_CLI_EXPERIMENTAL=enabled
```

### Permanent (Configuration File)
```bash
# Create or edit ~/.docker/config.json
{
  "experimental": "enabled"
}
```

### Verify
```bash
docker version --format '{{.Client.Experimental}}'
# Should output: true
```

## Platform Identifiers
- **OS**: Operating system (linux, windows, darwin)
- **Architecture**: CPU architecture (amd64, arm64, arm, 386)
- **Variant**: Architecture variant (v6, v7, v8)

## Best Practices

- **Use Buildx for new projects** - Simpler and more powerful than manifest commands
- **Test on target platforms** - Always verify images work on actual hardware
- **Label platform images** - Use clear tags (myapp:1.0-amd64, myapp:1.0-arm64)
- **Document supported platforms** - List architectures in README
- **Use CI/CD for builds** - Automate multi-platform builds in pipelines
- **Verify manifest lists** - Always inspect after creation
- **Clean up local manifests** - Use `--purge` flag when pushing
- **Enable experimental features** - Configure Docker CLI properly

## Understanding Manifest Lists

### Manifest vs Manifest List
- **Manifest**: Single image metadata (one platform)
- **Manifest List**: Collection of manifests (multiple platforms)

### Image Selection
When you pull `myapp:latest`:
1. Docker checks manifest list
2. Identifies current platform (OS/Architecture)
3. Selects matching manifest from list
4. Pulls corresponding image layers

### Manifest List Contents
```json
{
  "manifests": [
    {
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      },
      "digest": "sha256:abc123..."
    },
    {
      "platform": {
        "architecture": "arm64",
        "os": "linux"
      },
      "digest": "sha256:def456..."
    }
  ]
}
```

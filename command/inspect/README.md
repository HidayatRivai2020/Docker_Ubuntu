# inspect

## Overview
- Returns detailed information about Docker objects (containers, images, volumes, networks)
- Displays comprehensive metadata and configuration details in JSON format
- Essential tool for debugging, troubleshooting, and system administration
- Provides low-level information not available through other commands
- Supports multiple object types and batch inspection

## Basic Syntax
```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

## Common Options

### Output Formatting
- `-f, --format` - Format output using Go template syntax
- `--type` - Return JSON for specified type (container, image, volume, network)
- `-s, --size` - Display total file sizes (for containers)

### Object Selection
- Can inspect multiple objects simultaneously
- Supports partial IDs and names
- Automatically detects object type (container, image, etc.)

## How It Works

### Inspection Process
1. **Object Resolution** - Resolves names/IDs to specific Docker objects
2. **Type Detection** - Automatically determines object type if not specified
3. **Metadata Retrieval** - Gathers comprehensive configuration and state data
4. **Format Processing** - Applies specified output formatting or returns raw JSON
5. **Batch Processing** - Handles multiple objects in single command execution

### Information Categories
- **Configuration** - Runtime settings, environment variables, command arguments
- **State Information** - Current status, exit codes, timestamps
- **Network Details** - IP addresses, port mappings, network attachments
- **Storage Information** - Volume mounts, filesystem layers, disk usage
- **Resource Limits** - Memory, CPU, and other resource constraints

## Best Practices

- **Use format templates** - Extract specific information with `--format` option
- **Inspect before debugging** - Gather complete information before troubleshooting
- **Check state information** - Review status and timestamps for issue diagnosis
- **Verify configurations** - Confirm settings match expected values
- **Monitor resource usage** - Use size information for capacity planning
- **Script with jq** - Process JSON output with jq for automation
- **Document configurations** - Use inspect output for environment documentation
- **Compare configurations** - Use inspect to diff container/image settings
# logs

## Overview
- Displays logs from Docker containers in real-time or historical format
- Essential tool for debugging, monitoring, and troubleshooting containerized applications
- Supports multiple output formats and filtering options
- Can follow logs in real-time or retrieve historical log entries
- Works with both running and stopped containers

## Basic Syntax
```bash
docker logs [OPTIONS] CONTAINER
```

## Common Options

### Output Control
- `-f, --follow` - Follow log output in real-time (like `tail -f`)
- `-t, --timestamps` - Show timestamps for each log entry
- `--tail N` - Show only the last N lines of logs
- `--since TIMESTAMP` - Show logs since timestamp (e.g., 2023-01-01T12:00:00)
- `--until TIMESTAMP` - Show logs before timestamp
- `-n, --lines N` - Number of lines to show from the end of logs

### Format Options
- `--details` - Show extra details provided to logs
- `--no-color` - Disable color output
- `--no-stream` - Disable streaming, return logs and exit

### Time Filters
- `--since "1h"` - Logs from 1 hour ago
- `--since "2023-01-01"` - Logs since specific date
- `--since "10m"` - Logs from 10 minutes ago
- `--until "1h"` - Logs until 1 hour ago

## How It Works

### Log Collection Process
1. **Log Retrieval** - Docker daemon retrieves logs from container's log driver
2. **Filter Application** - Applies time-based filters and line limits
3. **Format Processing** - Adds timestamps and formatting as requested
4. **Output Streaming** - Displays logs with optional real-time following
5. **Buffer Management** - Handles log rotation and memory management

### Log Sources
- **STDOUT** - Standard output from container processes
- **STDERR** - Standard error output from container processes
- **Application Logs** - Direct application logging output
- **System Messages** - Container lifecycle and system events

### Log Drivers
- **json-file** - Default driver, stores logs as JSON files
- **syslog** - Sends logs to syslog daemon
- **journald** - Sends logs to systemd journal
- **fluentd** - Sends logs to Fluentd collector
- **awslogs** - Sends logs to Amazon CloudWatch
- **gcplogs** - Sends logs to Google Cloud Logging

## Best Practices

- **Use timestamps** - Always include `-t` for debugging and monitoring
- **Limit output** - Use `--tail` to avoid overwhelming output
- **Follow selectively** - Use `-f` only when actively monitoring
- **Filter by time** - Use `--since` and `--until` for specific time ranges
- **Monitor resource usage** - Be aware that log following can consume resources
- **Combine with grep** - Filter logs with grep for specific patterns
- **Use log rotation** - Configure proper log rotation to prevent disk space issues
- **Structure application logs** - Use structured logging (JSON) for better parsing
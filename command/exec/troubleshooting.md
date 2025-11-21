# Docker exec Troubleshooting

## Common Issues

### Container Not Running
**Problem**: Cannot exec into container that's not running
```
Error response from daemon: Container <container_id> is not running
```

**Solutions**:
```bash
# Check container status
docker ps -a --filter name=<container_name>

# Start the container if stopped
docker start <container_name>

# Check why container stopped
docker logs <container_name>

# Restart container if needed
docker restart <container_name>

# Then try exec again
docker exec -it <container_name> bash
```

### Command Not Found
**Problem**: Specified command doesn't exist in container
```
OCI runtime exec failed: exec: "bash": executable file not found
```

**Solutions**:
```bash
# Try alternative shells
docker exec -it <container_name> sh
docker exec -it <container_name> /bin/sh
docker exec -it <container_name> ash  # Alpine Linux

# Check available shells
docker exec <container_name> cat /etc/shells

# Find available executables
docker exec <container_name> ls /bin
docker exec <container_name> which bash

# Use full path
docker exec -it <container_name> /bin/bash
```

### Permission Denied
**Problem**: Insufficient permissions to execute command
```
Permission denied
```

**Solutions**:
```bash
# Check current user in container
docker exec <container_name> whoami

# Run as root
docker exec --user root <container_name> <command>

# Run as specific user
docker exec --user 0:0 <container_name> <command>

# Check file permissions
docker exec <container_name> ls -la /path/to/file

# Change permissions (as root)
docker exec --user root <container_name> chmod +x /path/to/file
```

### TTY/Interactive Issues
**Problem**: Issues with interactive sessions or terminal allocation
```
the input device is not a TTY
```

**Solutions**:
```bash
# Use proper TTY allocation
docker exec -it <container_name> bash

# For non-interactive environments (CI/CD)
docker exec -i <container_name> bash

# Force TTY allocation
docker exec -t <container_name> bash

# Check if TTY is available
test -t 0 && echo "TTY available" || echo "No TTY"
```

### Working Directory Issues
**Problem**: Command fails due to wrong working directory
```
No such file or directory
```

**Solutions**:
```bash
# Specify working directory
docker exec -w /app <container_name> <command>

# Check current working directory
docker exec <container_name> pwd

# Change to correct directory
docker exec <container_name> sh -c "cd /app && <command>"

# Use absolute paths
docker exec <container_name> /full/path/to/command
```

### Environment Variable Issues
**Problem**: Missing or incorrect environment variables
```
command: not found
```

**Solutions**:
```bash
# Check environment variables
docker exec <container_name> env

# Set required environment variables
docker exec -e PATH="/usr/local/bin:$PATH" <container_name> <command>

# Use environment file
docker exec --env-file .env <container_name> <command>

# Source profile files
docker exec <container_name> sh -c "source ~/.profile && <command>"
```

### Process Exit Issues
**Problem**: Processes exit unexpectedly or hang
```
Process exits with non-zero code
```

**Solutions**:
```bash
# Check process exit codes
echo "Exit code: $?"

# Run with error handling
docker exec <container_name> sh -c "<command> || echo 'Command failed'"

# Use timeout for hanging processes
timeout 30 docker exec <container_name> <command>

# Kill hanging processes
docker exec <container_name> pkill -f <process_name>
```

## Debugging Commands

### Container State Analysis
```bash
# Check container details
docker inspect <container_name>

# Check container processes
docker exec <container_name> ps aux

# Check container resource usage
docker stats <container_name> --no-stream

# Check container filesystem
docker exec <container_name> df -h
```

### Process Investigation
```bash
# List running processes
docker exec <container_name> ps auxf

# Check process tree
docker exec <container_name> pstree

# Monitor process activity
docker exec <container_name> top

# Check process resource usage
docker exec <container_name> top -p <pid>
```

### System Information
```bash
# Check system information
docker exec <container_name> uname -a

# Check available memory
docker exec <container_name> free -h

# Check disk usage
docker exec <container_name> du -sh /

# Check network interfaces
docker exec <container_name> ip addr show
```

### Environment Analysis
```bash
# Check environment variables
docker exec <container_name> printenv | sort

# Check PATH variable
docker exec <container_name> echo $PATH

# Check shell configuration
docker exec <container_name> cat /etc/passwd
docker exec <container_name> cat ~/.bashrc

# Check available commands
docker exec <container_name> compgen -c | sort
```

### Network Diagnostics
```bash
# Check network connectivity
docker exec <container_name> ping -c 3 google.com

# Check DNS resolution
docker exec <container_name> nslookup google.com

# Check open ports
docker exec <container_name> netstat -tlnp

# Check routing table
docker exec <container_name> ip route show
```

## Error Codes and Meanings

### Standard Exit Codes
- **0** - Successful execution
- **1** - General error
- **2** - Misuse of shell command
- **126** - Command not executable
- **127** - Command not found
- **128** - Invalid exit argument
- **130** - Script terminated by Ctrl+C

### Docker-Specific Errors
- **Container not running** - Target container is stopped
- **Command not found** - Specified command doesn't exist
- **Permission denied** - Insufficient privileges
- **No such file or directory** - Path or file doesn't exist
- **OCI runtime error** - Container runtime issue

### Checking Execution Status
```bash
# Check last command exit code
echo $?

# Run command with exit code capture
docker exec <container_name> <command>
exit_code=$?
echo "Command exited with code: $exit_code"

# Conditional execution based on exit code
if docker exec <container_name> <command>; then
    echo "Command succeeded"
else
    echo "Command failed"
fi
```

## Recovery Procedures

### Restart Container and Retry
```bash
# Step 1: Check container status
docker ps -a --filter name=<container_name>

# Step 2: Check container logs
docker logs <container_name> --tail 50

# Step 3: Restart container
docker restart <container_name>

# Step 4: Wait for container to be ready
sleep 10

# Step 5: Retry exec command
docker exec -it <container_name> bash
```

### Alternative Shell Access
```bash
#!/bin/bash
# Try multiple shells for container access
container="$1"
shells=("bash" "sh" "ash" "zsh" "/bin/bash" "/bin/sh")

for shell in "${shells[@]}"; do
    echo "Trying $shell..."
    if docker exec -it "$container" "$shell" 2>/dev/null; then
        echo "Successfully connected with $shell"
        exit 0
    fi
done

echo "No working shell found"
exit 1
```

### Permission Recovery
```bash
#!/bin/bash
# Fix permission issues in container
container="$1"
path="${2:-/app}"

echo "Fixing permissions in $container:$path"

# Try as current user first
if docker exec "$container" test -r "$path"; then
    echo "Path accessible with current user"
    exit 0
fi

# Try as root
echo "Attempting to fix permissions as root..."
docker exec --user root "$container" chown -R "$(id -u):$(id -g)" "$path"
docker exec --user root "$container" chmod -R 755 "$path"

# Verify fix
if docker exec "$container" test -r "$path"; then
    echo "Permissions fixed successfully"
else
    echo "Failed to fix permissions"
    exit 1
fi
```

### Environment Recovery
```bash
#!/bin/bash
# Recover from environment issues
container="$1"
command="${2:-bash}"

echo "Recovering environment for $container..."

# Method 1: Use login shell
echo "Trying login shell..."
if docker exec -it "$container" su - root; then
    exit 0
fi

# Method 2: Source environment files
echo "Sourcing environment files..."
docker exec -it "$container" sh -c "
source /etc/profile 2>/dev/null || true
source ~/.profile 2>/dev/null || true
source ~/.bashrc 2>/dev/null || true
exec $command
"

# Method 3: Set minimal environment
echo "Setting minimal environment..."
docker exec -it -e PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" \
    "$container" "$command"
```

### Emergency Access Script
```bash
#!/bin/bash
# Emergency container access when normal exec fails
container="$1"

echo "=== Emergency Container Access ==="
echo "Container: $container"

# Check if container exists and is running
if ! docker ps --format "{{.Names}}" | grep -q "^$container$"; then
    echo "Container is not running. Attempting to start..."
    docker start "$container"
    sleep 5
fi

# Try different access methods
echo "Method 1: Standard bash access"
if docker exec -it "$container" bash 2>/dev/null; then
    exit 0
fi

echo "Method 2: Root access"
if docker exec -it --user root "$container" bash 2>/dev/null; then
    exit 0
fi

echo "Method 3: Alternative shells"
for shell in sh ash dash; do
    echo "Trying $shell..."
    if docker exec -it "$container" "$shell" 2>/dev/null; then
        exit 0
    fi
done

echo "Method 4: Command execution"
echo "Available commands:"
docker exec "$container" ls /bin 2>/dev/null || echo "Cannot list /bin"

echo "Method 5: Container inspection"
echo "Container details:"
docker inspect "$container" --format '{{.State.Status}}'
docker inspect "$container" --format '{{.Config.Entrypoint}}'
docker inspect "$container" --format '{{.Config.Cmd}}'

echo "All access methods failed. Container may need to be recreated."
exit 1
```

### Network Connectivity Recovery
```bash
#!/bin/bash
# Recover network connectivity in container
container="$1"

echo "Diagnosing network connectivity for $container..."

# Check network configuration
echo "Network interfaces:"
docker exec "$container" ip addr show 2>/dev/null || echo "Cannot check interfaces"

echo "Routing table:"
docker exec "$container" ip route show 2>/dev/null || echo "Cannot check routes"

echo "DNS configuration:"
docker exec "$container" cat /etc/resolv.conf 2>/dev/null || echo "Cannot check DNS"

# Test connectivity
echo "Testing connectivity:"
if docker exec "$container" ping -c 1 8.8.8.8 >/dev/null 2>&1; then
    echo "✓ IP connectivity working"
else
    echo "✗ IP connectivity failed"
fi

if docker exec "$container" nslookup google.com >/dev/null 2>&1; then
    echo "✓ DNS resolution working"
else
    echo "✗ DNS resolution failed"
fi

# Suggest recovery actions
echo "Recovery suggestions:"
echo "1. Restart container: docker restart $container"
echo "2. Check Docker network: docker network ls"
echo "3. Recreate container with --network host for testing"
echo "4. Check host network connectivity"
```
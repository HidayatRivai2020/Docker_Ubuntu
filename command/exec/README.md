# exec

## Overview
- Executes commands inside running Docker containers
- Provides interactive access to container environments
- Does not create new containers - works only with existing running containers
- Essential tool for debugging, administration, and development workflows
- Maintains container state and environment throughout execution

## Basic Syntax
```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

## Common Options

### Interactive Options
- `-i, --interactive` - Keep STDIN open even if not attached
- `-t, --tty` - Allocate a pseudo-TTY for interactive sessions
- `-d, --detach` - Run command in background (detached mode)

### Environment Options
- `-e, --env` - Set environment variables
- `--env-file` - Read environment variables from file
- `-w, --workdir` - Working directory inside the container
- `--user` - Username or UID (format: <name|uid>[:<group|gid>])

### Privilege Options
- `--privileged` - Give extended privileges to the command
- `--group-add` - Add additional groups to run as

## How It Works

### Execution Process
1. **Container Validation** - Verifies target container is running
2. **Namespace Entry** - Enters container's namespaces (PID, network, filesystem)
3. **Environment Setup** - Applies specified environment variables and working directory
4. **Process Creation** - Spawns new process within container context
5. **I/O Handling** - Manages input/output streams between host and container
6. **Resource Isolation** - Maintains container's resource limits and security context
7. **Exit Handling** - Properly terminates process and returns exit code

### Container Context
- **Filesystem** - Access to container's filesystem and mounted volumes
- **Network** - Uses container's network interfaces and configuration
- **Process Space** - Runs within container's process namespace
- **User Context** - Inherits or overrides container's user permissions

### Session Types
- **Interactive Shell** - Full terminal access with TTY allocation
- **Single Command** - Execute specific command and exit
- **Background Tasks** - Long-running processes in detached mode
- **Administrative Tasks** - System maintenance and debugging operations

## Best Practices

- **Use interactive mode** - Combine `-it` flags for shell access
- **Specify working directory** - Use `-w` to set appropriate context
- **Mind user permissions** - Use `--user` to avoid running as root
- **Clean command syntax** - Use proper shell quoting for complex commands
- **Exit cleanly** - Always exit interactive sessions properly
- **Avoid long-running processes** - Use detached mode for background tasks
- **Security conscious** - Limit privileges and avoid `--privileged` when possible
- **Test commands first** - Verify commands work before automation
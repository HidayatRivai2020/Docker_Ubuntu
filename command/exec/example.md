# Docker exec Examples

## Basic Examples

### Interactive Shell Access
```bash
# Start interactive bash shell
docker exec -it mycontainer bash

# Start interactive shell with specific shell
docker exec -it mycontainer sh
docker exec -it mycontainer zsh
docker exec -it mycontainer /bin/bash

# Access container by ID
docker exec -it 1a2b3c4d5e6f bash
```

### Single Command Execution
```bash
# Execute simple command
docker exec mycontainer ls -la

# Check container processes
docker exec mycontainer ps aux

# View system information
docker exec mycontainer uname -a

# Check disk usage
docker exec mycontainer df -h
```

### File Operations
```bash
# View file contents
docker exec mycontainer cat /etc/passwd

# Create file
docker exec mycontainer touch /tmp/testfile

# Edit file (if editor available)
docker exec -it mycontainer nano /etc/config.txt

# Check file permissions
docker exec mycontainer ls -la /app/config
```

## Environment and Working Directory

### Setting Environment Variables
```bash
# Set single environment variable
docker exec -e NODE_ENV=production mycontainer node app.js

# Set multiple environment variables
docker exec -e VAR1=value1 -e VAR2=value2 mycontainer env

# Use environment file
docker exec --env-file .env mycontainer printenv

# Override existing environment
docker exec -e PATH=/custom/path mycontainer echo $PATH
```

### Working Directory
```bash
# Execute command in specific directory
docker exec -w /app mycontainer ls -la

# Combine working directory with interactive shell
docker exec -it -w /var/log mycontainer bash

# Run application from specific directory
docker exec -w /app mycontainer npm start

# Check current working directory
docker exec -w /tmp mycontainer pwd
```

## User and Permissions

### Running as Different User
```bash
# Run as specific user
docker exec --user postgres mycontainer whoami

# Run as user ID
docker exec --user 1000 mycontainer id

# Run as user and group
docker exec --user 1000:1000 mycontainer id

# Run as root (if needed for admin tasks)
docker exec --user root mycontainer apt update
```

### Permission Checking
```bash
# Check current user
docker exec mycontainer whoami

# Check user ID and groups
docker exec mycontainer id

# Check sudo access
docker exec mycontainer sudo -l

# Test file permissions
docker exec mycontainer ls -la /app
```

## Advanced Examples

### Background Tasks
```bash
# Run command in background
docker exec -d mycontainer python background_task.py

# Start background service
docker exec -d mycontainer service nginx start

# Run monitoring script
docker exec -d mycontainer ./monitor.sh

# Background file processing
docker exec -d mycontainer find /data -name "*.log" -delete
```

### Complex Commands
```bash
# Pipeline commands
docker exec mycontainer sh -c "ps aux | grep nginx"

# Command with redirects
docker exec mycontainer sh -c "echo 'test' > /tmp/test.txt"

# Multiple commands
docker exec mycontainer sh -c "cd /app && npm install && npm start"

# Conditional execution
docker exec mycontainer sh -c "[ -f /app/config.json ] && echo 'Config exists'"
```

### Network and Service Testing
```bash
# Test network connectivity
docker exec mycontainer ping google.com

# Check open ports
docker exec mycontainer netstat -tlnp

# Test HTTP endpoints
docker exec mycontainer curl -I http://localhost:8080

# Check DNS resolution
docker exec mycontainer nslookup google.com
```

## Real-World Use Cases

### 1. Database Administration
```bash
# MySQL administration
docker exec -it mysql_container mysql -u root -p

# PostgreSQL administration
docker exec -it postgres_container psql -U postgres

# Database backup
docker exec mysql_container mysqldump -u root -p database_name > backup.sql

# Database restore
docker exec -i mysql_container mysql -u root -p database_name < backup.sql

# Check database status
docker exec postgres_container pg_isready
```

### 2. Web Server Management
```bash
# Nginx operations
docker exec nginx_container nginx -t  # Test configuration
docker exec nginx_container nginx -s reload  # Reload configuration

# Apache operations
docker exec apache_container httpd -t
docker exec apache_container apachectl graceful

# Check web server logs
docker exec nginx_container tail -f /var/log/nginx/access.log

# Test web server response
docker exec nginx_container curl -I http://localhost
```

### 3. Application Debugging
```bash
# Node.js debugging
docker exec -it nodejs_app bash
docker exec nodejs_app npm run debug

# Python debugging
docker exec -it python_app python -c "import pdb; pdb.set_trace()"

# Java debugging
docker exec java_app jstack <pid>
docker exec java_app jmap -dump:format=b,file=/tmp/heap.hprof <pid>

# Check application logs
docker exec app_container tail -f /app/logs/application.log
```

### 4. System Monitoring
```bash
# Resource monitoring
docker exec mycontainer top
docker exec mycontainer htop
docker exec mycontainer iostat

# Memory usage
docker exec mycontainer free -h
docker exec mycontainer cat /proc/meminfo

# Disk usage
docker exec mycontainer df -h
docker exec mycontainer du -sh /app

# Network monitoring
docker exec mycontainer ss -tlnp
docker exec mycontainer iftop
```

### 5. Security and Maintenance
```bash
# Security scanning
docker exec mycontainer find / -perm -4000 -type f  # Find SUID files
docker exec mycontainer netstat -tlnp  # Check open ports

# System updates
docker exec --user root ubuntu_container apt update
docker exec --user root ubuntu_container apt upgrade -y

# Log analysis
docker exec mycontainer grep "ERROR" /var/log/app.log
docker exec mycontainer journalctl --since "1 hour ago"

# Configuration validation
docker exec nginx_container nginx -t
docker exec mycontainer python -m py_compile /app/config.py
```

## Development Workflows

### Code Development
```bash
# Install development dependencies
docker exec -it dev_container npm install
docker exec -it dev_container pip install -r requirements.txt

# Run tests
docker exec dev_container npm test
docker exec dev_container python -m pytest
docker exec dev_container mvn test

# Code formatting and linting
docker exec dev_container eslint src/
docker exec dev_container black --check .
docker exec dev_container go fmt ./...

# Interactive development shell
docker exec -it -w /workspace dev_container bash
```

### Hot Reloading and Live Development
```bash
# Start development server with hot reload
docker exec -d -w /app dev_container npm run dev

# Watch for file changes
docker exec -d dev_container nodemon app.js

# Run development tools
docker exec -it dev_container webpack --watch
docker exec -it dev_container gulp watch
```

## Scripting and Automation

### Health Check Script
```bash
#!/bin/bash
# Container health check script
container="$1"

echo "=== Health Check for $container ==="

# Check if container is running
if ! docker ps --format "{{.Names}}" | grep -q "^$container$"; then
    echo "Container $container is not running"
    exit 1
fi

# Check processes
echo "Running processes:"
docker exec "$container" ps aux | head -10

# Check memory usage
echo "Memory usage:"
docker exec "$container" free -h

# Check disk space
echo "Disk usage:"
docker exec "$container" df -h | grep -E "(Filesystem|/$)"

# Application-specific checks
if docker exec "$container" curl -f http://localhost:8080/health 2>/dev/null; then
    echo "✓ Application health check passed"
else
    echo "✗ Application health check failed"
fi
```

### Batch Command Execution
```bash
#!/bin/bash
# Execute commands across multiple containers
containers=("web1" "web2" "api1" "api2")
command="systemctl status nginx"

echo "Executing '$command' across containers..."

for container in "${containers[@]}"; do
    echo "=== $container ==="
    if docker exec "$container" $command; then
        echo "✓ Success on $container"
    else
        echo "✗ Failed on $container"
    fi
    echo
done
```

### Interactive Container Shell Function
```bash
# Add to ~/.bashrc or ~/.zshrc
dsh() {
    local container="$1"
    local shell="${2:-bash}"
    
    if [ -z "$container" ]; then
        echo "Usage: dsh <container> [shell]"
        return 1
    fi
    
    # Check if container is running
    if ! docker ps --format "{{.Names}}" | grep -q "^$container$"; then
        echo "Container '$container' is not running"
        return 1
    fi
    
    # Try different shells if specified one fails
    for try_shell in "$shell" bash sh; do
        if docker exec -it "$container" which "$try_shell" >/dev/null 2>&1; then
            echo "Connecting to $container with $try_shell..."
            docker exec -it "$container" "$try_shell"
            return
        fi
    done
    
    echo "No suitable shell found in container '$container'"
}

# Usage: dsh mycontainer
# Usage: dsh mycontainer zsh
```

### Log Monitoring Script
```bash
#!/bin/bash
# Multi-container log monitoring
containers=("web" "api" "db")
log_files=(
    "/var/log/nginx/error.log"
    "/app/logs/application.log"
    "/var/log/postgresql/postgresql.log"
)

echo "Starting log monitoring..."

for i in "${!containers[@]}"; do
    container="${containers[$i]}"
    log_file="${log_files[$i]}"
    
    echo "Monitoring $container:$log_file"
    docker exec -d "$container" tail -f "$log_file" | sed "s/^/[$container] /" &
done

# Wait for interrupt
trap 'kill $(jobs -p)' EXIT
wait
```
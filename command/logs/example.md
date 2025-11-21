# Docker logs Examples

## Basic Examples

### View Container Logs
```bash
# View all logs from a container
docker logs mycontainer

# View logs from container by ID
docker logs 1a2b3c4d5e6f

# View logs with timestamps
docker logs -t mycontainer

# Follow logs in real-time
docker logs -f mycontainer

# Combine timestamp and follow
docker logs -tf mycontainer
```

### Limit Output
```bash
# Show only last 100 lines
docker logs --tail 100 mycontainer

# Show last 50 lines with timestamps
docker logs -t --tail 50 mycontainer

# Follow logs showing only new entries
docker logs -f --tail 0 mycontainer

# Show specific number of lines
docker logs -n 20 mycontainer
```

### Time-Based Filtering
```bash
# Logs from the last hour
docker logs --since 1h mycontainer

# Logs from last 30 minutes
docker logs --since 30m mycontainer

# Logs from specific date
docker logs --since "2023-12-01" mycontainer

# Logs from specific datetime
docker logs --since "2023-12-01T10:00:00" mycontainer

# Logs between specific times
docker logs --since "2023-12-01T10:00:00" --until "2023-12-01T12:00:00" mycontainer
```

### Advanced Time Filtering
```bash
# Logs from yesterday
docker logs --since "$(date -d 'yesterday' '+%Y-%m-%d')" mycontainer

# Logs from last 24 hours
docker logs --since "24h" mycontainer

# Logs from specific duration ago
docker logs --since "2h30m" mycontainer

# Logs until 1 hour ago
docker logs --until "1h" mycontainer

# Logs from last week
docker logs --since "168h" mycontainer
```

## Filtering and Processing

### Using grep for Pattern Matching
```bash
# Filter logs for ERROR messages
docker logs mycontainer | grep ERROR

# Filter logs for specific patterns
docker logs mycontainer | grep -i "database"

# Filter logs with context (show surrounding lines)
docker logs mycontainer | grep -A 3 -B 3 "ERROR"

# Filter logs with timestamps
docker logs -t mycontainer | grep "$(date '+%Y-%m-%d').*ERROR"

# Case-insensitive search
docker logs mycontainer | grep -i "warning\|error\|fail"
```

### Using awk and sed for Processing
```bash
# Extract only log messages (remove container info)
docker logs mycontainer | sed 's/^[^ ]* //'

# Show only logs from specific time pattern
docker logs -t mycontainer | awk '/2023-12-01 10:/ {print}'

# Count ERROR occurrences
docker logs mycontainer | grep -c "ERROR"

# Show unique error messages
docker logs mycontainer | grep "ERROR" | sort | uniq

# Extract JSON logs and format them
docker logs mycontainer | grep '^{' | jq .
```

### Advanced Log Analysis
```bash
# Show logs with line numbers
docker logs mycontainer | nl

# Show logs with word count
docker logs mycontainer | wc -l

# Analyze log patterns by hour
docker logs -t mycontainer | awk '{print $1" "$2}' | cut -d: -f1-2 | sort | uniq -c

# Find most frequent log messages
docker logs mycontainer | sort | uniq -c | sort -nr | head -10
```

## Real-Time Monitoring

### Following Multiple Containers
```bash
# Follow logs from multiple containers in separate terminals
docker logs -f web &
docker logs -f db &
docker logs -f cache &

# Monitor multiple containers with labels
for container in $(docker ps --filter "label=app=myapp" --format "{{.Names}}"); do
    echo "Following logs for $container"
    gnome-terminal -- bash -c "docker logs -f $container; bash"
done

# Follow logs with container names
docker logs -f web 2>&1 | sed 's/^/[WEB] /' &
docker logs -f db 2>&1 | sed 's/^/[DB] /' &
```

### Real-Time Filtering
```bash
# Follow only ERROR logs
docker logs -f mycontainer | grep --line-buffered "ERROR"

# Follow logs with timestamps and filter
docker logs -tf mycontainer | grep --line-buffered "$(date '+%Y-%m-%d')"

# Follow and highlight specific patterns
docker logs -f mycontainer | grep --line-buffered --color=always "ERROR\|WARNING"

# Follow and save to file while displaying
docker logs -f mycontainer | tee /tmp/container_logs.txt
```

## Troubleshooting Examples

### Application Debugging
```bash
# Debug application startup issues
docker logs --tail 50 mycontainer | grep -i "error\|fail\|exception"

# Check for specific error codes
docker logs mycontainer | grep "HTTP [45][0-9][0-9]"

# Monitor memory issues
docker logs mycontainer | grep -i "memory\|oom\|killed"

# Check database connection issues
docker logs mycontainer | grep -i "connection\|database\|sql"
```

### Performance Monitoring
```bash
# Monitor response times
docker logs -t mycontainer | grep "response_time" | tail -20

# Check for timeout issues
docker logs mycontainer | grep -i "timeout\|slow\|delay"

# Monitor request patterns
docker logs mycontainer | grep "GET\|POST\|PUT\|DELETE" | tail -10

# Track error rates over time
docker logs -t mycontainer | grep "ERROR" | awk '{print $1" "$2}' | cut -d: -f1-2 | sort | uniq -c
```

### System Diagnostics
```bash
# Check container health
docker logs --since 10m mycontainer | grep -i "health\|ready\|alive"

# Monitor startup sequence
docker logs mycontainer | head -50

# Check shutdown sequence
docker logs --tail 50 mycontainer | tail -20

# Monitor resource usage logs
docker logs mycontainer | grep -i "cpu\|memory\|disk\|network"
```

## Real-World Use Cases

### 1. Application Error Monitoring
```bash
#!/bin/bash
# Monitor application errors across multiple containers
app_name="myapp"
echo "=== Error Monitor for $app_name ==="
echo "Started: $(date)"

# Get all containers for the app
containers=$(docker ps --filter "label=app=$app_name" --format "{{.Names}}")

if [ -z "$containers" ]; then
    echo "No containers found for app: $app_name"
    exit 1
fi

echo "Monitoring containers: $containers"
echo "Press Ctrl+C to stop monitoring"
echo

# Monitor each container for errors
for container in $containers; do
    {
        docker logs -f --since 1m "$container" | \
        grep --line-buffered -i "error\|exception\|fail\|critical" | \
        sed "s/^/[$container] /"
    } &
done

# Wait for user to stop
wait
```

### 2. Log Aggregation Script
```bash
#!/bin/bash
# Aggregate logs from multiple containers
output_dir="/tmp/docker_logs"
mkdir -p "$output_dir"

echo "=== Docker Log Aggregation ==="
echo "Output directory: $output_dir"
echo "Timestamp: $(date)"
echo

# Get all running containers
containers=$(docker ps --format "{{.Names}}")

for container in $containers; do
    echo "Collecting logs from: $container"
    
    # Get logs from last 24 hours
    docker logs --since 24h -t "$container" > "$output_dir/${container}_24h.log" 2>&1
    
    # Get error logs
    docker logs --since 24h "$container" 2>&1 | \
        grep -i "error\|exception\|fail" > "$output_dir/${container}_errors.log"
    
    # Get summary statistics
    echo "Container: $container" >> "$output_dir/summary.txt"
    echo "Total lines: $(docker logs --since 24h "$container" 2>&1 | wc -l)" >> "$output_dir/summary.txt"
    echo "Error lines: $(docker logs --since 24h "$container" 2>&1 | grep -ci "error\|exception\|fail")" >> "$output_dir/summary.txt"
    echo "---" >> "$output_dir/summary.txt"
done

echo "Log collection complete. Files saved to: $output_dir"
ls -la "$output_dir"
```

### 3. Performance Analysis
```bash
#!/bin/bash
# Analyze container performance from logs
container="$1"

if [ -z "$container" ]; then
    echo "Usage: $0 <container_name>"
    exit 1
fi

echo "=== Performance Analysis for $container ==="

# Response time analysis
echo "Response Time Analysis:"
docker logs --since 1h "$container" | \
    grep -o "response_time=[0-9]*" | \
    cut -d= -f2 | \
    awk '{
        sum += $1
        count++
        if ($1 > max || count == 1) max = $1
        if ($1 < min || count == 1) min = $1
    } END {
        if (count > 0) {
            printf "Average: %.2f ms\n", sum/count
            printf "Min: %d ms\n", min
            printf "Max: %d ms\n", max
        }
    }'

# Error rate analysis
echo
echo "Error Rate Analysis (last hour):"
total_requests=$(docker logs --since 1h "$container" | grep -c "HTTP")
error_requests=$(docker logs --since 1h "$container" | grep -c "HTTP [45][0-9][0-9]")

if [ "$total_requests" -gt 0 ]; then
    error_rate=$(echo "scale=2; $error_requests * 100 / $total_requests" | bc)
    echo "Total requests: $total_requests"
    echo "Error requests: $error_requests"
    echo "Error rate: ${error_rate}%"
else
    echo "No HTTP requests found in logs"
fi

# Request patterns
echo
echo "Top Request Patterns:"
docker logs --since 1h "$container" | \
    grep "HTTP" | \
    awk '{print $(NF-1)}' | \
    sort | uniq -c | sort -nr | head -10
```

### 4. Health Check Monitor
```bash
#!/bin/bash
# Monitor container health through logs
echo "=== Container Health Monitor ==="

# Function to check container health
check_container_health() {
    local container=$1
    local status=$(docker inspect --format='{{.State.Status}}' "$container" 2>/dev/null)
    
    echo "Container: $container"
    echo "Status: $status"
    
    if [ "$status" = "running" ]; then
        # Check for recent errors
        error_count=$(docker logs --since 5m "$container" 2>&1 | grep -ci "error\|exception\|fail")
        echo "Recent errors (5min): $error_count"
        
        # Check for health indicators
        health_logs=$(docker logs --since 5m "$container" 2>&1 | grep -i "health\|ready\|alive")
        if [ -n "$health_logs" ]; then
            echo "Health indicators:"
            echo "$health_logs" | tail -3
        fi
        
        # Check for memory issues
        memory_issues=$(docker logs --since 5m "$container" 2>&1 | grep -ci "oom\|memory\|killed")
        if [ "$memory_issues" -gt 0 ]; then
            echo "⚠️ Memory issues detected: $memory_issues"
        fi
    fi
    echo
}

# Monitor all running containers
for container in $(docker ps --format "{{.Names}}"); do
    check_container_health "$container"
done

# Check for containers that died recently
echo "Recently Stopped Containers:"
docker ps -a --filter "status=exited" --filter "exited=1" --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

echo
echo "Recent Container Events:"
docker system events --since 1h --until now | grep container | tail -10
```

### 5. Log Rotation and Cleanup
```bash
#!/bin/bash
# Log rotation and cleanup script
max_size_mb=100
backup_dir="/var/backup/docker_logs"

echo "=== Docker Log Rotation ==="
echo "Max log size: ${max_size_mb}MB"
echo "Backup directory: $backup_dir"

mkdir -p "$backup_dir"

# Function to rotate logs for a container
rotate_container_logs() {
    local container=$1
    local log_file="/var/lib/docker/containers/$container/$container-json.log"
    
    if [ ! -f "$log_file" ]; then
        return
    fi
    
    # Check log file size
    local size_mb=$(du -m "$log_file" | cut -f1)
    
    echo "Container: $container"
    echo "Log size: ${size_mb}MB"
    
    if [ "$size_mb" -gt "$max_size_mb" ]; then
        echo "Rotating logs for $container"
        
        # Create backup
        local timestamp=$(date '+%Y%m%d_%H%M%S')
        local backup_file="$backup_dir/${container}_${timestamp}.log"
        
        # Copy and compress log
        cp "$log_file" "$backup_file"
        gzip "$backup_file"
        
        # Truncate original log
        truncate -s 0 "$log_file"
        
        echo "Backup created: ${backup_file}.gz"
    else
        echo "Log size OK"
    fi
    echo
}

# Rotate logs for all containers
for container_id in $(docker ps -aq); do
    rotate_container_logs "$container_id"
done

# Clean old backups (older than 30 days)
echo "Cleaning old backups..."
find "$backup_dir" -name "*.gz" -mtime +30 -delete

echo "Log rotation completed"
```

### 6. Real-Time Dashboard
```bash
#!/bin/bash
# Simple real-time log dashboard
clear

while true; do
    echo "=== Docker Logs Dashboard $(date) ==="
    echo
    
    # Container status summary
    echo "Container Status:"
    printf "%-20s %-15s %-10s %s\n" "NAME" "STATUS" "ERRORS(5m)" "LAST_ERROR"
    printf "%-20s %-15s %-10s %s\n" "----" "------" "---------" "----------"
    
    for container in $(docker ps --format "{{.Names}}"); do
        status=$(docker inspect --format='{{.State.Status}}' "$container")
        error_count=$(docker logs --since 5m "$container" 2>&1 | grep -ci "error" || echo "0")
        last_error=$(docker logs --since 5m "$container" 2>&1 | grep -i "error" | tail -1 | cut -c1-50)
        
        printf "%-20s %-15s %-10s %s\n" "$container" "$status" "$error_count" "$last_error"
    done
    
    echo
    echo "Recent Critical Events:"
    docker logs --since 1m $(docker ps -q) 2>&1 | \
        grep -i "critical\|fatal\|emergency" | \
        tail -5 | \
        while read line; do
            echo "  $(date '+%H:%M:%S'): $line"
        done
    
    echo
    echo "System Resources:"
    echo "  CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)% used"
    echo "  Memory: $(free | awk 'NR==2{printf "%.1f%%", $3*100/$2 }')"
    echo "  Disk: $(df -h / | awk 'NR==2{print $5}')"
    
    echo
    echo "Press Ctrl+C to exit"
    sleep 10
    clear
done
```
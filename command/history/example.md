# Examples

## Basic Examples
```bash
# View image history
docker history nginx:latest

# View history with full commands
docker history --no-trunc nginx:latest

# Show only image IDs
docker history -q nginx:latest

# View history with human-readable output
docker history -H nginx:latest
```

## Format Examples
```bash
# Format as table with specific columns
docker history --format "table {{.ID}}\t{{.CreatedBy}}\t{{.Size}}" nginx:latest

# Format as JSON
docker history --format json nginx:latest

# Show only size and command
docker history --format "{{.Size}}\t{{.CreatedBy}}" nginx:latest

# Show creation time and size
docker history --format "{{.CreatedAt}}\t{{.Size}}" nginx:latest
```

## Analysis Examples
```bash
# Find layers larger than 10MB
docker history nginx:latest | grep MB | awk '$5+0 > 10 {print $0}'

# Count total layers
docker history nginx:latest | wc -l

# Sort layers by size
docker history --format "{{.Size}}\t{{.CreatedBy}}" nginx:latest | sort -h

# Check base image
docker history nginx:latest | tail -5

# Export history to file
docker history --no-trunc nginx:latest > nginx-history.txt
```

## Size Investigation
```bash
# Identify large layers in Python image
docker history python:3.9

# Find which RUN commands add most size
docker history --no-trunc myapp:latest | grep "RUN"

# Compare image sizes
docker history alpine:latest | head -1
docker history ubuntu:latest | head -1

# Calculate total size from layers
docker history myapp:latest --format "{{.Size}}" | grep MB
```

## Common Use Cases

### Debugging Build Issues
```bash
# Check what commands were executed
docker history --no-trunc myapp:broken

# Find the last successful layer
docker history myapp:broken | head -10

# Verify Dockerfile instructions
docker history --no-trunc myapp:latest | grep "ENV"
docker history --no-trunc myapp:latest | grep "EXPOSE"
docker history --no-trunc myapp:latest | grep "CMD"
```

### Image Optimization
```bash
# Before optimization - multiple RUN commands
docker history myapp:unoptimized

# After optimization - combined RUN commands
docker history myapp:optimized

# Compare layer counts
echo "Unoptimized layers: $(docker history myapp:unoptimized | wc -l)"
echo "Optimized layers: $(docker history myapp:optimized | wc -l)"

# Compare sizes
docker images myapp:unoptimized myapp:optimized
```

### Security Auditing
```bash
# Check for secrets in commands
docker history --no-trunc myapp:latest | grep -i "password\|secret\|key\|token"

# Check environment variables
docker history --no-trunc myapp:latest | grep "ENV"

# Look for sensitive files
docker history --no-trunc myapp:latest | grep -E "\.pem|\.key|id_rsa"

# Check for root user operations
docker history --no-trunc myapp:latest | grep "USER"
```

### Version Comparison
```bash
# Compare two versions
docker history myapp:v1.0 > v1-history.txt
docker history myapp:v2.0 > v2-history.txt
diff v1-history.txt v2-history.txt

# Track image growth over versions
for tag in 1.0 1.1 1.2 2.0; do
    echo "Version $tag:"
    docker history myapp:$tag | head -1
done

# Check what changed between versions
docker history --format "{{.CreatedBy}}" myapp:v1.0 > v1-commands.txt
docker history --format "{{.CreatedBy}}" myapp:v2.0 > v2-commands.txt
diff v1-commands.txt v2-commands.txt
```

## CI/CD Integration

### GitHub Actions
```yaml
name: Analyze Image History

on: [push]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Check layer count
        run: |
          LAYERS=$(docker history myapp:${{ github.sha }} | wc -l)
          echo "Image has $LAYERS layers"
          if [ $LAYERS -gt 20 ]; then
            echo "::warning::Too many layers: $LAYERS"
          fi
      
      - name: Find large layers
        run: |
          echo "Layers larger than 50MB:"
          docker history myapp:${{ github.sha }} | grep MB | awk '$5+0 > 50 {print $0}'
      
      - name: Export history
        run: docker history --no-trunc myapp:${{ github.sha }} > history.txt
```

### GitLab CI
```yaml
analyze-history:
  stage: test
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    
    # Check layer count
    - |
      LAYERS=$(docker history $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA | wc -l)
      echo "Layers: $LAYERS"
      if [ $LAYERS -gt 25 ]; then
        echo "Warning: Too many layers"
      fi
    
    # Export history
    - docker history --no-trunc $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA > history.txt
  
  artifacts:
    paths:
      - history.txt
```

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        stage('Analyze History') {
            steps {
                script {
                    def layers = sh(
                        script: "docker history myapp:${BUILD_NUMBER} | wc -l",
                        returnStdout: true
                    ).trim()
                    
                    echo "Image has ${layers} layers"
                    
                    if (layers.toInteger() > 20) {
                        unstable("Too many layers: ${layers}")
                    }
                }
                
                sh '''
                    docker history --no-trunc myapp:${BUILD_NUMBER} > history-${BUILD_NUMBER}.txt
                    docker history myapp:${BUILD_NUMBER} | grep MB | awk '$5+0 > 50' || true
                '''
                
                archiveArtifacts artifacts: 'history-*.txt'
            }
        }
    }
}
```

## Scripting Examples

### Layer Count Check Script
```bash
#!/bin/bash
# check-layers.sh - Validate layer count

IMAGE=$1
MAX_LAYERS=${2:-20}

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image> [max_layers]"
    exit 1
fi

LAYERS=$(docker history $IMAGE | tail -n +2 | wc -l)

echo "Image: $IMAGE"
echo "Layers: $LAYERS"
echo "Maximum: $MAX_LAYERS"

if [ $LAYERS -gt $MAX_LAYERS ]; then
    echo "ERROR: Too many layers ($LAYERS > $MAX_LAYERS)"
    exit 1
else
    echo "OK: Layer count within limits"
fi
```

### Size Analysis Script
```bash
#!/bin/bash
# analyze-size.sh - Analyze image layer sizes

IMAGE=$1

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image>"
    exit 1
fi

echo "=== Image Layer Analysis for $IMAGE ==="
echo ""

echo "Total Image Size:"
docker images $IMAGE --format "{{.Size}}"
echo ""

echo "Layer Count:"
docker history $IMAGE | tail -n +2 | wc -l
echo ""

echo "Top 5 Largest Layers:"
docker history $IMAGE --format "table {{.Size}}\t{{.CreatedBy}}" | \
    grep MB | sort -rh | head -5
echo ""

echo "Zero-size Layers (metadata):"
docker history $IMAGE | grep "0B" | wc -l
echo ""

echo "Reclaimable Space Estimation:"
docker history $IMAGE | grep "apt-get\|yum\|apk" | wc -l
echo "layers with package manager operations (may benefit from cleanup)"
```

### Comparison Script
```bash
#!/bin/bash
# compare-images.sh - Compare layer history of multiple images

if [ $# -lt 2 ]; then
    echo "Usage: $0 <image1> <image2> [image3...]"
    exit 1
fi

echo "=== Image Comparison Report ==="
printf "%-30s %-10s %-10s %-15s\n" "IMAGE" "LAYERS" "SIZE" "CREATED"
echo "-------------------------------------------------------------------"

for IMAGE in "$@"; do
    LAYERS=$(docker history $IMAGE 2>/dev/null | tail -n +2 | wc -l)
    SIZE=$(docker images $IMAGE --format "{{.Size}}" 2>/dev/null)
    CREATED=$(docker history $IMAGE --format "{{.CreatedAt}}" 2>/dev/null | head -1)
    
    if [ $? -eq 0 ]; then
        printf "%-30s %-10s %-10s %-15s\n" "$IMAGE" "$LAYERS" "$SIZE" "$CREATED"
    else
        printf "%-30s %-10s\n" "$IMAGE" "NOT FOUND"
    fi
done
```

### Audit Script
```bash
#!/bin/bash
# audit-image.sh - Security audit of image layers

IMAGE=$1

if [ -z "$IMAGE" ]; then
    echo "Usage: $0 <image>"
    exit 1
fi

echo "=== Security Audit for $IMAGE ==="
echo ""

echo "Checking for exposed secrets..."
docker history --no-trunc $IMAGE | grep -i "password\|secret\|key\|token" || echo "None found"
echo ""

echo "Checking for sensitive files..."
docker history --no-trunc $IMAGE | grep -E "\.pem|\.key|id_rsa|\.env" || echo "None found"
echo ""

echo "Checking for root operations..."
docker history --no-trunc $IMAGE | grep "USER" || echo "No USER directive found (running as root)"
echo ""

echo "Checking for package manager cleanup..."
docker history --no-trunc $IMAGE | grep -E "apt-get clean|rm -rf /var/lib/apt|yum clean" && \
    echo "✓ Cleanup commands found" || echo "⚠ No cleanup commands found"
echo ""

echo "Checking EXPOSE directives..."
docker history --no-trunc $IMAGE | grep "EXPOSE" || echo "No ports exposed"
```

## Advanced Use Cases

### JSON Processing with jq
```bash
# Extract all layer sizes as JSON
docker history --format json nginx:latest | jq '.Size'

# Find layers created in last 30 days
docker history --format json myapp:latest | \
    jq -r 'select(.CreatedAt | fromdateiso8601 > (now - 2592000))'

# Get total size of non-zero layers
docker history --format json myapp:latest | \
    jq -r 'select(.Size | contains("MB")) | .Size' | \
    sed 's/MB//' | awk '{sum+=$1} END {print sum " MB"}'

# Export as structured JSON
docker history --format json myapp:latest | jq -s '.' > history.json
```

### Automated Reporting
```bash
#!/bin/bash
# generate-report.sh - Generate comprehensive history report

IMAGE=$1
REPORT="report-$(date +%Y%m%d-%H%M%S).html"

cat > $REPORT <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>Image History Report - $IMAGE</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
        .warning { color: red; font-weight: bold; }
    </style>
</head>
<body>
    <h1>Image History Report</h1>
    <h2>$IMAGE</h2>
    <p>Generated: $(date)</p>
    
    <h3>Summary</h3>
    <ul>
        <li>Total Layers: $(docker history $IMAGE | tail -n +2 | wc -l)</li>
        <li>Image Size: $(docker images $IMAGE --format "{{.Size}}")</li>
        <li>Created: $(docker history $IMAGE --format "{{.CreatedAt}}" | head -1)</li>
    </ul>
    
    <h3>Layer Details</h3>
    <pre>
$(docker history $IMAGE)
    </pre>
    
    <h3>Large Layers (>10MB)</h3>
    <pre>
$(docker history $IMAGE | grep MB | awk '$5+0 > 10 {print $0}')
    </pre>
</body>
</html>
EOF

echo "Report generated: $REPORT"
```

### Continuous Monitoring
```bash
#!/bin/bash
# monitor-history.sh - Monitor image history changes

IMAGE=$1
BASELINE="baseline-$(echo $IMAGE | tr ':/' '-').txt"

if [ ! -f "$BASELINE" ]; then
    docker history --no-trunc $IMAGE > $BASELINE
    echo "Baseline created: $BASELINE"
    exit 0
fi

CURRENT=$(mktemp)
docker history --no-trunc $IMAGE > $CURRENT

if diff -q $BASELINE $CURRENT > /dev/null; then
    echo "No changes detected in image history"
else
    echo "Changes detected:"
    diff $BASELINE $CURRENT
    
    echo ""
    echo "Update baseline? (y/n)"
    read -r response
    if [ "$response" = "y" ]; then
        cp $CURRENT $BASELINE
        echo "Baseline updated"
    fi
fi

rm $CURRENT
```

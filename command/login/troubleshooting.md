# Troubleshooting

## Common Issues

### Authentication Failed
**Problem**: Cannot authenticate with registry
```bash
# Verify credentials are correct
docker login -u myusername

# Check if registry is accessible
curl -I https://registry.example.com/v2/

# Verify network connectivity
ping registry.example.com

# Try with explicit registry URL
docker login registry.example.com

# Check for typos in username
docker login -u myusername --password-stdin
```

### Permission Denied
**Problem**: Cannot write to Docker config directory
```bash
# Check Docker config permissions
ls -la ~/.docker/

# Create config directory if missing
mkdir -p ~/.docker
chmod 700 ~/.docker

# Fix permissions
sudo chown -R $USER:$USER ~/.docker

# Check Docker socket permissions
ls -la /var/run/docker.sock
```

### Credential Store Error
**Problem**: Credential helper fails or not found
```bash
# Check credential helper configuration
cat ~/.docker/config.json | jq '.credsStore'

# List available credential helpers
docker-credential-<helper> list

# Remove credential store configuration
jq 'del(.credsStore)' ~/.docker/config.json > ~/.docker/config.json.tmp
mv ~/.docker/config.json.tmp ~/.docker/config.json

# Login without credential helper
docker login
```

### Token Expired
**Problem**: Previously saved credentials no longer work
```bash
# Logout and login again
docker logout registry.example.com
docker login registry.example.com

# Clear all credentials
rm ~/.docker/config.json
docker login

# For AWS ECR, refresh token
aws ecr get-login-password --region us-east-1 | \
    docker login --username AWS --password-stdin <registry-url>
```

### Registry Not Responding
**Problem**: Cannot connect to registry server
```bash
# Check registry status
curl -v https://registry.example.com/v2/

# Verify DNS resolution
nslookup registry.example.com

# Check proxy settings
echo $HTTP_PROXY
echo $HTTPS_PROXY

# Test with curl
curl -u username:password https://registry.example.com/v2/_catalog

# Try different network
docker login --tls-verify=true registry.example.com
```

### Certificate Issues
**Problem**: SSL/TLS certificate verification fails
```bash
# Check certificate
openssl s_client -connect registry.example.com:443

# Add certificate to trusted store
sudo cp registry.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# Temporarily skip verification (not recommended)
docker login --tls-verify=false registry.example.com

# Use insecure registry (for testing only)
# Edit /etc/docker/daemon.json
{
  "insecure-registries": ["registry.example.com:5000"]
}
```

### Two-Factor Authentication
**Problem**: 2FA is enabled but login fails
```bash
# Use access token instead of password
docker login -u myusername

# Enter personal access token as password
# (not your account password)

# Generate token from registry web interface first
# Then use it for login
echo $ACCESS_TOKEN | docker login -u myusername --password-stdin
```

## Debugging Commands

### Verify Login Status
```bash
# Check stored credentials
cat ~/.docker/config.json

# List authenticated registries
cat ~/.docker/config.json | jq '.auths | keys'

# Check credential helper
docker-credential-pass list

# Verify authentication works
docker pull private/image:latest
```

### Test Registry Connection
```bash
# Test registry API
curl -v https://registry.example.com/v2/

# Check authentication endpoint
curl -v https://registry.example.com/v2/_catalog

# Test with credentials
curl -u username:password https://registry.example.com/v2/_catalog

# Check registry version
curl https://registry.example.com/v2/ | jq
```

### Inspect Configuration
```bash
# View Docker client config
cat ~/.docker/config.json | jq

# Check daemon configuration
cat /etc/docker/daemon.json

# View credential helpers
ls -la /usr/bin/docker-credential-*

# Check environment variables
env | grep DOCKER
```

### Network Diagnostics
```bash
# Test connectivity
nc -zv registry.example.com 443

# Check routing
traceroute registry.example.com

# Verify DNS
dig registry.example.com

# Test with wget
wget --spider https://registry.example.com/v2/
```

### Debug Mode
```bash
# Enable debug logging
docker --debug login registry.example.com

# Check Docker daemon logs
sudo journalctl -u docker.service -f

# View client logs
docker --log-level debug login

# Verbose curl test
curl -v -u username:password https://registry.example.com/v2/
```

## Exit Codes and Meanings

### Standard Exit Codes
- **0** - Successful login
- **1** - General error (wrong credentials, network issue)
- **125** - Docker daemon error
- **126** - Command not executable
- **127** - Command not found

### Common Error Messages
- **"Error response from daemon: Get https://...: unauthorized"** - Wrong credentials
- **"Error response from daemon: Get https://...: x509"** - Certificate issue
- **"Error saving credentials"** - Credential helper error
- **"Cannot connect to the Docker daemon"** - Docker not running

### Checking Exit Codes
```bash
# Login and check result
docker login registry.example.com
echo $?

# Capture error output
ERROR=$(docker login registry.example.com 2>&1)
if [ $? -ne 0 ]; then
    echo "Login failed: $ERROR"
fi
```

## Recovery Procedures

### Reset Docker Credentials
```bash
# Step 1: Logout from all registries
docker logout
docker logout registry.example.com

# Step 2: Remove config file
rm ~/.docker/config.json

# Step 3: Recreate config directory
mkdir -p ~/.docker
chmod 700 ~/.docker

# Step 4: Login again
docker login
```

### Fix Credential Helper Issues
```bash
# Step 1: Check current configuration
cat ~/.docker/config.json

# Step 2: Remove credential helper
jq 'del(.credsStore)' ~/.docker/config.json > /tmp/config.json
mv /tmp/config.json ~/.docker/config.json

# Step 3: Clear credentials
docker logout

# Step 4: Login without helper
docker login
```

### Handle AWS ECR Token Expiration
```bash
#!/bin/bash
# ecr-login.sh - Refresh ECR token

REGION="us-east-1"
ACCOUNT_ID="123456789012"
REGISTRY="$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com"

echo "Refreshing ECR login token..."

aws ecr get-login-password --region $REGION | \
    docker login --username AWS --password-stdin $REGISTRY

if [ $? -eq 0 ]; then
    echo "✓ Successfully logged in to ECR"
else
    echo "✗ Failed to login to ECR"
    exit 1
fi
```

### Automated Debugging Script
```bash
#!/bin/bash
# docker-login-debug.sh - Debug login issues

REGISTRY=${1:-docker.io}

echo "=== Docker Login Debug Report ==="
echo "Registry: $REGISTRY"
echo "Timestamp: $(date)"
echo

echo "=== Docker Version ==="
docker version
echo

echo "=== Current Configuration ==="
if [ -f ~/.docker/config.json ]; then
    cat ~/.docker/config.json | jq
else
    echo "No config file found"
fi
echo

echo "=== Credential Helpers ==="
ls -la /usr/bin/docker-credential-* 2>/dev/null || echo "No credential helpers found"
echo

echo "=== Registry Connectivity ==="
curl -I https://$REGISTRY/v2/ 2>&1
echo

echo "=== DNS Resolution ==="
nslookup $REGISTRY
echo

echo "=== Network Test ==="
nc -zv $REGISTRY 443 2>&1
echo

echo "=== Docker Daemon Status ==="
systemctl status docker 2>&1 | head -10
```

### Multi-Registry Login Check
```bash
#!/bin/bash
# check-logins.sh - Verify login status for multiple registries

REGISTRIES=(
    "docker.io"
    "gcr.io"
    "registry.example.com"
)

echo "=== Registry Login Status ==="
echo

for REGISTRY in "${REGISTRIES[@]}"; do
    echo "Testing $REGISTRY..."
    
    # Check if credentials exist
    if cat ~/.docker/config.json | jq -e ".auths.\"$REGISTRY\"" > /dev/null 2>&1; then
        echo "  ✓ Credentials found"
        
        # Test with a simple API call
        if curl -f -s -o /dev/null https://$REGISTRY/v2/; then
            echo "  ✓ Registry accessible"
        else
            echo "  ✗ Registry not accessible"
        fi
    else
        echo "  ✗ No credentials found"
    fi
    echo
done
```

## Best Practices for Troubleshooting

### Before Login
```bash
# 1. Verify Docker is running
docker version

# 2. Check network connectivity
ping registry.example.com

# 3. Verify credentials
# Don't use password in command line

# 4. Test registry API
curl https://registry.example.com/v2/
```

### When Login Fails
```bash
# 1. Check error message carefully
docker login 2>&1 | tee login-error.log

# 2. Verify username (no typos)
echo "Username: $USERNAME"

# 3. Try interactive login
docker login

# 4. Check Docker daemon logs
sudo journalctl -u docker.service -n 50
```

### For CI/CD Issues
```bash
# 1. Use environment variables
export DOCKER_USERNAME="myuser"
export DOCKER_PASSWORD="mypass"

# 2. Login via stdin
echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

# 3. Verify in CI logs (mask credentials)
docker login 2>&1 | grep -v password

# 4. Test locally first
# Replicate CI environment
```

### Regular Maintenance
```bash
# Periodically refresh credentials
docker logout registry.example.com
docker login registry.example.com

# Clean up old configs
rm ~/.docker/config.json.backup*

# Verify permissions
chmod 600 ~/.docker/config.json

# Update Docker
sudo apt-get update && sudo apt-get upgrade docker-ce
```

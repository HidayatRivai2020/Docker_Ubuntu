# Examples

## Basic Examples
```bash
# Login to Docker Hub (default)
docker login

# Login with username
docker login -u myusername

# Login to private registry
docker login registry.example.com

# Login with specific credentials
docker login -u myusername registry.example.com
```

## Secure Login Examples
```bash
# Login using stdin (recommended)
echo "mypassword" | docker login -u myusername --password-stdin

# Login from file
cat password.txt | docker login -u myusername --password-stdin

# Login using environment variable
echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
```

## Registry-Specific Examples
```bash
# Login to Docker Hub
docker login docker.io

# Login to Amazon ECR
docker login 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Login to Google Container Registry
docker login gcr.io

# Login to Azure Container Registry
docker login myregistry.azurecr.io

# Login to GitHub Container Registry
docker login ghcr.io
```

## Advanced Examples
```bash
# Login to multiple registries
docker login docker.io
docker login registry.company.com
docker login ghcr.io

# Login with long-lived token
echo $ACCESS_TOKEN | docker login -u oauth2accesstoken --password-stdin gcr.io

# Login to insecure registry (not recommended)
docker login --tls-verify=false insecure-registry.local
```

## Common Use Cases

### CI/CD Pipeline Login
```bash
# GitHub Actions
echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

# GitLab CI
echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY

# Jenkins
echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

# Azure DevOps
echo $AZURE_CR_PASSWORD | docker login -u $AZURE_CR_USERNAME --password-stdin myregistry.azurecr.io
```

### AWS ECR Authentication
```bash
# Get login password from AWS CLI
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Using AWS CLI v1 (deprecated)
$(aws ecr get-login --no-include-email --region us-east-1)

# With profile
aws ecr get-login-password --profile myprofile --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
```

### Google Cloud Registry
```bash
# Using gcloud as credential helper
gcloud auth configure-docker

# Direct login with access token
gcloud auth print-access-token | docker login -u oauth2accesstoken --password-stdin gcr.io

# Using service account key
cat key.json | docker login -u _json_key --password-stdin gcr.io
```

### Azure Container Registry
```bash
# Login with Azure CLI
az acr login --name myregistry

# Login with service principal
docker login myregistry.azurecr.io -u <service-principal-id> -p <service-principal-password>

# Using admin credentials
docker login myregistry.azurecr.io -u myregistry -p <admin-password>
```

## CI/CD Integration

### GitHub Actions
```yaml
name: Login Example

on: [push]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
      
      - name: Login to GitHub Container Registry
        run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
```

### GitLab CI
```yaml
docker-login:
  stage: build
  script:
    # Login to GitLab registry
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
    
    # Login to Docker Hub
    - echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USER --password-stdin
```

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    
    stages {
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }
    }
}
```

## Scripting Examples

### Login Script with Error Handling
```bash
#!/bin/bash
# docker-login.sh - Safe Docker login script

REGISTRY=${1:-docker.io}
USERNAME=${2}
PASSWORD=${3}

if [ -z "$USERNAME" ] || [ -z "$PASSWORD" ]; then
    echo "Usage: $0 [registry] <username> <password>"
    exit 1
fi

echo "Logging in to $REGISTRY..."
echo "$PASSWORD" | docker login "$REGISTRY" -u "$USERNAME" --password-stdin

if [ $? -eq 0 ]; then
    echo "✓ Successfully logged in to $REGISTRY"
else
    echo "✗ Failed to login to $REGISTRY"
    exit 1
fi
```

### Multi-Registry Login Script
```bash
#!/bin/bash
# multi-registry-login.sh - Login to multiple registries

# Docker Hub
echo "Logging in to Docker Hub..."
echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin

# AWS ECR
echo "Logging in to AWS ECR..."
aws ecr get-login-password --region us-east-1 | \
    docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Private Registry
echo "Logging in to private registry..."
echo $PRIVATE_REGISTRY_PASSWORD | docker login registry.company.com -u $PRIVATE_REGISTRY_USERNAME --password-stdin

echo "✓ All logins completed"
```

### Automated Token Refresh
```bash
#!/bin/bash
# refresh-token.sh - Refresh Docker registry token

TOKEN_FILE="$HOME/.docker/token"
REGISTRY="registry.example.com"

# Get new token (example using API)
NEW_TOKEN=$(curl -s -X POST https://auth.example.com/token \
    -d "username=$USERNAME" \
    -d "password=$PASSWORD" | jq -r .token)

# Login with new token
echo "$NEW_TOKEN" | docker login $REGISTRY -u token --password-stdin

# Save token
echo "$NEW_TOKEN" > "$TOKEN_FILE"
chmod 600 "$TOKEN_FILE"

echo "Token refreshed successfully"
```

## Security Considerations

### Secure Credential Storage
```bash
# Use credential helper
docker login --help | grep credential

# Configure credential store
mkdir -p ~/.docker
cat > ~/.docker/config.json <<EOF
{
  "credsStore": "pass"
}
EOF

# Login with credential helper
docker login
```

### Temporary Access
```bash
# Login for single session
docker login

# Perform operations
docker pull myimage
docker push myimage

# Logout immediately
docker logout
```

### Environment-Specific Login
```bash
# Development
docker login dev-registry.company.com

# Staging
docker login staging-registry.company.com

# Production (use service accounts)
echo $PROD_TOKEN | docker login prod-registry.company.com -u serviceaccount --password-stdin
```

## Troubleshooting Examples

### Verify Login Status
```bash
# Check if logged in
cat ~/.docker/config.json | jq '.auths'

# Verify authentication works
docker pull private-image:latest

# Test with a push
docker tag test:latest registry.example.com/test:latest
docker push registry.example.com/test:latest
```

### Debug Login Issues
```bash
# Enable debug logging
docker --debug login registry.example.com

# Check credential helper
docker-credential-pass list

# Verify registry connectivity
curl -I https://registry.example.com/v2/

# Test authentication
curl -u username:password https://registry.example.com/v2/_catalog
```

### Re-login After Errors
```bash
# Logout first
docker logout registry.example.com

# Clear credentials
rm -f ~/.docker/config.json

# Login again
docker login registry.example.com
```

# How to Deploy Docker Image

## Deployment Workflow

```
Build Image → Tag Image → Push to Registry → Pull on Server → Run Container
```

## Prerequisites

### 1. Docker Image Ready
```bash
# Verify your image exists
docker images

# Test the image locally
docker run --rm -p 8080:8080 myapp:latest
```

### 2. Container Registry Access
- Docker Hub
- Amazon ECR
- Google Container Registry (GCR)
- Azure Container Registry (ACR)
- Private Registry
- GitHub Container Registry (GHCR)

## Deploy to Docker Hub

### Step 1: Create Docker Hub Account
Sign up at [https://hub.docker.com](https://hub.docker.com)

### Step 2: Login to Docker Hub
```bash
# Login with credentials
docker login

# Login with access token (recommended)
docker login -u username -p access_token
```

### Step 3: Tag Your Image
```bash
# Tag with Docker Hub username
docker tag myapp:latest username/myapp:latest
docker tag myapp:latest username/myapp:1.0.0

# Tag with specific version
docker tag myapp:latest username/myapp:v1.0.0
```

### Step 4: Push to Docker Hub
```bash
# Push latest tag
docker push username/myapp:latest

# Push specific version
docker push username/myapp:1.0.0

# Push all tags
docker push username/myapp --all-tags
```

### Step 5: Deploy on Target Server
```bash
# Pull the image
docker pull username/myapp:latest

# Run the container
docker run -d \
  --name myapp \
  -p 80:8080 \
  --restart unless-stopped \
  username/myapp:latest
```

## Deploy to Amazon ECR

### Step 1: Install AWS CLI
```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS CLI
aws configure
```

### Step 2: Create ECR Repository
```bash
# Create repository
aws ecr create-repository \
  --repository-name myapp \
  --region us-east-1

# Get repository URI
aws ecr describe-repositories \
  --repository-names myapp \
  --region us-east-1
```

### Step 3: Login to ECR
```bash
# Get login password and authenticate
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-east-1.amazonaws.com
```

### Step 4: Tag and Push Image
```bash
# Tag image
docker tag myapp:latest \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Push image
docker push \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

### Step 5: Deploy on EC2 or ECS
```bash
# On EC2 instance
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-east-1.amazonaws.com

docker pull 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

docker run -d \
  --name myapp \
  -p 80:8080 \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

## Deploy to Google Cloud (GCR)

### Step 1: Install gcloud CLI
```bash
# Install gcloud
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# Initialize gcloud
gcloud init
```

### Step 2: Configure Docker for GCR
```bash
# Configure Docker authentication
gcloud auth configure-docker

# Or for specific region
gcloud auth configure-docker us-central1-docker.pkg.dev
```

### Step 3: Tag and Push Image
```bash
# Tag image
docker tag myapp:latest \
  gcr.io/project-id/myapp:latest

# Push image
docker push gcr.io/project-id/myapp:latest
```

### Step 4: Deploy on GCE or Cloud Run
```bash
# Deploy to Cloud Run
gcloud run deploy myapp \
  --image gcr.io/project-id/myapp:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated

# On Compute Engine VM
docker pull gcr.io/project-id/myapp:latest
docker run -d --name myapp -p 80:8080 gcr.io/project-id/myapp:latest
```

## Deploy to Azure Container Registry (ACR)

### Step 1: Install Azure CLI
```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login
```

### Step 2: Create ACR
```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# Create container registry
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Basic
```

### Step 3: Login to ACR
```bash
# Login to registry
az acr login --name myregistry
```

### Step 4: Tag and Push Image
```bash
# Get login server
az acr show --name myregistry --query loginServer --output table

# Tag image
docker tag myapp:latest myregistry.azurecr.io/myapp:latest

# Push image
docker push myregistry.azurecr.io/myapp:latest
```

### Step 5: Deploy on Azure VM or Container Instances
```bash
# Deploy to Azure Container Instances
az container create \
  --resource-group myResourceGroup \
  --name myapp \
  --image myregistry.azurecr.io/myapp:latest \
  --cpu 1 \
  --memory 1 \
  --registry-login-server myregistry.azurecr.io \
  --registry-username username \
  --registry-password password \
  --ip-address Public \
  --ports 8080
```

## Deploy to Private Registry

### Step 1: Set Up Private Registry
```bash
# Run registry container
docker run -d \
  -p 5000:5000 \
  --name registry \
  --restart always \
  -v /mnt/registry:/var/lib/registry \
  registry:2
```

### Step 2: Tag and Push Image
```bash
# Tag image
docker tag myapp:latest localhost:5000/myapp:latest

# Push image
docker push localhost:5000/myapp:latest
```

### Step 3: Deploy on Remote Server
```bash
# On remote server
docker pull registry.company.com:5000/myapp:latest

docker run -d \
  --name myapp \
  -p 80:8080 \
  registry.company.com:5000/myapp:latest
```

## Deploy to GitHub Container Registry

### Step 1: Create Personal Access Token
- Go to GitHub Settings → Developer settings → Personal access tokens
- Generate new token with `write:packages` and `read:packages` scopes

### Step 2: Login to GHCR
```bash
# Login to GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u username --password-stdin
```

### Step 3: Tag and Push Image
```bash
# Tag image
docker tag myapp:latest ghcr.io/username/myapp:latest

# Push image
docker push ghcr.io/username/myapp:latest
```

### Step 4: Deploy on Server
```bash
# Login on target server
echo $GITHUB_TOKEN | docker login ghcr.io -u username --password-stdin

# Pull and run
docker pull ghcr.io/username/myapp:latest
docker run -d --name myapp -p 80:8080 ghcr.io/username/myapp:latest
```

## Deploy with Docker Compose

### Step 1: Create docker-compose.yml
```yaml
version: '3.8'

services:
  app:
    image: username/myapp:latest
    ports:
      - "80:8080"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/mydb
    depends_on:
      - db
    restart: unless-stopped
    networks:
      - app-network

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - db-data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

### Step 2: Deploy Application
```bash
# Pull latest images
docker-compose pull

# Start services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f

# Update service
docker-compose pull app
docker-compose up -d app
```

## Deploy to Kubernetes

### Step 1: Create Deployment YAML
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: username/myapp:latest
        ports:
        - containerPort: 8080
        env:
        - name: NODE_ENV
          value: "production"
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: myapp
```

### Step 2: Deploy to Kubernetes
```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Check deployment
kubectl get deployments
kubectl get pods
kubectl get services

# Update image
kubectl set image deployment/myapp myapp=username/myapp:v2

# Scale deployment
kubectl scale deployment/myapp --replicas=5
```

## Deployment Strategies

### 1. Rolling Update
```bash
# Update container with zero downtime
docker service update \
  --image username/myapp:v2 \
  myapp
```

### 2. Blue-Green Deployment
```bash
# Start new version (green)
docker run -d --name myapp-green -p 8081:8080 username/myapp:v2

# Test green deployment
curl http://localhost:8081/health

# Switch traffic (update load balancer or reverse proxy)
# Stop old version (blue)
docker stop myapp-blue
docker rm myapp-blue
```

### 3. Canary Deployment
```bash
# Deploy to small subset of servers first
# Run new version alongside old version
docker run -d --name myapp-canary -p 8082:8080 username/myapp:v2

# Monitor metrics
# Gradually increase traffic to canary
# Full rollout if successful
```

## Environment-Specific Deployment

### Development
```bash
docker run -d \
  --name myapp-dev \
  -p 8080:8080 \
  -e NODE_ENV=development \
  -v $(pwd):/app \
  username/myapp:latest
```

### Staging
```bash
docker run -d \
  --name myapp-staging \
  -p 8080:8080 \
  -e NODE_ENV=staging \
  -e DATABASE_URL=$STAGING_DB_URL \
  username/myapp:latest
```

### Production
```bash
docker run -d \
  --name myapp-prod \
  -p 80:8080 \
  -e NODE_ENV=production \
  -e DATABASE_URL=$PROD_DB_URL \
  --restart unless-stopped \
  --memory 2g \
  --cpus 2 \
  username/myapp:latest
```

## Automated Deployment with CI/CD

### GitHub Actions Example
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: username/myapp:latest,username/myapp:${{ github.sha }}
      
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker pull username/myapp:latest
            docker stop myapp || true
            docker rm myapp || true
            docker run -d --name myapp -p 80:8080 username/myapp:latest
```

## Monitoring Deployed Containers

### Check Container Status
```bash
# List running containers
docker ps

# Check container logs
docker logs -f myapp

# Monitor resource usage
docker stats myapp

# Inspect container
docker inspect myapp
```

### Health Checks
```bash
# Check application health endpoint
curl http://localhost:8080/health

# Check container health status
docker inspect --format='{{.State.Health.Status}}' myapp
```

## Rollback Deployment

### Using Docker
```bash
# Stop current version
docker stop myapp
docker rm myapp

# Run previous version
docker run -d --name myapp -p 80:8080 username/myapp:v1
```

### Using Docker Compose
```bash
# Specify previous version in docker-compose.yml
docker-compose down
docker-compose up -d
```

### Using Kubernetes
```bash
# Rollback to previous version
kubectl rollout undo deployment/myapp

# Rollback to specific revision
kubectl rollout undo deployment/myapp --to-revision=2
```

## Security Best Practices

### 1. Use Image Scanning
```bash
# Scan image for vulnerabilities
docker scan username/myapp:latest

# Using Trivy
trivy image username/myapp:latest
```

### 2. Use Secrets Management
```bash
# Use Docker secrets (Swarm)
echo "mysecretpassword" | docker secret create db_password -

# Use environment variables from file
docker run -d --env-file .env username/myapp:latest

# Use Kubernetes secrets
kubectl create secret generic app-secrets \
  --from-literal=db-password=secret
```

### 3. Run as Non-Root User
```dockerfile
# In Dockerfile
USER node
```

### 4. Use Read-Only File System
```bash
docker run -d \
  --read-only \
  --tmpfs /tmp \
  username/myapp:latest
```

## Troubleshooting Deployment

### Image Pull Failures
```bash
# Check registry authentication
docker login

# Pull image manually
docker pull username/myapp:latest

# Check image exists
docker images | grep myapp
```

### Container Won't Start
```bash
# Check logs
docker logs myapp

# Run interactively
docker run -it --rm username/myapp:latest /bin/sh

# Check port conflicts
netstat -tulpn | grep 8080
```

### Network Issues
```bash
# Check container network
docker network inspect bridge

# Test connectivity
docker exec myapp ping google.com

# Check exposed ports
docker port myapp
```

## Best Practices

1. **Use specific image tags**: Avoid `latest` in production
2. **Implement health checks**: Monitor container health
3. **Set resource limits**: Prevent resource exhaustion
4. **Use secrets management**: Don't hardcode credentials
5. **Enable logging**: Configure proper log drivers
6. **Plan rollback strategy**: Be prepared to revert
7. **Test before production**: Deploy to staging first
8. **Monitor metrics**: Track performance and errors
9. **Automate deployments**: Use CI/CD pipelines
10. **Document deployment process**: Maintain runbooks

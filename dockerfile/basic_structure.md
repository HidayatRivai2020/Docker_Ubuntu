# Basic Structure

## Essential Components
```dockerfile
# 1. Base Image (Required)
FROM ubuntu:22.04

# 2. Metadata (Optional but recommended)
LABEL maintainer="your-email@example.com"
LABEL version="1.0"
LABEL description="Your application description"

# 3. Environment Setup
ENV APP_HOME=/app
ENV NODE_ENV=production

# 4. Working Directory
WORKDIR /app

# 5. Dependencies Installation
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*

# 6. Copy Application Files
COPY . /app

# 7. Expose Ports
EXPOSE 8080

# 8. Define Startup Command
CMD ["./start.sh"]
```

## Common Patterns

### 1. Simple Web Application
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Set user
USER node

# Start application
CMD ["node", "server.js"]
```

### 2. Python Application
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

CMD ["python", "main.py"]
```

### 3. Multi-Stage Build
```dockerfile
# Build stage
FROM node:18 AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

USER node

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

### 4. Go Application with Minimal Image
```dockerfile
# Build stage
FROM golang:1.21 AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Production stage
FROM alpine:3.18

RUN apk --no-cache add ca-certificates

WORKDIR /app

COPY --from=builder /app/main .

EXPOSE 8080

USER nobody

CMD ["./main"]
```

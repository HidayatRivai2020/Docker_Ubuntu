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
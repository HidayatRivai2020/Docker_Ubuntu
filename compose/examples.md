# Examples

## Web Application Stack

```yaml
version: '3.8'

services:
  # Database
  postgres:
    image: postgres:15-alpine
    container_name: myapp_db
    restart: unless-stopped
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-db:/docker-entrypoint-initdb.d
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend API
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    container_name: myapp_api
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      DATABASE_URL: postgresql://admin:${DB_PASSWORD}@postgres:5432/myapp
      JWT_SECRET: ${JWT_SECRET}
      PORT: 5000
    volumes:
      - ./api:/app
      - /app/node_modules
    networks:
      - backend
      - frontend
    expose:
      - "5000"

  # Frontend
  web:
    build:
      context: ./web
      dockerfile: Dockerfile
    container_name: myapp_web
    restart: unless-stopped
    depends_on:
      - api
    environment:
      REACT_APP_API_URL: http://localhost:5000
    ports:
      - "3000:3000"
    volumes:
      - ./web:/app
      - /app/node_modules
    networks:
      - frontend

  # Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: myapp_nginx
    restart: unless-stopped
    depends_on:
      - web
      - api
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    networks:
      - frontend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  postgres_data:
```

## Microservices Architecture

```yaml
version: '3.8'

services:
  # Message Broker
  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - microservices

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: redis
    command: redis-server --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - microservices

  # User Service
  user-service:
    build: ./services/user
    container_name: user_service
    depends_on:
      - rabbitmq
      - redis
    environment:
      SERVICE_NAME: user-service
      RABBITMQ_URL: amqp://admin:${RABBITMQ_PASSWORD}@rabbitmq:5672
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
    networks:
      - microservices

  # Order Service
  order-service:
    build: ./services/order
    container_name: order_service
    depends_on:
      - rabbitmq
      - redis
    environment:
      SERVICE_NAME: order-service
      RABBITMQ_URL: amqp://admin:${RABBITMQ_PASSWORD}@rabbitmq:5672
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
    networks:
      - microservices

  # API Gateway
  gateway:
    build: ./gateway
    container_name: api_gateway
    depends_on:
      - user-service
      - order-service
    ports:
      - "8000:8000"
    environment:
      USER_SERVICE_URL: http://user-service:3000
      ORDER_SERVICE_URL: http://order-service:3000
    networks:
      - microservices

networks:
  microservices:
    driver: bridge

volumes:
  rabbitmq_data:
  redis_data:
```

## Development Environment

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: development
    container_name: dev_app
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      DEBUG: app:*
    command: npm run dev
    stdin_open: true
    tty: true

  db:
    image: postgres:15-alpine
    container_name: dev_db
    environment:
      POSTGRES_DB: dev_db
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data

volumes:
  postgres_dev_data:
```

### Health Checks

```yaml
services:
  web:
    image: nginx:alpine
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Resource Limits

```yaml
services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### Profiles

```yaml
services:
  web:
    image: nginx:alpine
    profiles:
      - production
  
  debug-tools:
    image: nicolaka/netshoot
    profiles:
      - debug
```

```bash
# Start only production profile
docker compose --profile production up

# Start with debug profile
docker compose --profile debug up
```

### Override Files

**docker-compose.override.yml** (automatically loaded):
```yaml
services:
  app:
    volumes:
      - ./src:/app/src  # Mount source for hot reload
    environment:
      DEBUG: "true"
```

```bash
# Use specific override file
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```
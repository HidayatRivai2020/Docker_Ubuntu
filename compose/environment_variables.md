# Environment Variables

## .env File

Create a `.env` file in the same directory as `docker-compose.yml`:

```env
# Database
DB_PASSWORD=secret_password
POSTGRES_VERSION=15

# Application
NODE_ENV=production
API_PORT=5000

# Services
REDIS_PASSWORD=redis_secret
RABBITMQ_PASSWORD=rabbitmq_secret
```

## Using in docker-compose.yml

```yaml
services:
  app:
    image: myapp:${APP_VERSION:-latest}
    ports:
      - "${API_PORT:-5000}:5000"
    environment:
      NODE_ENV: ${NODE_ENV}
      DATABASE_URL: postgresql://user:${DB_PASSWORD}@db:5432/mydb
```

## Multiple Environment Files

```bash
# Specify custom env file
docker compose --env-file .env.production up

# Use multiple compose files
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```
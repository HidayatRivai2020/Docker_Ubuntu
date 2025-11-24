# Troubleshooting

## View Service Status

```bash
# Check all services
docker compose ps

# View detailed status
docker compose ps -a

# Check service logs
docker compose logs --tail=50 service_name
```

## Network Issues

```bash
# List networks
docker network ls

# Inspect compose network
docker network inspect folder_default

# Test connectivity between services
docker compose exec web ping db
```

## Volume Issues

```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect folder_postgres_data

# Remove all volumes
docker compose down -v
```

## Rebuild Issues

```bash
# Force rebuild without cache
docker compose build --no-cache

# Remove everything and start fresh
docker compose down -v --rmi all
docker compose up -d --build
```

## Permission Issues

```bash
# Check file ownership in container
docker compose exec web ls -la /app

# Fix permissions (if needed)
docker compose exec web chown -R www-data:www-data /app
```

## Debug Container Startup

```bash
# View container logs
docker compose logs service_name

# Check container processes
docker compose top service_name

# Inspect service configuration
docker compose config

# Validate compose file
docker compose config --quiet
```
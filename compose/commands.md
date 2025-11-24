# Common Commands

## Starting and Stopping

```bash
# Start all services
docker compose up -d

# Start specific services
docker compose up -d web db

# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v

# Stop and remove images
docker compose down --rmi all
```

## Viewing Status

```bash
# List running services
docker compose ps

# View logs
docker compose logs

# Follow logs for specific service
docker compose logs -f web

# View last 100 lines
docker compose logs --tail=100
```

## Managing Services

```bash
# Restart services
docker compose restart

# Restart specific service
docker compose restart web

# Stop services
docker compose stop

# Start stopped services
docker compose start

# Remove stopped containers
docker compose rm
```

## Building and Updating

```bash
# Build or rebuild services
docker compose build

# Build without cache
docker compose build --no-cache

# Pull latest images
docker compose pull

# Recreate containers
docker compose up -d --force-recreate
```

## Executing Commands

```bash
# Execute command in running container
docker compose exec web sh

# Run one-off command
docker compose run web python manage.py migrate

# Run without dependencies
docker compose run --no-deps web npm test
```

## Scaling Services

```bash
# Scale service to 3 instances
docker compose up -d --scale web=3

# Scale multiple services
docker compose up -d --scale web=3 --scale worker=5
```


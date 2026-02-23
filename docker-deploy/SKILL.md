---
name: docker-deploy
version: 1.0.0
description: Deploy, update, and manage Docker Compose services in production.
author: ZeptoClaw
license: MIT
tags:
  - docker
  - devops
  - deployment
metadata: {"zeptoclaw":{"emoji":"🐳","requires":{"anyBins":["docker"]}}}
---

# Docker Deploy Skill

Deploy and manage containerized services with Docker and Docker Compose.

## Deploy / Update a Service

```bash
# Pull latest image and recreate changed containers only
docker compose pull && docker compose up -d

# Force recreate a single service
docker compose up -d --force-recreate --no-deps api

# Zero-downtime rolling update (requires a load balancer)
docker compose up -d --scale api=2 --no-recreate
docker compose up -d --scale api=1
```

## Check Status

```bash
# All services
docker compose ps

# Live logs (follow)
docker compose logs -f --tail=100 api

# Resource usage
docker stats --no-stream

# Inspect a container
docker inspect $(docker compose ps -q api)
```

## Common Operations

```bash
# Run a one-off command in a running service
docker compose exec api bash
docker compose exec db psql -U postgres mydb

# Run migrations (one-off container)
docker compose run --rm api python manage.py migrate

# Copy a file out of a container
docker cp $(docker compose ps -q api):/app/logs/error.log ./error.log

# Hard restart (stop → rm → start)
docker compose down && docker compose up -d
```

## Cleanup

```bash
# Remove stopped containers and unused images (safe)
docker system prune -f

# Also remove unused volumes (⚠️ check first)
docker volume ls
docker system prune -f --volumes

# Remove a specific image
docker rmi myapp:old-tag
```

## Deployment Workflow (Production)

```bash
# 1. Pull code changes on server
git pull origin main

# 2. Build new image (if building locally)
docker compose build --no-cache api

# 3. Apply without downtime
docker compose up -d --no-deps api

# 4. Verify it started
docker compose ps api
docker compose logs --tail=50 api

# 5. Rollback if bad
docker compose up -d --no-deps api --image myapp:previous-tag
```

## Tips

- Always use `--no-deps` when restarting a single service — avoids restarting dependencies
- `docker compose pull` before `up -d` ensures you get the latest image tag
- Health checks in `docker-compose.yml` prevent traffic before the app is ready
- Keep `.env` on the server — never deploy it with rsync/git (it contains secrets)

# Accounting production stack

This Compose file deploys the Accounting app from GHCR, together with Nginx,
PostgreSQL, Redis, a queue worker, and Laravel scheduler.

1. Copy `.env.production.example` to `.env.production` and replace every
   `CHANGE_ME` value. Generate `APP_KEY` with
   `php artisan key:generate --show` from `apps/accounting`.
2. Authenticate the server to GHCR if the package remains private:
   `docker login ghcr.io -u thisnugroho`.
3. Pull and start the stack:
   `docker compose --env-file .env.production -f docker/docker-compose.production.yml pull`
   then `docker compose --env-file .env.production -f docker/docker-compose.production.yml up -d`.
4. Run the migration once per release:
   `docker compose --env-file .env.production -f docker/docker-compose.production.yml --profile tools run --rm migrate`.

PostgreSQL and Redis are intentionally private to the Compose network. Only
Nginx exposes a host port.

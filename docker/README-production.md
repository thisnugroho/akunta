# Accounting production stack

This Compose file deploys one Accounting image from GHCR. The image contains
Nginx and PHP-FPM; the same image also runs the queue worker and Laravel
scheduler with command overrides. PostgreSQL and Redis remain separate.

1. In Dokploy, paste the variables from `.env.example` into the Compose
   service's **Environment** tab. Dokploy writes them to its local `.env`
   file automatically. Replace every `CHANGE_ME` value. Generate `APP_KEY` with
   `php artisan key:generate --show` from `apps/accounting`.
2. Authenticate the server to GHCR if the package remains private:
   `docker login ghcr.io -u thisnugroho`.
3. For a manual Docker Compose deployment, copy `.env.example` to
   `docker/.env`, then pull and start the stack:
   `docker compose -f docker/docker-compose.production.yml pull`
   then `docker compose -f docker/docker-compose.production.yml up -d`.
4. Run the migration once per release:
   `docker compose -f docker/docker-compose.production.yml --profile tools run --rm migrate`.

PostgreSQL and Redis are intentionally private to the Compose network. Only
Nginx exposes a host port.

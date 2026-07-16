# Accounting production stack

This Compose file deploys one Accounting image from GHCR. The image contains
Nginx and PHP-FPM; the same image also runs the queue worker and Laravel
scheduler with command overrides. PostgreSQL and Redis remain separate.

1. In Dokploy, paste the variables from `.env.example` into the Compose
   service's **Environment** tab. Dokploy writes them to its local `.env`
   file automatically. Replace every `CHANGE_ME` value. Generate `APP_KEY` with
   `php artisan key:generate --show` from `apps/accounting`.
2. Add the application domain in Dokploy's **Domains** tab with **Container
   Port** `80`. Do not add a host port mapping: Dokploy's Traefik owns ports
   `80` and `443` and routes traffic internally to the accounting container.
3. Authenticate the server to GHCR if the package remains private:
   `docker login ghcr.io -u thisnugroho`.
4. For a manual Docker Compose deployment, copy `.env.example` to
   `docker/.env`, then pull and start the stack:
   `docker compose -f docker/docker-compose.production.yml pull`
   then `docker compose -f docker/docker-compose.production.yml up -d`.
5. Database migrations run automatically on each deployment. The web, queue,
   and scheduler services wait until `php artisan migrate --force` completes
   successfully.

PostgreSQL and Redis are intentionally private to the Compose network. Nginx
is reachable only through Dokploy's Traefik routing. The accounting service is
attached to Dokploy's external `dokploy-network` so Traefik can reach port 80.

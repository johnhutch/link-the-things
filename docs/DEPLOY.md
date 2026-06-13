# Deploy — Quartets

Self-hosted on a **Synology DS918+** via DSM **Container Manager**.

We use **GitHub Actions** to automatically build the Docker image and push it to the GitHub Container Registry (GHCR) whenever code is pushed to `main`. On the Synology NAS, a **Watchtower** container automatically detects the new image, pulls it, and restarts the app with zero clicks.

A Cloudflare Tunnel securely exposes the app to the internet without opening any ports on the router.

## Initial Setup on Synology

1. **Install Container Manager** from the Synology Package Center.
2. **Make the project dir**, e.g., `/volume1/docker/quartets`, and put the `.env` file in it (copy `.env.example`, fill in real values — `RAILS_MASTER_KEY` is the verbatim contents of `config/master.key`, and include your `TUNNEL_TOKEN`).
3. **Open Container Manager**:
   - Go to the **Project** tab.
   - Click **Create**.
   - Name it `quartets`, point the Path to your project directory.
   - For Source, choose **Create docker-compose.yml** and upload or paste the contents of `docker-compose.yml` from this repository.
   - Click **Next** and complete the wizard. The NAS will pull the Postgres, Web, Cloudflared, and Watchtower images and start them up.

## Deploying Updates

Just `git push` to the `main` branch. 

GitHub Actions will build the new image and push it to GHCR. Within 5 minutes, Watchtower on your NAS will notice the new image, gracefully shut down the old `web` container, and start a new one.

*(Note: If your GitHub repository is private, you will need to generate a GitHub Personal Access Token (classic) with `read:packages` permission and add it to your `.env` file as `GITHUB_TOKEN` along with your username as `GITHUB_ACTOR`, so Watchtower has permission to pull the image.)*

## Expose it (HTTPS via Cloudflare)

The `docker-compose.yml` includes a `cloudflared` service. 

- Create a Zero Trust tunnel in the Cloudflare dashboard.
- Set its public hostname to point to `http://web:80`.
- Drop the provided token into your `.env` file as `TUNNEL_TOKEN`.

Traffic will route securely to your NAS without needing Synology's reverse proxy or opening ports.

## Backups

- **Database:** `docker compose exec db pg_dump -U quartets quartets_production > backup.sql` (cron it via DSM Task Scheduler).
- The `pgdata` volume also gets swept up by Synology's Hyper Backup if you include the Docker volumes path.

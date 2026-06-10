# Deploy — Quartets

Self-hosted on a **Synology DS918+** via DSM **Container Manager**. No GitHub
CI/CD, no registry: you build the image on your Mac (fast) and **ship it straight
to the NAS over SSH** with `bin/deploy`. The Celeron never builds anything. Dev
stays native (chruby + Postgres on the Mac); only production is containerized.
See ADR-0002 (containerized prod) and ADR-0004 (build-and-ship). GitHub is just
the archive remote — nothing deploys from it.

```
bin/deploy ─▶ docker build (on the Mac, linux/amd64)
           ─▶ docker save | ssh nas docker load
           ─▶ ssh nas: docker compose up -d
```

## One-time setup on the Synology

1. **Enable SSH** — Control Panel → Terminal & SNMP → Enable SSH service. Add
   your Mac's public key to the NAS account's `~/.ssh/authorized_keys` so
   `bin/deploy` runs without a password prompt.
2. **Install Container Manager** from Package Center (gives `docker` +
   `docker compose`, usually under `/usr/local/bin`).
3. **Make the project dir**, e.g. `/volume1/docker/quartets`, and put two
   files in it:
   - `docker-compose.yml` (copy from this repo)
   - `.env` (copy `.env.example`, fill in real values — `RAILS_MASTER_KEY` is the
     verbatim contents of `config/master.key`)

## Deploying

From the repo on your Mac:

```bash
NAS_SSH=you@your-nas.local bin/deploy
# optional: NAS_DIR=/volume1/docker/quartets (this is the default)
```

It builds for `linux/amd64` (the DS918+ is Intel — important on Apple Silicon),
streams the image to the NAS, and restarts the stack. The web container's
entrypoint runs `db:prepare`, which creates the schema and seeds the superuser
from `ADMIN_EMAIL` / `ADMIN_PASSWORD` on first boot, and migrates thereafter.

> If `docker compose` isn't on the SSH `PATH`, prefix the remote commands with
> `export PATH=$PATH:/usr/local/bin` (or symlink it).

## Expose it (HTTPS)

DSM handles the annoying parts natively:

- **DDNS** — Control Panel → External Access → DDNS (`you.synology.me` is free).
- **Reverse proxy** — Control Panel → Login Portal → Advanced → Reverse Proxy:
  source `https://you.synology.me:443` → destination `http://localhost:<WEB_PORT>`
  (default 3000). Enable **WebSocket** in the custom headers (for Turbo/cable),
  and make sure `X-Forwarded-Proto` is forwarded.
- **Cert** — Control Panel → Security → Certificate → add a Let's Encrypt cert
  for the DDNS hostname and assign it to the reverse-proxy entry.

Rails runs with `force_ssl` + `assume_ssl`, so it trusts the proxy's TLS
termination and won't redirect-loop.

## Backups

- **Database:** `docker compose exec db pg_dump -U quartets
  quartets_production > backup.sql` (cron it via DSM Task Scheduler).
- The `pgdata` volume also gets swept up by Synology's Hyper Backup if you
  include the Docker volumes path.

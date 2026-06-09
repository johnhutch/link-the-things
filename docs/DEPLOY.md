# Deploy — Link the Things

Self-hosted on a **Synology DS918+** via DSM **Container Manager**. Dev stays
native (chruby + Postgres on the Mac); only production is containerized. CI
builds the image on GitHub's runners and the NAS just pulls it — the Celeron
never compiles anything. Push to `main` ships it. See ADR-0002 for the why.

```
push to main ─▶ GitHub Action ─▶ build image ─▶ push to GHCR
                                                     │
                          SSH to NAS ◀───────────────┘
                          docker compose pull && up -d
```

## One-time setup on the Synology

1. **Enable SSH** — DSM → Control Panel → Terminal & SNMP → Enable SSH service.
2. **Install Container Manager** from Package Center (gives you `docker` +
   `docker compose`, usually under `/usr/local/bin`).
3. **Make a project dir**, e.g. `/volume1/docker/link-the-things`, and put two
   files in it:
   - `docker-compose.yml` (copy from this repo)
   - `.env` (copy `.env.example`, fill in real values — `RAILS_MASTER_KEY` is the
     verbatim contents of `config/master.key`)
4. **Authenticate to GHCR** so the NAS can pull the image. Either:
   - make the GHCR package public (GitHub → repo → Packages → package settings),
     then no login is needed; or
   - `docker login ghcr.io -u johnhutch` with a Personal Access Token that has
     `read:packages`.
5. **First boot:** from the project dir, `docker compose up -d`. The web
   container's entrypoint runs `db:prepare`, which creates the schema and seeds
   the superuser from `ADMIN_EMAIL` / `ADMIN_PASSWORD`.

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

## GitHub repo secrets (for the deploy Action)

Set these under repo → Settings → Secrets and variables → Actions:

| Secret | Value |
|--------|-------|
| `SYNOLOGY_HOST` | NAS hostname / public IP |
| `SYNOLOGY_USER` | SSH user (a DSM admin account) |
| `SYNOLOGY_SSH_KEY` | private key whose public half is authorized on the NAS |
| `SYNOLOGY_SSH_PORT` | SSH port (DSM defaults to 22; change it if you have) |
| `SYNOLOGY_APP_DIR` | e.g. `/volume1/docker/link-the-things` |

`GITHUB_TOKEN` (for the GHCR push) is provided automatically.

## Routine deploys

Just `git push origin main`. The Action builds, pushes, and rolls the container.
If you ever change `docker-compose.yml` or `.env`, update the copy on the NAS by
hand (those don't ride along with the image).

## Backups

- **Database:** `docker compose exec db pg_dump -U link_the_things
  link_the_things_production > backup.sql` (cron it via DSM Task Scheduler).
- The `pgdata` volume also gets swept up by Synology's Hyper Backup if you
  include the Docker volumes path.

# Home server runbook

How the Mac mini in the hallway cupboard is set up, how to deploy to it, and what to do when something breaks at 2 a.m.

## Hardware

| Part | Model | Notes |
|------|-------|-------|
| Machine | Mac mini M4, 32 GB | Wired to the router, static lease `192.168.1.40` |
| Storage | 2 TB external SSD | Mounted at `/Volumes/data`, Time Machine excluded |
| Power | APC Back-UPS 700 | Gives about 25 minutes on battery |

## Network

### Router and DNS

The router hands out a fixed address to the mini. Public DNS points at a Cloudflare tunnel, so no ports are open on the router.

### Cloudflare tunnel

#### Installing cloudflared

```bash
brew install cloudflared
cloudflared tunnel login
cloudflared tunnel create home
```

#### Routing hostnames

Each app gets its own hostname in `~/.cloudflared/config.yml`:

```yaml
ingress:
  - hostname: app.example.com
    service: http://localhost:8080
  - service: http_status:404
```

#### Running as a service

```bash
sudo cloudflared service install
```

### Local access

Use `ssh mini` from the laptop. The SSH config entry uses the static lease, so it works even when the tunnel is down.

## Deploying

### First deploy

1. Clone the app into `~/apps/<name>`.
2. Copy `.env.example` to `.env` and fill in secrets.
3. Run `docker compose up -d --build`.

### Updates

```bash
ssh mini 'cd ~/apps/app && git pull && docker compose up -d --build'
```

### Rolling back

Check out the previous tag and rebuild. Images are kept for seven days, so `docker compose up -d` with the old tag is usually instant.

## Backups

### What gets backed up

- Postgres dumps, every night at 03:00
- `/Volumes/data/uploads`
- The `.env` files, encrypted with `age`

### Restoring

#### Postgres

```bash
gunzip -c dump.sql.gz | docker compose exec -T db psql -U app app
```

#### Uploads

Copy the folder back from the backup bucket with `rclone copy`.

## Troubleshooting

### The site returns 502

The tunnel is up but the app isn't. Run `docker compose ps` and check the logs of whatever is restarting.

### Disk is full

Docker build cache is the usual culprit:

```bash
docker system prune --volumes --filter "until=168h"
```

### Power cut

> The UPS shuts the mini down cleanly after 20 minutes. It powers back on by itself when the mains returns, because "Start up automatically after a power failure" is ticked in Energy settings.

- [x] Auto power-on enabled
- [x] Docker starts at login
- [ ] Alert when the UPS switches to battery

# docker-logseq

Run [Logseq](https://logseq.com/) in a Docker container, accessible from any browser. Designed for headless servers so you can edit your notes remotely without syncing data to the machine you're working on.

## Images

Two Dockerfiles are provided:

| File | Base | Description |
|------|------|-------------|
| `dockerfile` | `linuxserver/baseimage-kasmvnc` | Lightweight, KasmVNC-based |
| `dockerfile.webtop` | `linuxserver/webtop:ubuntu-xfce` | Recommended — XFCE desktop, Selkies streaming, built-in auth |

## Quick Start (webtop)

```bash
cp .env.example .env
# Edit .env with your username and password
docker compose up -d
```

Then open `https://<your-server-ip>` in your browser and accept the self-signed certificate.

## Usage without Compose

```bash
docker run -d \
  --privileged \
  -p 3000:3000 \
  -e CUSTOM_USER=yourname \
  -e PASSWORD=yourpassword \
  -e RESTART_APP=/usr/local/bin/Logseq \
  -e PUID=1000 \
  -e PGID=1000 \
  -v ./config:/config \
  -v ./notes:/notes \
  --name logseq \
  --restart unless-stopped \
  logseq-local:webtop
```

## Build

Supports both `amd64` and `arm64`:

```bash
docker build -t logseq-local:webtop -f dockerfile.webtop .
```

- **amd64**: downloads and extracts the Logseq AppImage
- **arm64**: downloads and extracts the Logseq zip release

## Volumes

| Path | Description |
|------|-------------|
| `/config` | App and desktop config (persistent) |
| `/notes` | Your Logseq graph — point Logseq here on first run |

## Ports

| Port | Description |
|------|-------------|
| `3000` | Web UI (HTTP) |
| `3001` | Web UI (HTTPS) |

## HTTPS & Reverse Proxy (Caddy)

A `Caddyfile` is included. With `docker compose`, Caddy runs alongside Logseq and handles HTTPS automatically.

**Local network (self-signed cert):**
```
:443 {
    tls internal
    reverse_proxy logseq:3000
}
```

**With a domain (automatic Let's Encrypt cert):**
```
notes.yourdomain.com {
    reverse_proxy logseq:3000
}
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `CUSTOM_USER` | Login username |
| `PASSWORD` | Login password |
| `RESTART_APP` | Path to app — auto-restarts if closed |
| `PUID` / `PGID` | User/group ID (match your host user) |

# Traefik Setup

Traefik v2.9 reverse proxy using Let's Encrypt TLS via Cloudflare DNS challenge.

## Prerequisites

- Docker and Docker Compose installed
- A Cloudflare account managing your domain's DNS
- A Cloudflare API token with **Zone → DNS → Edit** permission scoped to your domain

## First-time setup

### 1. Create the ACME storage file

Traefik requires this file to exist with strict permissions before starting:

```bash
sudo touch /etc/traefik/acme.json
sudo chmod 600 /etc/traefik/acme.json
```

### 2. Create the `.env` file

Copy the example and fill in your credentials:

```bash
cp .env.example .env
```

Edit `.env`:

```env
CLOUDFLARE_EMAIL=your@email.com
CF_DNS_API_TOKEN=your_cloudflare_api_token
```

`.env` is gitignored and will not be committed.

### 3. Start Traefik

```bash
docker compose up -d
```

## Configuration files

| File | Purpose |
|------|---------|
| `traefik.yml` | Static configuration — entrypoints, ACME, providers |
| `dynamic_conf.yml` | Dynamic configuration — routers, services, middlewares |
| `/etc/traefik/acme.json` | Let's Encrypt certificate storage (on host) |

## Adding a new service

In `dynamic_conf.yml`, add a router, service, and middleware following the existing pattern:

```yaml
http:
  routers:
    my-router:
      rule: "Host(`my-service.markridgwell.com`)"
      service: my-service
      entryPoints:
        - web-secure
      tls:
        certResolver: letsencrypt
      middlewares:
        - my-header

  services:
    my-service:
      loadBalancer:
        servers:
          - url: "https://192.168.10.10:5555"
          - url: "https://192.168.10.20:5555"

  middlewares:
    my-header:
      headers:
        customRequestHeaders:
          Host: "my-service.local"
```

Traefik watches `dynamic_conf.yml` for changes and reloads automatically — no restart required.

## Dashboard

The Traefik dashboard is available at `https://proxy.markridgwell.com` once running.

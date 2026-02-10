---
title: Self-hosting on a Raspberry Pi with Docker (and exposing it safely)
pubDate: 2026-02-10
description: A beginner-friendly, step-by-step guide to running web apps on a Raspberry Pi with Docker, then exposing them to the internet with HTTPS—plus safer alternatives like tunnels.
tags: ["raspberry-pi", "docker", "self-hosting", "homelab", "security"]
---

You *can* host your own web servers on a Raspberry Pi and expose them to the internet — and it’s a great way to learn. It’s also a great way to accidentally create a tiny internet-facing computer that’s easy to break into.

So this guide is opinionated:

- **Default recommendation (safest for most people):** use a **tunnel** (Cloudflare Tunnel / Tailscale Funnel) so you **don’t port-forward** from your home router.
- **Classic “real server” method (most universal):** port-forward **80/443** to your Pi, run a **reverse proxy**, and use **Let’s Encrypt** for HTTPS.

I’ll show both, step-by-step.

---

## Mental model (what you’re building)

Your Pi will run containers:

- a **reverse proxy** (the “front door”) that handles HTTPS and routes requests
- one or more **apps** (your websites, dashboards, APIs)

### Option A (recommended): tunnel

```
Internet  →  Tunnel provider  →  Tunnel client on your Pi  →  Your containers
(no router port forwarding)
```

### Option B (classic): port forwarding + reverse proxy

```
Internet → your domain (DNS) → your home router → Pi (80/443) → reverse proxy → containers
```

---

## Prereqs

Hardware/software:

- Raspberry Pi 4/5 recommended (2GB+ RAM is fine)
- microSD or SSD (SSD is nicer for reliability)
- Raspberry Pi OS Lite 64-bit (or another Debian-based OS)

Accounts:

- A domain name you control (example: `example.com`)
- (Optional) Cloudflare account if using Cloudflare Tunnel

---

## Step 0 — install Raspberry Pi OS + enable SSH

1. Flash **Raspberry Pi OS Lite (64-bit)** using Raspberry Pi Imager.
2. In the Imager advanced settings:
   - set a hostname (e.g. `pi-web`)
   - enable SSH
   - set a user/password (we’ll switch to SSH keys shortly)
3. Boot the Pi.
4. Find its IP address (router UI, `arp -a`, or `ping pi-web.local`).
5. SSH in:

```bash
ssh pi@pi-web.local
```

---

## Step 1 — basic OS hardening (do this before exposing anything)

### 1) Update everything

```bash
sudo apt update
sudo apt -y full-upgrade
sudo reboot
```

### 2) Use SSH keys (and disable password SSH)

On your laptop:

```bash
ssh-keygen -t ed25519 -C "pi"
ssh-copy-id pi@pi-web.local
```

Then on the Pi:

```bash
sudo nano /etc/ssh/sshd_config
```

Set (or ensure):

```text
PasswordAuthentication no
PermitRootLogin no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

### 3) Firewall (and a Docker gotcha)

Install ufw:

```bash
sudo apt -y install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

Important: Docker networking can bypass some host firewall rules unless you configure it correctly. Docker’s docs call out **ufw/firewalld limitations** and recommend using the **DOCKER-USER** chain for packet filtering.

Source: Docker Engine install docs + firewall notes: https://docs.docker.com/engine/install/debian/

If you’re a beginner, the practical takeaway is:

- don’t publish random container ports
- publish only your reverse proxy ports (typically 80/443)

---

## Step 2 — install Docker + Compose

Raspberry Pi OS is Debian-based, so the “official” path is the Docker Debian instructions.

Source: https://docs.docker.com/engine/install/debian/

A simple, commonly used approach for Pi is Docker’s install script (fine for a home lab; for production, use the apt repo method in the official docs):

```bash
curl -fsSL https://get.docker.com | sh
```

Then:

```bash
sudo usermod -aG docker $USER
newgrp docker

docker version
```

Docker Compose (plugin) is usually included now; verify:

```bash
docker compose version
```

---

## Step 3 — create a clean folder layout

I like:

```bash
mkdir -p ~/services
mkdir -p ~/services/caddy
mkdir -p ~/services/apps
```

The idea:

- each “stack” gets its own folder
- volumes live under those folders (easy backups)

---

## Step 4 — run a reverse proxy with automatic HTTPS

There are a few good choices. For “easy for anyone,” I recommend **Caddy** because it automatically gets and renews TLS certificates.

Automatic HTTPS overview (Caddy): https://caddyserver.com/docs/automatic-https

### 4A) Caddy + a sample app (compose)

Create: `~/services/docker-compose.yml`

```yaml
services:
  caddy:
    image: caddy:2
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./caddy/Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config

  whoami:
    image: traefik/whoami
    restart: unless-stopped

volumes:
  caddy_data:
  caddy_config:
```

Create: `~/services/caddy/Caddyfile`

```caddy
# Replace with your real domain
whoami.example.com {
  reverse_proxy whoami:80
}
```

Start it:

```bash
cd ~/services
docker compose up -d

docker compose ps
```

At this point, Caddy is listening on ports 80/443 and ready — but you still need to route internet traffic to it (next step).

---

## Step 5 — choose how to expose it publicly

### Option A (recommended): Cloudflare Tunnel (no port forwarding)

This is the “I want it to work without opening my router” path.

High level:

1. Put your domain’s DNS on Cloudflare (one-time)
2. Run `cloudflared` on the Pi
3. Create a tunnel + route `whoami.example.com` → your Caddy (or directly to the app)

Cloudflare Tunnel docs: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/

Pros:

- no ports exposed on your home router
- reduces drive-by scans against your IP

Cons:

- you’re trusting a third-party tunnel provider
- some protocols/apps need extra configuration

### Option B (classic): DNS + router port forwarding (80/443)

This is the “real internet server at home” path.

Steps:

1) **Give your Pi a static LAN IP** (either in your router DHCP reservation, or on the Pi).

2) **Set DNS records** at your domain provider:

- `A` record: `whoami.example.com` → your **home public IP**

3) **Port forward** on your router:

- forward **TCP 80** → Pi LAN IP port 80
- forward **TCP 443** → Pi LAN IP port 443

4) Wait a minute for DNS to propagate, then visit:

- `https://whoami.example.com`

If everything is correct, Caddy will obtain a certificate automatically and your page will load over HTTPS.

---

## Step 6 — deploy a real app

Once `whoami` works, swap in something real.

Example (a simple static site container):

```yaml
  site:
    image: nginx:alpine
    restart: unless-stopped
    volumes:
      - ./apps/site:/usr/share/nginx/html:ro
```

And in `Caddyfile`:

```caddy
example.com {
  reverse_proxy site:80
}
```

Put an `index.html` in `~/services/apps/site/` and you’re live.

---

## Best practices that matter (don’t skip these)

### 1) Don’t expose admin panels directly

If your app has an admin UI (Grafana, Portainer, Home Assistant, etc.):

- put it behind auth
- restrict by IP when possible
- or only access it via VPN/tunnel

### 2) Backups: copy *volumes*, not just compose files

Your important data lives in:

- bind-mounted folders (like `./apps/...`)
- Docker volumes (like `caddy_data`)

Back up both.

### 3) Updates: OS + containers

A reasonable home setup:

- monthly `apt update && apt full-upgrade`
- regularly pull new images and restart:

```bash
docker compose pull
docker compose up -d
```

### 4) Logs + monitoring

At minimum:

```bash
docker compose logs -f --tail=100
```

If you go further later, add Uptime Kuma or Prometheus/Grafana.

### 5) Security reality check

If you do classic port forwarding, your IP will get scanned. That’s normal.

The goal is:

- only expose 80/443
- keep software updated
- avoid weak passwords
- don’t run random images you don’t trust

---

## A beginner-friendly “recommended stack”

If you want the simplest reliable setup:

- Raspberry Pi OS Lite
- Docker + Compose
- Caddy as reverse proxy
- Cloudflare Tunnel (optional but recommended)

That gets you:

- one box
- one command to bring services up/down
- HTTPS without manual certificate pain

---

## Troubleshooting checklist

- **Domain not working?**
  - check DNS `A` record points to the right public IP
  - if your IP changes, consider a dynamic DNS solution

- **HTTPS failing?**
  - ensure ports **80 and 443** reach Caddy
  - Let’s Encrypt needs port 80/443 reachable for most HTTP challenge flows

- **It works locally but not from the internet?**
  - router port forwarding is wrong, or ISP is blocking inbound 80/443
  - in that case, use a tunnel

---

## Sources

- Docker Engine install (Debian) + firewall limitations: https://docs.docker.com/engine/install/debian/
- Caddy automatic HTTPS: https://caddyserver.com/docs/automatic-https
- Cloudflare Tunnel docs: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/

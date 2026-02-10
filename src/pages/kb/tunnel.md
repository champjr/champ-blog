---
layout: ../../layouts/BaseLayout.astro
title: What is a tunnel?
description: A plain-English explanation of tunnels (and why they’re useful for self-hosting).
---

# What is a “tunnel”?

A **tunnel** is a way for your computer (or Raspberry Pi) to make an **outgoing** connection to a trusted service, and then use that connection to let **incoming** web requests reach your server **without** opening ports on your home router.

If you remember one sentence, make it this:

> A tunnel lets your server “call out” to the internet, and then the internet can “call back” through that same connection.

---

## Why tunnels exist (the real problem they solve)

Normally, if you want the public internet to reach something in your house, you have to:

- know your public IP address
- **port-forward** through your router (e.g., open 80/443)
- keep a firewall tight
- accept that bots will scan your IP forever

That’s doable — it’s just a lot when you’re new.

A tunnel is an alternative: **no inbound ports opened**, and your home IP can stay hidden.

---

## A simple analogy

Imagine your Raspberry Pi is in a building that doesn’t accept deliveries.

- **Port forwarding** is like installing a public door and hoping only the right people walk in.
- A **tunnel** is like your Pi hiring a courier who already has access to the outside world.
  - Your Pi calls the courier (outbound).
  - The courier brings visitors back to your Pi through a private hallway.

---

## What tunnels look like in practice

Common tunnel approaches:

- **Cloudflare Tunnel** (`cloudflared`)
- **Tailscale** (VPN-style access; some features can publish services)

With these, your Pi runs a small client that maintains the tunnel connection.

---

## Pros and cons

### Pros

- You usually **don’t need router changes**.
- Your home IP is less exposed to random scanning.
- Great for beginners and for sensitive apps (dashboards, admin panels).

### Cons

- You’re often relying on a third party (or at least a second moving piece).
- If the tunnel is down, your site is down.
- Some protocols / “non-HTTP” apps need extra setup.

---

## How it relates to my Raspberry Pi + Docker post

In the self-hosting guide, a tunnel is the “safer default” way to make your site reachable from the internet without port forwarding.

- Back to the post: **[Self-hosting on a Raspberry Pi with Docker](/posts/2026-02-10-self-hosting-on-a-raspberry-pi-with-docker/)**

Next term you’ll see a lot: **[reverse proxy](/kb/reverse-proxy/)**.

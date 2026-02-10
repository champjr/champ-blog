---
layout: ../../layouts/BaseLayout.astro
title: What is a reverse proxy?
description: "A beginner-friendly explanation of reverse proxies: the ‘front door’ that routes traffic to your apps and handles HTTPS."
---

# What is a reverse proxy?

A **reverse proxy** is a server that sits in front of your apps and acts like a **single front door**.

It receives requests from the internet (like `https://example.com`) and then forwards (“proxies”) them to the right internal service running on your machine.

> In plain English: a reverse proxy is the receptionist for your web apps.

---

## Why you want one (even for a tiny homelab)

When you run multiple services (a website, a dashboard, an API), they usually want to run on different internal ports:

- website might be `localhost:3000`
- dashboard might be `localhost:8080`
- API might be `localhost:5000`

You *could* expose those ports directly to the internet… but that’s messy and often unsafe.

A reverse proxy lets you expose **only one or two ports** publicly:

- **80** (HTTP)
- **443** (HTTPS)

…and route everything else internally.

---

## What it actually does

A reverse proxy commonly handles:

1) **Routing by hostname**
- `app1.example.com` → app container A
- `app2.example.com` → app container B

2) **HTTPS (TLS certificates)**
- terminates HTTPS at the proxy
- forwards to internal services over your private network

3) (Often) **basic security features**
- rate limiting
- access logs
- optional authentication gates

---

## Reverse proxy vs “forward proxy” (quickly)

- A **forward proxy** is used by clients to reach the internet (e.g., a corporate proxy).
- A **reverse proxy** is used by servers to present services to the public.

---

## Common reverse proxies you’ll see

- **Caddy** (very beginner-friendly; automatic HTTPS)
- **Nginx** (classic, powerful, lots of guides)
- **Traefik** (Docker-friendly routing via labels)

In my Raspberry Pi guide I used **Caddy** because it reduces the amount of TLS plumbing you have to learn on day one.

---

## How it relates to my Raspberry Pi + Docker post

In the self-hosting guide, the reverse proxy is the part that:

- listens on 80/443
- routes traffic to your containers
- makes HTTPS easy

- Back to the post: **[Self-hosting on a Raspberry Pi with Docker](/posts/2026-02-10-self-hosting-on-a-raspberry-pi-with-docker/)**

Related: **[tunnels](/kb/tunnel/)**.

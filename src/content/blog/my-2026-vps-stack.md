---
title: My 2026 VPS Stack
description: Tailscale, Docker, Caddy, Postgres, Celery, and n8n running on a single $25/mo VPS. Here's what I run, how it's configured, and where the bottlenecks are.
author: Armin Marxer
date: 2026-05-27
wordCount: 950
image: /og-image.png
draft: false
---

I run four products on a single VPS. One Hetzner CX42 — 4 vCPU, 16 GB RAM, 160 GB NVMe. $25/month.

Before I get called out: yes, this is a single point of failure. No, I'm not worried about it yet. When one of these products justifies its own box, it'll get one. Until then, consolidation keeps costs predictable.

Here's what's running on it.

## The Stack

**Docker Compose** for everything. No Kubernetes, no Swarm. Each product has a `docker-compose.yml` and a `.env` file. One `docker-compose-proxy.yml` for the reverse proxy layer.

**Caddy** as the reverse proxy. One Caddy instance handles routing for all four products. Automatic HTTPS via Let's Encrypt. The `Caddyfile` has about 40 lines — one `handle_path` block per service. Wildcard certs for `*.kern.web.za`, `*.coolminds.co.za`, and `*.styx.web.za`.

**Tailscale** for networking. Every VPS, every dev machine, every deployment target connects via Tailscale. No open ports except the Caddy HTTP/HTTPS ports. SSH is Tailscale-only. The ops dashboard, n8n, and dev DBs are accessible only over Tailscale — no public endpoint.

**Postgres** — one instance per product. Six Postgres containers total. Each on a different port, each with its own volume. No pooling yet. Haven't needed it.

**Redis** — shared instance for Celery and caching. Lightweight. About 200 MB RAM at peak.

**n8n** — handles lead outreach automation, daily health checks, and a few cron-replacement workflows. Self-hosted behind Caddy, accessible only over Tailscale. Three active workflows, about 50 runs per day.

**Celery** — Python task queue for async processing in the Kern API. Handles email sending, blog generation calls, and a few maintenance tasks. Workers run as Docker containers, broker through Redis.

**Custom containers** — the Kern API (FastAPI), the client portal (Nginx + React build), the ops dashboard (Flask), and a few utility scripts.

## The Configuration

The Caddyfile is the simplest part:

```
styx.web.za {
    reverse_proxy dashboard:5000
}

n8n.styx.web.za {
    reverse_proxy n8n:5678
}

*.kern.web.za {
    reverse_proxy kern-api:8000
}
```

Tailscale subnet routing means I don't expose management UIs. n8n, the ops dashboard, and database ports are unreachable from the public internet. I connect via Tailscale and access them at their internal IPs.

## The Bottlenecks

**Memory, not CPU.** The CX42 has 16 GB RAM. At idle, about 6 GB is in use. Under load (all products active, blog generation running, n8n processing leads), it peaks around 11 GB. I'll hit memory limits before CPU limits.

**Disk I/O under Celery bursts.** When the blog pipeline fires for multiple clients simultaneously, Celery workers write logs, generate content, and trigger deployments. The NVMe handles it, but `iowait` spikes briefly. A dedicated Celery worker box would isolate this, but not worth it yet.

**n8n is resource-heavy for what it does.** Three workflows using about 800 MB RAM. It's Node.js running in Docker, so expected, but it's the least efficient service on the box per unit of work.

## Why This Works

Single-box deployments are unfashionable. Everything is microservices, Kubernetes, serverless. For a solo builder running multiple products, the overhead of distributed infrastructure would cost more in time and complexity than it saves in reliability.

The bet is: when something breaks, I'll know before the monitoring alerts. The VPS has been up for 8 months. The only downtime was a Hetzner host reboot that I scheduled.

When one product needs its own box, it'll get one. Until then, this stack runs everything I have.

---
name: knowledge-vps-patterns
type: knowledge
description: Reference for VPS infrastructure patterns — directory layout, CI/CD model, and the four deployment patterns. No project-specific values; read PROJECT_CONTEXT.md for those.
---

# VPS Infrastructure Patterns

## Architecture

**Two-tier Nginx**: Internet → System Nginx (SSL, reverse proxy) → Docker containers (PHP-FPM / Node.js / static).

System Nginx lives in `/etc/nginx/sites-enabled/`. Container Nginx lives inside Docker. They never share config files.

## Key Conventions

- **Directory layout**: apps in `/var/www/_projects/<app>/`, repos in `/var/www/_repos/<name>/`, secrets in `/var/www/_deploy/configs/`
- **CI/CD**: cron-based polling every 1 min — check script → git fetch → compare HEAD → deploy if changed
- **Port allocation**: assign a range per app (e.g. 8300–8399); `:X0` = primary (what System Nginx proxies to)
- **Sudo workflow**: agent writes to `_temp/`, then sudo-installs to `/usr/local/bin/` or `/etc/nginx/`

## Deployment Patterns (summary)

| Pattern | App type | Core steps |
|---------|----------|------------|
| 1 | PHP/CakePHP | git checkout → copy app_local.php → composer install → migrations |
| 2 | Frontend SPA | git checkout → temp dir → docker build → copy dist → cleanup |
| 3 | Node.js | git checkout → copy .env → docker compose down → up --build |
| 4 | Static site | git checkout → app dir (direct) |

## When to use this skill

Load when provisioning a new app, writing deploy scripts, configuring Nginx, or reviewing infrastructure decisions.

For full details — directory layout, deployment pattern steps, new project checklist, Nginx config skeleton, and sudo workflow — see `./SKILL-DETAILS.md`.

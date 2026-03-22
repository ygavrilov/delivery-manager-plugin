---
name: knowledge-vps-patterns
type: knowledge
description: Reference for VPS infrastructure patterns — directory layout, CI/CD model, and the four deployment patterns. No project-specific values; read PROJECT_CONTEXT.md for those.
---

## Architecture

**Two-tier Nginx:**

```
Internet → System Nginx (SSL termination, reverse proxy by domain)
               └→ Docker Nginx (app files + PHP-FPM)   [PHP apps]
               └→ Docker Node.js container              [Node.js apps]
               └→ Docker Nginx (static files only)      [Static sites]
```

System Nginx lives in `/etc/nginx/sites-enabled/`. Container Nginx lives inside Docker. They never share config files.

---

## Directory Layout

```
/var/www/
├── _projects/<app>/        # Running app (docker-compose.yml, app/, logs/, nginx/)
├── _repos/<name>/          # Bare git clone — source of truth for deploys
├── _deploy/configs/        # Env files and secrets — NOT in git
├── _temp/                  # Staging area — agent writes here, sudo installs from here
├── _backups/               # DB backups
└── [legacy apps]/          # Apps not yet migrated to _projects/
```

**Convention:** underscore prefix = infrastructure folder, not a web app.

---

## CI/CD Model

All apps use **cron-based polling** (every 1 minute):

```
cron → check script → git fetch → compare HEAD
    → if changed: git pull → deploy script
    → if unchanged: log "no changes"
```

Scripts live in `/usr/local/bin/`. Logs in `/var/log/update_<app>.log`.

---

## Port Allocation

Assign a port range per app (e.g. 8300–8399). Within the range:
- `:X0` → Nginx / primary service (the one System Nginx proxies to)
- `:X1`, `:X2`, ... → PHP-FPM, DB, WebSocket, etc.

Document the range in `PROJECT_CONTEXT.md` when a project is created.

---

## Deployment Patterns

### Pattern 1 — PHP App (CakePHP)

```
git checkout → copy app_local.php
→ composer install --no-dev
→ bin/cake plugin assets symlink
→ bin/cake cache clear_all
→ bin/cake migrations migrate
```

- App files land in `_projects/<app>/app/` via `git --work-tree` checkout
- `app_local.php` is never in git — copy from `_deploy/configs/`
- DB host in config is the **Docker service name**, not `localhost`

### Pattern 2 — Frontend SPA (Vite/Vue)

```
git checkout → temp dir → copy .env
→ docker build (Node 20 Alpine)
→ docker run → docker cp /app/dist → cleanup container + image
```

- Never overwrite prod files until build succeeds
- Temp dir and build container are cleaned up on exit (use `trap cleanup EXIT`)

### Pattern 3 — Node.js App

```
git checkout → copy .env
→ docker compose down → docker compose up -d --build
```

- App dir is `_projects/<app>/app/`
- Env file from `_deploy/configs/`
- Container rebuild on every deploy (no in-place update)

### Pattern 4 — Static Site

```
git checkout → app dir (direct)
```

- Simplest pattern — no build, no container restart
- Use `rm -rf app/*` before checkout if the repo can delete files

---

## New Project Checklist

1. Create `_projects/<app>/{app,logs,nginx}/` and `_repos/<name>/`
2. Clone repo into `_repos/<name>/`
3. Write `docker-compose.yml` using the appropriate pattern
4. Write env file into `_deploy/configs/`
5. Write check + deploy scripts into `_temp/scripts/`, then sudo-install to `/usr/local/bin/`
6. Write nginx config into `_temp/nginx/`, then sudo-install to `/etc/nginx/sites-available/` and symlink
7. Issue SSL cert: `sudo certbot certonly --nginx -d <domain>`
8. Run initial deploy manually
9. Add cron entry: `*/1 * * * * /usr/local/bin/check_<app>_updates.sh >> /var/log/update_<app>.log 2>&1`
10. Add site to monitoring script

---

## Nginx Config Structure

Every site follows the same skeleton — no exceptions:

```nginx
# HTTP → HTTPS redirect
server {
    listen <ip>:80;
    server_name <domain>;
    return 301 https://$host$request_uri;
}

# HTTPS
server {
    listen <ip>:443 ssl http2;
    server_name <domain>;

    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://127.0.0.1:<HOST_PORT>;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Add CORS headers, WebSocket upgrade, or `client_max_body_size` only when the app requires it.

---

## Sudo Workflow

The agent never runs system-level writes directly. It stages files in `_temp/` and uses the scoped sudoers rule (see `vps-ssh-setup`) to install them:

```
agent writes → _temp/scripts/deploy_<app>.sh
agent runs  → sudo cp _temp/scripts/deploy_<app>.sh /usr/local/bin/
agent runs  → sudo chmod +x /usr/local/bin/deploy_<app>.sh
```

Same for nginx configs. This is the boundary — `_temp/` is free-write, everything else goes through sudo.

# VPS — Reference

## One-Time SSH Setup

### Step 1 — Generate SSH key (local)

Use a dedicated key:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/claude_devops -C "claude-devops"
```

### Step 2 — Authorize on VPS

```bash
ssh-copy-id -i ~/.ssh/claude_devops.pub <user>@<vps-ip>
```

Verify (no password):

```bash
ssh -i ~/.ssh/claude_devops <user>@<vps-ip> echo OK
```

### Step 3 — Sudoers rule

```bash
sudo visudo -f /etc/sudoers.d/claude-devops
```

Paste (replace `<user>`):

```
<user> ALL=(ALL) NOPASSWD: \
    /usr/local/bin/deploy_*, \
    /usr/local/bin/build_*, \
    /usr/local/bin/check_*_updates*, \
    /usr/sbin/nginx -t, \
    /usr/sbin/nginx -s reload, \
    /usr/bin/certbot *, \
    /bin/cp /var/www/_temp/* /usr/local/bin/*, \
    /bin/cp /var/www/_temp/* /etc/nginx/*, \
    /bin/chmod +x /usr/local/bin/*, \
    /bin/touch /var/log/update_*.log, \
    /bin/ln -sf /etc/nginx/sites-available/* /etc/nginx/sites-enabled/*
```

Verify: `sudo nginx -t` (no password prompt expected).

### Step 4 — Record in PROJECT_CONTEXT.md

```
VPS: <user>@<ip> | key: ~/.ssh/claude_devops | workdir: /var/www/
```

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

Underscore prefix = infrastructure folder, not a web app.

---

## CI/CD Model

All apps use cron-based polling (every 1 minute):

```
cron → check script → git fetch → compare HEAD
    → if changed: git pull → deploy script
    → if unchanged: log "no changes"
```

Scripts in `/usr/local/bin/`. Logs in `/var/log/update_<app>.log`.

---

## Port Allocation

Assign a port range per app (e.g. 8300–8399):
- `:X0` → Nginx / primary service (proxied by system Nginx)
- `:X1`, `:X2`, ... → PHP-FPM, DB, WebSocket, etc.

Document range in `PROJECT_CONTEXT.md` when project is created.

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
- `app_local.php` never in git — copy from `_deploy/configs/`
- DB host in config is Docker service name, not `localhost`

### Pattern 2 — Frontend SPA (Vite/Vue)

```
git checkout → temp dir → copy .env
→ docker build (Node 20 Alpine)
→ docker run → docker cp /app/dist → cleanup container + image
```

- Never overwrite prod files until build succeeds
- Temp dir and build container cleaned up on exit (`trap cleanup EXIT`)

### Pattern 3 — Node.js App

```
git checkout → copy .env
→ docker compose down → docker compose up -d --build
```

- App dir: `_projects/<app>/app/`
- Env file from `_deploy/configs/`
- Container rebuild on every deploy

### Pattern 4 — Static Site

```
git checkout → app dir (direct)
```

- No build, no container restart
- Use `rm -rf app/*` before checkout if repo can delete files

---

## New Project Checklist

1. Create `_projects/<app>/{app,logs,nginx}/` and `_repos/<name>/`
2. Clone repo into `_repos/<name>/`
3. Write `docker-compose.yml` for appropriate pattern
4. Write env file into `_deploy/configs/`
5. Write check + deploy scripts into `_temp/scripts/`, sudo-install to `/usr/local/bin/`
6. Write nginx config into `_temp/nginx/`, sudo-install to `/etc/nginx/sites-available/` + symlink
7. Issue SSL cert: `sudo certbot certonly --nginx -d <domain>`
8. Run initial deploy manually
9. Add cron: `*/1 * * * * /usr/local/bin/check_<app>_updates.sh >> /var/log/update_<app>.log 2>&1`
10. Add site to monitoring script

---

## Nginx Config Structure

```nginx
server {
    listen <ip>:80;
    server_name <domain>;
    return 301 https://$host$request_uri;
}

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

Add CORS headers, WebSocket upgrade, or `client_max_body_size` only when app requires it.

---

## Sudo Workflow

Agent never runs system-level writes directly. Stages in `_temp/`, installs via scoped sudoers:

```
agent writes → _temp/scripts/deploy_<app>.sh
agent runs  → sudo cp _temp/scripts/deploy_<app>.sh /usr/local/bin/
agent runs  → sudo chmod +x /usr/local/bin/deploy_<app>.sh
```

Same for nginx configs. `_temp/` is free-write; everything else goes through sudo.

---

## Operations Rules

### SSH Command Pattern

```bash
ssh -i ~/.ssh/claude_devops <user>@<ip> '<command>'
```

Connection details from `PROJECT_CONTEXT.md` VPS line. If missing, ask user to add it.

### File Staging Rule

Never write directly to system paths. Always: write to `_temp/` → install via scoped sudo.

### Decision Rules

1. **Pattern first** — check this reference before writing any deploy script or nginx config
2. **Check before create** — `ls` target dir before writing; do not overwrite without confirming
3. **Test before activate** — `sudo nginx -t` before `sudo nginx -s reload`
4. **Verify after deploy** — check `docker ps` and `curl -I https://<domain>/`
5. **Lean changes** — only modify what was asked

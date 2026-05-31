# VPS Operations Reference - Complete DevOps Knowledge Base

**Server**: Linux 5.4.0-167-generic | **IP**: 185.113.134.130 | **Working Directory**: `/var/www/`

This document contains everything needed to operate, maintain, deploy, monitor, and extend the VPS infrastructure. It serves as the single source of truth for building DevOps agents and skills.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Directory Structure](#2-directory-structure)
3. [Application Registry](#3-application-registry)
4. [Docker Infrastructure](#4-docker-infrastructure)
5. [Nginx Configuration](#5-nginx-configuration)
6. [Deployment System](#6-deployment-system)
7. [Monitoring & Backup](#7-monitoring--backup)
8. [Database Operations](#8-database-operations)
9. [SSL & Certificates](#9-ssl--certificates)
10. [New Project Setup](#10-new-project-setup)
11. [Troubleshooting Runbooks](#11-troubleshooting-runbooks)
12. [Script Reference](#12-script-reference)
13. [Configuration Files Reference](#13-configuration-files-reference)
14. [Cron Jobs](#14-cron-jobs)
15. [Security Notes](#15-security-notes)

---

## 1. Architecture Overview

### Traffic Flow

```
Internet --> System Nginx (HTTPS:443, SSL termination)
               |
               +--> Docker Nginx (localhost ports) --> PHP-FPM (CakePHP apps)
               +--> Docker Node.js (localhost ports)  (Node.js apps)
               +--> Docker Nginx (localhost ports)     (static sites)
```

### Two-Tier Nginx

- **System Nginx** (`/etc/nginx/`) - Reverse proxy, SSL termination, routes traffic by domain
- **Container Nginx** (inside Docker) - Serves application files, routes to PHP-FPM

### CI/CD Model

All apps use **cron-based polling** (every 1 minute):
1. Cron runs update-check script
2. Script does `git fetch` and compares local vs remote HEAD
3. If different: `git pull` + deployment script
4. Deployment: checkout code, copy configs, install deps, run migrations, restart

### Application Types

| Type | Stack | Deploy Method | Examples |
|------|-------|---------------|----------|
| **PHP API** | CakePHP + Nginx + MariaDB + Ofelia | git checkout + composer + migrations | bot, crm API, tracker API |
| **Frontend SPA** | Vue.js/Vite + Nginx | Docker build + extract static files | crm-front, tracker-front |
| **Node.js API** | Node.js + Express | git checkout + docker rebuild | ah.gavrilov.kz |
| **Static Site** | Nginx only | git checkout files | bomb-royale, otaustore, gavrilov.kz |

---

## 2. Directory Structure

```
/var/www/
|
+-- _projects/              # New-style web applications
|   +-- ah.gavrilov.kz/     # Node.js/Playwright API (port 8600)
|   +-- bomb-royale/        # Static site (port 8500)
|   +-- gavrilov.kz/        # Main website (port 8080)
|   +-- otaustore/          # Static site (port 8700)
|
+-- _repos/                 # Git repository clones (new-style)
|   +-- kaspi-parser/       # Source for ah.gavrilov.kz
|   +-- bomber-royale/      # Source for bomb-royale
|   +-- gavrilov.kz/        # Source for gavrilov.kz
|   +-- otaustore/          # Source for otaustore
|
+-- _deploy/                # Production-ready configs
|   +-- configs/            # Environment files (.env, app_local.php)
|   +-- claude-code-skill-master-plugin/  # Skill master plugin
|
+-- _temp/                  # Staging area (Claude works here)
|   +-- scripts/            # Draft deployment scripts
|   +-- nginx/              # Draft nginx configs
|
+-- _templates/             # Project templates
|   +-- nodejs-api/         # Template for Node.js APIs
|
+-- _docs/                  # Documentation
+-- _backups/               # Database backups (created by db-backup-rotate)
+-- _archive/               # Archived/inactive projects
|
+-- [Legacy apps - not yet migrated to _projects/]
|   +-- bot.gavrilov.kz/    # Bot app (port 8480)
|   +-- crm/                # CRM API (port 8380)
|   +-- crm-front/          # CRM frontend (port 8300)
|   +-- tracker/            # Tracker (port 8088)
|
+-- configs/                # Legacy configs (migrating to _deploy/configs/)
+-- git_repos/              # Legacy git repos (migrating to _repos/)
|   +-- bots/               # Source for bot.gavrilov.kz
|   +-- crm/                # Source for crm API
|   +-- crm-front/          # Source for crm frontend
|   +-- kaspi-api/          # Source for tracker API
|   +-- kaspi-app/          # Source for tracker frontend
|
+-- html/                   # Default web root
```

### Naming Convention

| Prefix | Meaning |
|--------|---------|
| `_` (underscore) | Infrastructure folder (not a web app) |
| No prefix | Web application directory |

---

## 3. Application Registry

### Complete Port Map

| Application | Domain | Location | Service | Host Port | Container Port |
|-------------|--------|----------|---------|-----------|----------------|
| gavrilov.kz | gavrilov.kz | `_projects/gavrilov.kz/` | nginx | 8080 | 80 |
| gavrilov.kz | | | php | 8000 | 9000 |
| bot | bot.gavrilov.kz | `bot.gavrilov.kz/` | nginx | 8480 | 80 |
| bot | | | php | 8402 | 9000 |
| bot | | | db | 8403 | 3306 |
| crm-front | crm.gavrilov.kz | `crm-front/` | nginx | 8300 | 80 |
| crm API | api.gavrilov.kz | `crm/` | nginx | 8380 | 80 |
| crm API | | | php | 8302 | 9000 |
| crm API | | | db | 8303 | 3306 |
| crm API | | | websocket | 8399 | 8080 |
| tracker | tracker.gavrilov.kz | `tracker/` | nginx | 8088 | 80 |
| tracker | | | php | 9001 | 9000 |
| tracker | | | db | 3307 | 3306 |
| bomb-royale | bomb-royale.gavrilov.kz | `_projects/bomb-royale/` | nginx | 8500 | 80 |
| ah API | ah.gavrilov.kz | `_projects/ah.gavrilov.kz/` | node | 8600 | 3000 |
| otaustore | otaustore.gavrilov.kz | `_projects/otaustore/` | nginx | 8700 | 80 |

### Port Allocation Ranges

| Range | Reserved For |
|-------|-------------|
| 8000-8099 | gavrilov.kz |
| 8300-8399 | CRM (frontend + API) |
| 8400-8499 | Bot |
| 8500-8599 | Bomb Royale |
| 8600-8699 | ah.gavrilov.kz |
| 8700-8799 | otaustore |
| 8800-8899 | Available |
| 8900-8999 | Available |
| 9000-9099 | Tracker |

### Git Repository Map

| Repository | Clone Location | Remote URL | Branch | Deploy Target |
|------------|---------------|------------|--------|---------------|
| bots | `git_repos/bots` | `git@github.com:ygavrilov/bots.git` | master | `bot.gavrilov.kz/app` |
| crm | `git_repos/crm` | `git@github.com:ygavrilov/crm.git` | main | `crm/app` |
| crm-front | `git_repos/crm-front` | `git@github.com:ygavrilov/crm-front.git` | master | `crm-front/app` |
| kaspi-api | `git_repos/kaspi-api` | `git@github.com:ygavrilov/kaspi-api.git` | master | `tracker/api` |
| kaspi-app | `git_repos/kaspi-app` | `git@github.com:ygavrilov/kaspi-app.git` | master | `tracker/app` |
| kaspi-parser | `_repos/kaspi-parser` | GitHub | master | `_projects/ah.gavrilov.kz/app` |
| bomber-royale | `_repos/bomber-royale` | GitHub | master | `_projects/bomb-royale/app` |
| gavrilov.kz | `_repos/gavrilov.kz` | GitHub | master | `_projects/gavrilov.kz/app` |
| otaustore | `_repos/otaustore` | GitHub | master | `_projects/otaustore/app` |

**Note**: Legacy repos use `git_repos/` and sometimes remote name `github` instead of `origin`. New repos use `_repos/` and remote `origin`.

---

## 4. Docker Infrastructure

### Docker Compose Files - Complete Contents

#### bot.gavrilov.kz (`/var/www/bot.gavrilov.kz/docker-compose.yml`)

```yaml
services:
    bot_nginx:
        container_name: bot_nginx
        image: nginx
        ports:
            - "127.0.0.1:8480:80"
        links:
            - bot_php
        volumes:
            - ./nginx/conf:/etc/nginx/conf.d
            - ./app:/var/www
            - ./nginx/logs:/var/logs

    bot_php:
        container_name: bot_php
        build:
            context: ./php
            dockerfile: Dockerfile
        links:
            - bot_db
        volumes:
            - ./app:/var/www
        ports:
            - "127.0.0.1:8402:9000"
        environment:
            - TZ=Asia/Astana

    bot_db:
        image: mariadb:10.6
        container_name: bot_db
        ports:
            - "127.0.0.1:8403:3306"
        environment:
            - MYSQL_ROOT_PASSWORD=mypassword
            - MYSQL_DATABASE=bot_db
            - MYSQL_USER=bot_usr
            - MYSQL_PASSWORD=mypassword
        volumes:
            - ./mariadb:/var/lib/mysql:rw
```

#### CRM API (`/var/www/crm/docker-compose.yml`)

```yaml
services:
    crm_nginx:
        container_name: crm_nginx
        image: nginx
        ports:
            - "127.0.0.1:8380:80"
        links:
            - crm_php
        volumes:
            - ./nginx/conf:/etc/nginx/conf.d
            - ./app:/var/www
            - ./nginx/logs:/var/logs

    crm_php:
        container_name: crm_php
        build:
            context: ./php
            dockerfile: Dockerfile
        links:
            - crm_db
        volumes:
            - ./app:/var/www
        ports:
            - "127.0.0.1:8302:9000"
        environment:
            - TZ=Asia/Astana
        labels:
            ofelia.enabled: "true"
            ofelia.job-exec.ProcessWebhookQueue.user: "yg"
            ofelia.job-exec.ProcessWebhookQueue.schedule: "@every 15s"
            ofelia.job-exec.ProcessWebhookQueue.command: "bin/cake ProcessWebhookQueue"
            ofelia.job-exec.ProcessWebhookQueue.container: "crm_php"
            ofelia.job-exec.RunJobs.user: "yg"
            ofelia.job-exec.RunJobs.schedule: "@every 10s"
            ofelia.job-exec.RunJobs.command: "bin/cake RunJobs"
            ofelia.job-exec.RunJobs.container: "crm_php"

    crm_db:
        image: mariadb:10.6
        container_name: crm_db
        build:
            context: ./mariadb
        ports:
            - "127.0.0.1:8303:3306"
        environment:
            - MYSQL_ROOT_PASSWORD=mypassword
            - MYSQL_DATABASE=crm_db
            - MYSQL_USER=crm_usr
            - MYSQL_PASSWORD=mypassword
        volumes:
            - ./mariadb:/var/lib/mysql:rw

    ofelia:
        container_name: crm_scheduler
        image: mcuadros/ofelia:latest
        depends_on:
            - crm_php
        command: daemon --docker
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock:ro
        labels:
            ofelia.enabled: "true"
        restart: unless-stopped

    crm_websocket:
        container_name: crm_websocket
        build:
            context: ./websocket
            dockerfile: Dockerfile
        ports:
            - "127.0.0.1:8399:8080"
        volumes:
            - ./websocket:/app
```

#### CRM Frontend (`/var/www/crm-front/docker-compose.yml`)

```yaml
version: "3.9"
services:
    nginx:
        container_name: crm_front_nginx
        image: nginx
        restart: always
        ports:
            - "127.0.0.1:8300:80"
        volumes:
            - ./nginx/conf:/etc/nginx/conf.d
            - ./app:/var/www
            - ./nginx/logs:/var/logs
```

#### Tracker (`/var/www/tracker/docker-compose.yml`)

```yaml
version: '3'
services:
  tracker_nginx:
    container_name: tracker_nginx
    image: nginx
    ports:
      - "127.0.0.1:8088:80"
    links:
      - tracker_php
    volumes:
      - ./nginx/conf:/etc/nginx/conf.d
      - ./app:/var/www
      - ./api:/var/www_api
      - ./nginx/logs:/var/logs

  tracker_php:
    container_name: tracker_php
    build:
        context: ./php
        dockerfile: Dockerfile
    links:
        - tracker_db
    volumes:
        - ./app:/var/www
        - ./api:/var/www_api
    ports:
        - "127.0.0.1:9001:9000"
    environment:
        - TZ=Asia/Astana
    labels:
        ofelia.enabled: "true"
        ofelia.job-exec.kaspi_order_sync.user: "yg"
        ofelia.job-exec.kaspi_order_sync.schedule: "@every 1m"
        ofelia.job-exec.kaspi_order_sync.command: "/var/www_api/bin/cake kaspi_order_sync"
        ofelia.job-exec.kaspi_order_sync.container: "tracker_php"

  tracker_db:
        image: mariadb:10.6
        container_name: tracker_db
        ports:
            - "127.0.0.1:3307:3306"
        environment:
            - MYSQL_ROOT_PASSWORD=mypassword
            - MYSQL_DATABASE=tracker_db
            - MYSQL_USER=tracker_usr
            - MYSQL_PASSWORD=mypassword
        volumes:
            - ./mariadb:/var/lib/mysql:rw

  tracker_ofelia:
      container_name: tracker_scheduler
      image: mcuadros/ofelia:latest
      depends_on:
          - tracker_php
      command: daemon --docker
      volumes:
          - /var/run/docker.sock:/var/run/docker.sock:ro
      labels:
          ofelia.enabled: "true"
      restart: unless-stopped
```

#### ah.gavrilov.kz (`/var/www/_projects/ah.gavrilov.kz/docker-compose.yml`)

```yaml
version: '3.8'
services:
  ah_api:
    build:
      context: ./app
      dockerfile: ../Dockerfile
    container_name: ah_api
    restart: unless-stopped
    ports:
      - "127.0.0.1:8600:3000"
    environment:
      - NODE_ENV=production
      - TZ=Asia/Astana
    volumes:
      - ./logs:/app/logs
    networks:
      - ah_network

networks:
  ah_network:
    driver: bridge
```

#### bomb-royale (`/var/www/_projects/bomb-royale/docker-compose.yml`)

```yaml
version: "3.9"
services:
    nginx:
        container_name: bomb_royale_nginx
        image: nginx
        restart: always
        ports:
            - "127.0.0.1:8500:80"
        volumes:
            - ./nginx/conf:/etc/nginx/conf.d
            - ./app:/var/www
            - ./nginx/logs:/var/logs
```

#### otaustore (`/var/www/_projects/otaustore/docker-compose.yml`)

```yaml
version: '3.8'
services:
  otaustore_nginx:
    image: nginx:latest
    container_name: otaustore_nginx
    restart: unless-stopped
    ports:
      - "127.0.0.1:8700:80"
    volumes:
      - ./app:/usr/share/nginx/html:ro
    networks:
      - otaustore_network

networks:
  otaustore_network:
    driver: bridge
```

#### gavrilov.kz (`/var/www/_projects/gavrilov.kz/docker-compose.yml`)

```yaml
version: "3.9"
services:
    nginx_gavrilov:
        image: nginx
        ports:
            - "127.0.0.1:8080:80"
        links:
            - php_gavrilov
        volumes:
            - ./nginx/conf:/etc/nginx/conf.d
            - ./app/webroot:/var/www
            - ./nginx/logs:/var/logs
    php_gavrilov:
        container_name: php_gavrilov
        build:
            context: ./php
            dockerfile: Dockerfile
        volumes:
            - ./app/webroot:/var/www
        ports:
            - "127.0.0.1:8000:9000"
```

### Docker Common Commands

```bash
# View running containers
docker ps
docker ps -a                          # Include stopped

# Container logs
docker logs <container_name>
docker logs -f <container_name>       # Follow

# Execute inside container
docker exec -it <container> bash
docker exec <container> bin/cake <command>
docker exec <container> composer install --no-dev

# Restart
docker restart <container>
cd /var/www/<app> && docker-compose restart

# Full rebuild
cd /var/www/<app> && docker-compose down && docker-compose up -d --build

# Cleanup
docker system prune -a                # WARNING: removes all unused images
```

### Ofelia Scheduled Jobs

| Container | Job | Schedule | Command |
|-----------|-----|----------|---------|
| crm_php | ProcessWebhookQueue | @every 15s | `bin/cake ProcessWebhookQueue` |
| crm_php | RunJobs | @every 10s | `bin/cake RunJobs` |
| tracker_php | kaspi_order_sync | @every 1m | `/var/www_api/bin/cake kaspi_order_sync` |

---

## 5. Nginx Configuration

### Main Nginx Config (`/etc/nginx/nginx.conf`)

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_names_hash_bucket_size 64;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    gzip on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### Site Configurations

All sites follow the pattern: HTTP redirect to HTTPS, SSL via Let's Encrypt, proxy_pass to Docker container.

#### gavrilov.kz

```nginx
server {
    server_name gavrilov.kz www.gavrilov.kz;
    access_log /var/log/nginx/gavrilov.kz.access.log combined;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 185.113.134.130:443 ssl;
    ssl_certificate /etc/letsencrypt/live/gavrilov.kz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/gavrilov.kz/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server {
    if ($host = www.gavrilov.kz) { return 301 https://$host$request_uri; }
    if ($host = gavrilov.kz) { return 301 https://$host$request_uri; }
    listen 185.113.134.130:80;
    server_name gavrilov.kz www.gavrilov.kz;
    return 404;
}
```

#### bot.gavrilov.kz (with CORS)

```nginx
server {
    listen 185.113.134.130:80;
    server_name bot.gavrilov.kz www.bot.gavrilov.kz;
    location /.well-known/acme-challenge/ { root /var/www/html; }
    location / { return 301 https://$host$request_uri; }
}
server {
    listen 185.113.134.130:443 ssl;
    server_name bot.gavrilov.kz www.bot.gavrilov.kz;
    access_log /var/log/nginx/bot.gavrilov.kz.access.log combined;

    ssl_certificate /etc/letsencrypt/live/bot.gavrilov.kz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/bot.gavrilov.kz/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        if ($request_method = OPTIONS) {
            add_header 'Access-Control-Allow-Methods' 'DELETE, PUT, PATCH, GET, POST, OPTIONS';
            return 204;
        }
        proxy_pass http://127.0.0.1:8480;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        add_header Access-Control-Allow-Origin $http_origin always;
        add_header Access-Control-Allow-Credentials 'true';
    }
}
```

#### api.gavrilov.kz (with WebSocket + CORS)

```nginx
server {
    server_name api.gavrilov.kz;
    access_log /var/log/nginx/api.gavrilov.kz.access.log combined;
    client_max_body_size 64M;

    location / {
        if ($request_method = OPTIONS) {
            add_header 'Access-Control-Allow-Methods' 'DELETE, PUT, PATCH, GET, POST, OPTIONS';
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Access-Control-Allow-Origin' $http_origin always;
            add_header 'Access-Control-Allow-Headers' *;
            add_header Access-Control-Allow-Credentials 'true';
            return 204;
        }
        proxy_pass http://127.0.0.1:8380;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /websocket {
        proxy_pass http://127.0.0.1:8399;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 300s;
        proxy_connect_timeout 75s;
    }

    listen 185.113.134.130:443 ssl;
    ssl_certificate /etc/letsencrypt/live/api.gavrilov.kz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.gavrilov.kz/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server {
    if ($host = api.gavrilov.kz) { return 301 https://$host$request_uri; }
    server_name api.gavrilov.kz;
    listen 185.113.134.130:80;
    return 404;
}
```

#### crm.gavrilov.kz

```nginx
server {
    server_name crm.gavrilov.kz;
    access_log /var/log/nginx/crm.gavrilov.kz.access.log combined;

    location / {
        if ($request_method = OPTIONS) {
            add_header 'Access-Control-Allow-Methods' 'DELETE, PUT, PATCH, GET, POST, OPTIONS';
            return 204;
        }
        proxy_pass http://127.0.0.1:8300;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        add_header Access-Control-Allow-Origin $http_origin always;
        add_header Access-Control-Allow-Credentials 'true';
    }

    listen 185.113.134.130:443 ssl;
    ssl_certificate /etc/letsencrypt/live/crm.gavrilov.kz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/crm.gavrilov.kz/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server {
    if ($host = crm.gavrilov.kz) { return 301 https://$host$request_uri; }
    server_name crm.gavrilov.kz;
    listen 185.113.134.130:80;
    return 404;
}
```

#### tracker.gavrilov.kz

```nginx
server {
    server_name tracker.gavrilov.kz;
    access_log /var/log/nginx/tracker.gavrilov.kz.access.log combined;

    location / {
        proxy_pass http://127.0.0.1:8088;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 185.113.134.130:443 ssl;
    ssl_certificate /etc/letsencrypt/live/tracker.gavrilov.kz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tracker.gavrilov.kz/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}
server {
    if ($host = tracker.gavrilov.kz) { return 301 https://$host$request_uri; }
    listen 185.113.134.130:80;
    server_name tracker.gavrilov.kz;
    return 404;
}
```

#### ah.gavrilov.kz (Node.js with longer timeouts)

```nginx
server {
    listen 185.113.134.130:80;
    server_name ah.gavrilov.kz;
    return 301 https://$host$request_uri;
}
server {
    listen 185.113.134.130:443 ssl http2;
    server_name ah.gavrilov.kz;
    access_log /var/log/nginx/ah.gavrilov.kz.access.log combined;
    error_log /var/log/nginx/ah.gavrilov.kz.error.log;

    ssl_certificate /etc/letsencrypt/live/ah.gavrilov.kz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ah.gavrilov.kz/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://127.0.0.1:8600;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 60s;
        proxy_send_timeout 300s;
        proxy_read_timeout 300s;
    }
}
```

### Nginx Common Commands

```bash
sudo nginx -t                    # Test configuration
sudo nginx -s reload             # Reload without downtime
sudo systemctl restart nginx     # Full restart
sudo nginx -T                    # Dump full running config
```

### Nginx Config Template (for new sites)

```nginx
# HTTP to HTTPS redirect
server {
    listen 185.113.134.130:80;
    server_name <domain>;
    return 301 https://$host$request_uri;
}

# HTTPS server
server {
    listen 185.113.134.130:443 ssl http2;
    server_name <domain>;

    access_log /var/log/nginx/<domain>.access.log combined;
    error_log /var/log/nginx/<domain>.error.log;

    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://127.0.0.1:<HOST_PORT>;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 60s;
        proxy_send_timeout 120s;
        proxy_read_timeout 120s;
    }
}
```

---

## 6. Deployment System

### Deployment Patterns

There are 4 deployment patterns used across all applications:

#### Pattern 1: PHP Application (CakePHP)

Used by: **bot**, **crm API**, **tracker API**

```
git fetch -> compare HEAD -> git pull -> git --work-tree checkout
-> copy app_local.php -> composer install --no-dev
-> bin/cake plugin assets symlink -> bin/cake cache clear_all
-> bin/cake migrations migrate
```

#### Pattern 2: Frontend Build (Docker-based)

Used by: **crm-front**, **tracker-front**

```
git fetch -> compare HEAD -> git pull
-> mktemp build dir -> git --work-tree checkout to temp
-> copy .env -> docker build (Node.js 20)
-> docker run -> docker cp /app/webroot -> cleanup container/image
```

#### Pattern 3: Node.js Application

Used by: **ah.gavrilov.kz**

```
git fetch -> compare HEAD -> git reset --hard
-> git --work-tree checkout -> copy .env
-> docker compose down -> docker compose up -d --build
```

#### Pattern 4: Static Site

Used by: **bomb-royale**, **gavrilov.kz**, **otaustore**

```
git fetch -> compare HEAD -> git pull
-> git --work-tree checkout (directly to app dir)
```

### Complete Script Contents

#### Update Checker Scripts

All update checkers follow the same pattern. Here is the **canonical pattern**:

```bash
#!/bin/bash
REPO_DIR="/var/www/_repos/<repo-name>"   # or /var/www/git_repos/<repo>
BRANCH="master"                          # or "main"
REMOTE_NAME="origin"                     # or "github" for legacy
DEPLOY_SCRIPT="/usr/local/bin/deploy_<app>.sh"

git config --global --add safe.directory "$REPO_DIR"

cd "$REPO_DIR" || exit 1
git fetch "$REMOTE_NAME" "$BRANCH"

LOCAL=$(git rev-parse HEAD)
REMOTE=$(git rev-parse "$REMOTE_NAME/$BRANCH")

if [ "$LOCAL" != "$REMOTE" ]; then
    echo "Updates found. Pulling and deploying..."
    git pull "$REMOTE_NAME" "$BRANCH"
    "$DEPLOY_SCRIPT"
else
    echo "No changes found."
fi
```

**Variations by app**:

| Script | Repo Dir | Remote | Branch | Deploy Script |
|--------|----------|--------|--------|---------------|
| `check_for_updates_bot.sh` | `git_repos/bots` | origin | master | `deploy_bot.sh` |
| `api_github_pull.sh` | `git_repos/crm` | github | main | (inline deploy) |
| `update_crm_front.sh` | `git_repos/crm-front` | github | master | `crm_front_build.sh` |
| `check_kaspi_api_updates.sh` | `git_repos/kaspi-api` | origin | master | `deploy_tracker_api.sh` |
| `check_kaspi_app_updates.sh` | `git_repos/kaspi-app` | origin | master | `build_tracker_front.sh` |
| `check_ah_updates.sh` | `_repos/kaspi-parser` | origin | master | `deploy_ah.sh` |
| `check_otaustore_updates.sh` | `_repos/otaustore` | origin | master | `deploy_otaustore.sh` |
| `cron_update_checker_bomb_royale_prod.sh` | `_repos/bomber-royale` | origin | master | `deploy_bomb_royale_prod.sh` |
| `cron_update_checker_gavrilov_prod.sh` | `_repos/gavrilov.kz` | origin | master | `deploy_gavrilov_prod.sh` |

#### deploy_bot.sh (PHP App Pattern)

```bash
#!/bin/bash
REPO_DIR="/var/www/git_repos/bots"
APP_DIR="/var/www/bot.gavrilov.kz/app"
ENV_FILE="/var/www/configs/prod_bot_app_local.php"
APP_ENV_FILE="$APP_DIR/config/app_local.php"
BRANCH="master"
DOCKER_COMPOSE_DIR="/var/www/bot.gavrilov.kz"
CONTAINER="bot_php"

mkdir -p $APP_DIR
chown yg:yg $APP_DIR
cd $DOCKER_COMPOSE_DIR || exit 1

git --work-tree=$APP_DIR --git-dir=$REPO_DIR/.git checkout -f $BRANCH
cp $ENV_FILE $APP_ENV_FILE
chown -R yg:yg $APP_DIR

docker exec $CONTAINER composer install --no-dev --working-dir=/var/www
docker exec $CONTAINER bin/cake plugin assets symlink
docker exec $CONTAINER bin/cake cache clear_all
docker exec $CONTAINER bin/cake migrations migrate
```

#### api_github_pull.sh (CRM API - Inline Deploy)

```bash
#!/bin/bash
REPO_DIR="/var/www/git_repos/crm"
APP_DIR="/var/www/crm/app"
ENV_FILE="/var/www/configs/prod_crm_app_local.php"
APP_ENV_FILE="$APP_DIR/config/app_local.php"
BRANCH="main"
CONTAINER="crm_php"
REMOTE_NAME="github"

git config --global --add safe.directory $REPO_DIR
cd $REPO_DIR || exit

# Check + deploy inline
git fetch $REMOTE_NAME $BRANCH
LOCAL=$(git rev-parse HEAD)
REMOTE=$(git rev-parse $REMOTE_NAME/$BRANCH)

if [ $LOCAL != $REMOTE ]; then
    git pull $REMOTE_NAME $BRANCH
    git --work-tree=$APP_DIR --git-dir=$REPO_DIR/.git checkout -f
    cp $ENV_FILE $APP_ENV_FILE

    docker exec $CONTAINER composer install --working-dir=/var/www -vvv > ~/composer.log 2>&1
    docker exec $CONTAINER composer dumpautoload -o --working-dir=/var/www
    docker exec $CONTAINER bin/cake plugin assets symlink
    docker exec $CONTAINER bin/cake cache clear_all
    docker exec $CONTAINER bin/cake migrations migrate
fi
```

#### deploy_tracker_api.sh (Tracker API with validation)

```bash
#!/bin/bash
REPO_DIR="/var/www/git_repos/kaspi-api"
APP_DIR="/var/www/tracker/api"
ENV_FILE="/var/www/configs/tracker_app_local.php"
APP_ENV_FILE="$APP_DIR/config/app_local.php"
CONTAINER="tracker_php"

# Validate prerequisites exist
[ ! -d "$APP_DIR" ] && exit 1
[ ! -d "$REPO_DIR" ] && exit 1
[ ! -f "$ENV_FILE" ] && exit 1
docker ps --format '{{.Names}}' | grep -q "^$CONTAINER$" || exit 1

# Clean old files (preserve webroot and logs)
find "$APP_DIR" -mindepth 1 -maxdepth 1 ! -name 'webroot' ! -name 'logs' -exec rm -rf {} +

git --work-tree="$APP_DIR" --git-dir="$REPO_DIR/.git" checkout -f
cp "$ENV_FILE" "$APP_ENV_FILE"

docker exec -w /var/www_api "$CONTAINER" composer install --no-dev --optimize-autoloader
docker exec -w /var/www_api "$CONTAINER" bin/cake plugin assets symlink
docker exec -w /var/www_api "$CONTAINER" bin/cake cache clear_all
docker exec -w /var/www_api "$CONTAINER" bin/cake migrations migrate
```

#### crm_front_build.sh (Frontend Build Pattern)

```bash
#!/bin/bash
BRANCH=$1
GIT_DIR="/var/www/git_repos/crm-front"
BUILD_CONTAINER_NAME="vite_builder_$(date +%s)"
TARGET="/var/www/crm-front"
CLEANUP_NEEDED=false

cleanup() {
    if [ "$CLEANUP_NEEDED" = true ]; then
        [ -d "$TMP_DIR" ] && rm -rf "$TMP_DIR"
        docker stop $BUILD_CONTAINER_NAME >/dev/null 2>&1
        docker rm $BUILD_CONTAINER_NAME >/dev/null 2>&1
    fi
}
trap cleanup EXIT

if [ "$BRANCH" == "master" ]; then
    ENV_FILE="/var/www/configs/crm.gavrilov.kz.env"
else
    exit 1
fi

TMP_DIR=$(mktemp -d ${TARGET}/deploy-XXXXXXXXXX)
CLEANUP_NEEDED=true

git --work-tree=$TMP_DIR --git-dir=$GIT_DIR/.git checkout -f $BRANCH
cp $ENV_FILE $TMP_DIR/.env

docker build -t $BUILD_CONTAINER_NAME $TMP_DIR
docker run -d --name $BUILD_CONTAINER_NAME $BUILD_CONTAINER_NAME
docker cp $BUILD_CONTAINER_NAME:/app/webroot $TARGET/app

docker stop $BUILD_CONTAINER_NAME
docker rm $BUILD_CONTAINER_NAME
docker rmi $BUILD_CONTAINER_NAME
rm -rf "$TMP_DIR"
CLEANUP_NEEDED=false
```

#### build_tracker_front.sh (Tracker Frontend Build)

```bash
#!/bin/bash
BRANCH=$1
GIT_DIR="/var/www/git_repos/kaspi-app"
BUILD_CONTAINER_NAME="tracker_vite_builder_$(date +%s)"
TARGET="/var/www/tracker/app"
ENV_FILE="/var/www/configs/tracker_app_local.php"
CLEANUP_NEEDED=false

cleanup() {
    if [ "$CLEANUP_NEEDED" = true ]; then
        [ -d "$TMP_DIR" ] && rm -rf "$TMP_DIR"
        docker stop $BUILD_CONTAINER_NAME >/dev/null 2>&1
        docker rm $BUILD_CONTAINER_NAME >/dev/null 2>&1
    fi
}
trap cleanup EXIT

if [ "$BRANCH" != "master" ]; then exit 1; fi

TMP_DIR=$(mktemp -d ${TARGET}/deploy-XXXXXXXXXX)
CLEANUP_NEEDED=true

git --work-tree=$TMP_DIR --git-dir=$GIT_DIR/.git checkout -f $BRANCH
cp $ENV_FILE $TMP_DIR/.env

# Creates inline Dockerfile for Node.js 20 build
cat <<EOF > $TMP_DIR/Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
RUN ls -la /app
EOF

docker build -t $BUILD_CONTAINER_NAME $TMP_DIR
docker create --name $BUILD_CONTAINER_NAME $BUILD_CONTAINER_NAME

rm -rf $TARGET/*
mkdir -p $TARGET
docker cp $BUILD_CONTAINER_NAME:/app/webroot $TARGET
mv $TARGET/webroot/* $TARGET/
rm -rf $TARGET/webroot

docker rm $BUILD_CONTAINER_NAME
docker rmi $BUILD_CONTAINER_NAME
rm -rf "$TMP_DIR"
CLEANUP_NEEDED=false
```

#### deploy_ah.sh (Node.js App)

```bash
#!/bin/bash
set -e
REPO_DIR="/var/www/_repos/kaspi-parser"
APP_DIR="/var/www/_projects/ah.gavrilov.kz/app"
COMPOSE_DIR="/var/www/_projects/ah.gavrilov.kz"
CONFIG_SOURCE="/var/www/_deploy/configs/ah_app.env"
BRANCH="master"

cd "$REPO_DIR"
git fetch origin
git reset --hard origin/$BRANCH
git --work-tree="$APP_DIR" checkout -f $BRANCH

[ -f "$CONFIG_SOURCE" ] && cp "$CONFIG_SOURCE" "$APP_DIR/.env"

cd "$COMPOSE_DIR"
docker compose down
docker compose up -d --build
docker ps | grep ah_api
```

#### deploy_otaustore.sh (Static Site - Simple)

```bash
#!/bin/bash
set -e
REPO_DIR="/var/www/_repos/otaustore"
APP_DIR="/var/www/_projects/otaustore/app"
BRANCH="master"

cd "$REPO_DIR"
git fetch origin
git reset --hard origin/$BRANCH
git --work-tree="$APP_DIR" checkout -f $BRANCH
```

#### deploy_bomb_royale_prod.sh (Static Site)

```bash
#!/bin/bash
REPO_DIR="/var/www/_repos/bomber-royale"
APP_DIR="/var/www/_projects/bomb-royale/app"
BRANCH="master"
DOCKER_COMPOSE_DIR="/var/www/_projects/bomb-royale"

mkdir -p $APP_DIR
chown yg:yg $APP_DIR
cd $DOCKER_COMPOSE_DIR || exit 1
git --work-tree=$APP_DIR --git-dir=$REPO_DIR/.git checkout -f $BRANCH
chown -R yg:yg $APP_DIR
```

#### deploy_gavrilov_prod.sh (Static Site with full clean)

```bash
#!/bin/bash
REPO_DIR="/var/www/_repos/gavrilov.kz"
APP_DIR="/var/www/_projects/gavrilov.kz/app"
BRANCH="master"
DOCKER_COMPOSE_DIR="/var/www/_projects/gavrilov.kz"

mkdir -p $APP_DIR
chown yg:yg $APP_DIR
cd $DOCKER_COMPOSE_DIR || exit 1

rm -rf $APP_DIR/*
git --work-tree=$APP_DIR --git-dir=$REPO_DIR checkout -f $BRANCH
chown -R yg:yg $APP_DIR
```

---

## 7. Monitoring & Backup

### Site Monitor (`/usr/local/bin/site_monitor.sh`)

Runs every minute via cron. Checks all production sites and sends Telegram alerts on failure/recovery.

**Monitored sites**:

| URL | Expected Behavior |
|-----|-------------------|
| https://gavrilov.kz | HTTP 200-399 |
| https://bot.gavrilov.kz | HTTP 500 (expected - no GET handler) |
| https://crm.gavrilov.kz | HTTP 200-399 |
| https://api.gavrilov.kz | HTTP 404 (expected - no root endpoint) |
| https://tracker.gavrilov.kz | HTTP 200-399 |

**Alert behavior**:
- Sends Telegram alert on first failure only (avoids spam)
- Sends recovery alert when site comes back up
- State stored in `/var/tmp/site_monitor/`
- 10-second timeout per check

**Telegram config**:
- Bot token: `YOUR_TELEGRAM_BOT_TOKEN`
- Chat ID: `89050322`

### Database Backup (`/usr/local/bin/db-backup-rotate`)

Runs daily at 3:00 AM. Backs up all databases with rotation policy.

**Databases backed up**:
- `bot_db` from container `bot_db`
- `crm_db` from container `crm_db`
- `tracker_db` from container `tracker_db`

**Retention policy**:
- Last 3 days: keep all daily backups
- ~2 weeks old (days 12-16): keep 1 snapshot closest to day 14
- ~4 weeks old (days 26-30): keep 1 snapshot closest to day 28
- Everything else: delete

**Backup location**: `/var/www/_backups/<app>/`
**Format**: `<db>_YYYY-MM-DD.sql.gz`

**Usage**:
```bash
/usr/local/bin/db-backup-rotate          # Backup all databases
/usr/local/bin/db-backup-rotate bot      # Backup specific app
```

### Manual Backup (`/usr/local/bin/db_backup.sh`)

Legacy script. Run from an app directory - auto-detects container and DB from `docker-compose.yml`.

```bash
cd /var/www/crm && /usr/local/bin/db_backup.sh
# Output: crm_db-14-03-26.sql
```

---

## 8. Database Operations

### Database Access

| App | Container | Docker Host | Port | Database | User | Password |
|-----|-----------|-------------|------|----------|------|----------|
| bot | bot_db | bot_db:3306 | 8403 | bot_db | bot_usr | mypassword |
| crm | crm_db | crm_db:3306 | 8303 | crm_db | crm_usr | mypassword |
| tracker | tracker_db | tracker_db:3306 | 3307 | tracker_db | tracker_usr | mypassword |

**From host**: `mysql -h 127.0.0.1 -P <port> -u <user> -p`
**From container**: `docker exec -it <php_container> bash` then `mysql -h <db_service> -P 3306 -u <user> -p`
**CakePHP configs use Docker service name** (e.g., `bot_db`, `crm_db`, `tracker_db`) as host, NOT `localhost`

### CakePHP Database Commands

```bash
docker exec <php_container> bin/cake migrations migrate
docker exec <php_container> bin/cake migrations rollback
docker exec <php_container> bin/cake cache clear_all
docker exec <php_container> bin/cake plugin assets symlink
```

---

## 9. SSL & Certificates

**Provider**: Let's Encrypt via Certbot

**Active certificates**:
- gavrilov.kz (+ www.gavrilov.kz)
- bot.gavrilov.kz
- crm.gavrilov.kz
- api.gavrilov.kz
- tracker.gavrilov.kz
- ah.gavrilov.kz
- bomb-royale.gavrilov.kz
- otaustore.gavrilov.kz

**Certificate paths**: `/etc/letsencrypt/live/<domain>/`
**Auto-renewal**: Certbot systemd timer

**Commands**:
```bash
sudo certbot certificates                     # List all certs
sudo certbot renew                            # Renew all
sudo certbot renew --cert-name <domain>       # Renew specific
sudo certbot renew --dry-run                  # Test renewal
sudo certbot certonly --nginx -d <domain>     # New certificate
```

---

## 10. New Project Setup

### Step-by-Step Process

#### Step 1: Create Directory Structure

```bash
mkdir -p /var/www/_projects/<app-name>/{app,logs,nginx}
```

#### Step 2: Set Up Git Repository

```bash
mkdir -p /var/www/_repos/<repo-name>
cd /var/www/_repos/<repo-name>
git init
git remote add origin git@github.com:<user>/<repo>.git
git fetch origin
git checkout -b <branch> origin/<branch>
```

#### Step 3: Create Dockerfile

**Node.js standard**:
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

**Node.js with Playwright**:
```dockerfile
FROM mcr.microsoft.com/playwright:v1.50.0-noble
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
RUN npx playwright install chromium
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

#### Step 4: Create docker-compose.yml

```yaml
version: '3.8'
services:
  <app>_api:
    build:
      context: ./app
      dockerfile: ../Dockerfile
    container_name: <app>_api
    restart: unless-stopped
    ports:
      - "127.0.0.1:<PORT>:3000"
    environment:
      - NODE_ENV=production
      - TZ=Asia/Astana
    volumes:
      - ./logs:/app/logs
    networks:
      - <app>_network

networks:
  <app>_network:
    driver: bridge
```

#### Step 5: Create Environment Config

Create `/var/www/_deploy/configs/<app>_app.env`

#### Step 6: Create Scripts in `_temp/`

Create update checker and deploy script (see templates in Section 12).

#### Step 7: Create Nginx Config in `_temp/`

Use the nginx template from Section 5.

#### Step 8: User Deploys with Sudo

```bash
# Deploy scripts
sudo cp /var/www/_temp/scripts/check_<app>_updates.sh /usr/local/bin/
sudo cp /var/www/_temp/scripts/deploy_<app>.sh /usr/local/bin/
sudo chmod +x /usr/local/bin/check_<app>_updates.sh
sudo chmod +x /usr/local/bin/deploy_<app>.sh

# Create log file
sudo touch /var/log/update_<app>.log
sudo chown $USER:$USER /var/log/update_<app>.log

# Deploy nginx
sudo cp /var/www/_temp/nginx/<domain>.conf /etc/nginx/sites-available/
sudo ln -sf /etc/nginx/sites-available/<domain>.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo nginx -s reload
```

#### Step 9: SSL Certificate

```bash
sudo certbot certonly --nginx -d <domain>
```

#### Step 10: Initial Deploy + Cron

```bash
/usr/local/bin/deploy_<app>.sh

crontab -e
# Add: */1 * * * * /usr/local/bin/check_<app>_updates.sh >> /var/log/update_<app>.log 2>&1
```

### Verification Checklist

```bash
docker ps | grep <app>                      # Container running
docker logs <container>                     # Logs healthy
curl -I https://<domain>/                   # Site responds
tail -f /var/log/update_<app>.log           # Deploy logs
crontab -l | grep <app>                     # Cron active
```

---

## 11. Troubleshooting Runbooks

### App Not Responding (502 Bad Gateway)

```bash
# 1. Check container
docker ps | grep <app>
docker logs <container>

# 2. Check nginx config
sudo nginx -t
grep proxy_pass /etc/nginx/sites-enabled/<domain>.conf

# 3. Check port binding
docker ps --format '{{.Names}}\t{{.Ports}}' | grep <app>

# 4. Restart
docker restart <container>
# or full rebuild:
cd /var/www/<app> && docker-compose down && docker-compose up -d --build

# 5. Reload nginx
sudo nginx -s reload
```

### Deployment Not Triggering

```bash
# 1. Check logs
tail -50 /var/log/update_<app>.log

# 2. Check cron
crontab -l | grep <app>
ps aux | grep cron

# 3. Manually test
/usr/local/bin/check_<app>_updates.sh

# 4. Check git
cd /var/www/_repos/<repo>   # or git_repos/<repo>
git fetch origin
git log origin/<branch> -3
ssh -T git@github.com

# 5. Check disk
df -h
```

### Database Connection Issues

```bash
# 1. Check DB container
docker ps | grep _db
docker logs <db_container>

# 2. Test connection
mysql -h 127.0.0.1 -P <port> -u <user> -p

# 3. Verify config
# PHP apps: host must be Docker service name (bot_db, crm_db, tracker_db)
# NOT localhost or 127.0.0.1
# Port must be 3306 (internal), NOT the host port

# 4. Restart
docker restart <db_container>
```

### Frontend Build Failure

```bash
# 1. Check build logs
tail -100 /var/log/crm_github_pull.log
tail -100 /var/log/tracker_app_updates.log

# 2. Check disk space
df -h

# 3. Manual build test
/usr/local/bin/crm_front_build.sh master
/usr/local/bin/build_tracker_front.sh master

# 4. Check for stale containers
docker ps -a | grep builder
docker rm <stale_builder>
```

### WebSocket Issues (CRM)

```bash
# 1. Check container
docker ps | grep websocket
docker logs crm_websocket

# 2. Verify nginx has upgrade headers
grep -A 5 "location /websocket" /etc/nginx/sites-enabled/crm-api.gavrilov.kz.conf

# 3. Test port
curl -I http://127.0.0.1:8399

# 4. Restart
docker restart crm_websocket
```

### Scheduled Jobs Not Running (Ofelia)

```bash
# 1. Check scheduler
docker ps | grep scheduler
docker logs crm_scheduler
docker logs tracker_scheduler

# 2. Test command manually
docker exec crm_php bin/cake ProcessWebhookQueue
docker exec crm_php bin/cake RunJobs
docker exec tracker_php /var/www_api/bin/cake kaspi_order_sync

# 3. Restart scheduler
docker restart crm_scheduler
docker restart tracker_scheduler
```

---

## 12. Script Reference

### Script Location Summary

| Script | Location | Purpose | Called By |
|--------|----------|---------|-----------|
| `check_for_updates_bot.sh` | `/usr/local/bin/` | Check bot updates | Cron |
| `deploy_bot.sh` | `/usr/local/bin/` | Deploy bot app | Update checker |
| `api_github_pull.sh` | `/usr/local/bin/` | Check + deploy CRM API | Cron |
| `update_crm_front.sh` | `/usr/local/bin/` | Check CRM frontend updates | Cron |
| `crm_front_build.sh` | `/usr/local/bin/` | Build CRM frontend | Update checker |
| `check_kaspi_api_updates.sh` | `/usr/local/bin/` | Check tracker API updates | Cron |
| `deploy_tracker_api.sh` | `/usr/local/bin/` | Deploy tracker API | Update checker |
| `check_kaspi_app_updates.sh` | `/usr/local/bin/` | Check tracker frontend updates | Cron |
| `build_tracker_front.sh` | `/usr/local/bin/` | Build tracker frontend | Update checker |
| `check_ah_updates.sh` | `/usr/local/bin/` | Check AH API updates | Cron |
| `deploy_ah.sh` | `/usr/local/bin/` | Deploy AH API | Update checker |
| `check_otaustore_updates.sh` | `/usr/local/bin/` | Check otaustore updates | Cron |
| `deploy_otaustore.sh` | `/usr/local/bin/` | Deploy otaustore | Update checker |
| `cron_update_checker_bomb_royale_prod.sh` | `/usr/local/bin/` | Check bomb-royale updates | Cron |
| `deploy_bomb_royale_prod.sh` | `/usr/local/bin/` | Deploy bomb-royale | Update checker |
| `cron_update_checker_gavrilov_prod.sh` | `/usr/local/bin/` | Check gavrilov.kz updates | Cron |
| `deploy_gavrilov_prod.sh` | `/usr/local/bin/` | Deploy gavrilov.kz | Update checker |
| `db-backup-rotate` | `/usr/local/bin/` | Daily DB backup with rotation | Cron (3 AM) |
| `db_backup.sh` | `/usr/local/bin/` | Manual DB backup (legacy) | Manual |
| `site_monitor.sh` | `/usr/local/bin/` | Site health monitoring | Cron (every min) |

### Script Templates (for new projects)

#### Update Checker Template

```bash
#!/bin/bash
REPO_DIR="/var/www/_repos/<repo-name>"
BRANCH="master"
LOG_FILE="/var/log/update_<app>.log"

cd "$REPO_DIR" || exit 1
git fetch origin

LOCAL=$(git rev-parse HEAD)
REMOTE=$(git rev-parse origin/$BRANCH)

if [ "$LOCAL" != "$REMOTE" ]; then
    echo "[$(date)] New commits detected, deploying..." >> "$LOG_FILE"
    /usr/local/bin/deploy_<app>.sh >> "$LOG_FILE" 2>&1
else
    echo "[$(date)] No changes" >> "$LOG_FILE"
fi
```

#### Deploy Template (Node.js)

```bash
#!/bin/bash
set -e

REPO_DIR="/var/www/_repos/<repo-name>"
APP_DIR="/var/www/_projects/<app-name>/app"
COMPOSE_DIR="/var/www/_projects/<app-name>"
CONFIG_SOURCE="/var/www/_deploy/configs/<app>_app.env"
BRANCH="master"

echo "=========================================="
echo "[$(date)] Starting deployment..."
echo "=========================================="

cd "$REPO_DIR"
git fetch origin
git reset --hard origin/$BRANCH

echo "Checking out files to $APP_DIR..."
git --work-tree="$APP_DIR" checkout -f $BRANCH

if [ -f "$CONFIG_SOURCE" ]; then
    echo "Copying environment config..."
    cp "$CONFIG_SOURCE" "$APP_DIR/.env"
fi

cd "$COMPOSE_DIR"
echo "Rebuilding container..."
docker-compose down
docker-compose up -d --build

echo "=========================================="
echo "[$(date)] Deployment complete!"
echo "=========================================="
docker ps | grep <app>
```

---

## 13. Configuration Files Reference

### Environment Config Files

| File | App | Type | Key Settings |
|------|-----|------|-------------|
| `configs/prod_bot_app_local.php` | bot | CakePHP | DB: bot_db, host: bot_db |
| `configs/prod_crm_app_local.php` | crm API | CakePHP | DB: crm_db, host: crm_db, Telegram, Wazzup, CORS, WebSocket |
| `configs/crm.gavrilov.kz.env` | crm-front | Vite env | API URL, Telegram bot, WebSocket URL |
| `configs/tracker_app_local.php` | tracker API | CakePHP | DB: tracker_db, host: tracker_db, Telegram |
| `configs/tracker_app.env` | tracker-front | Vite env | API URL, Telegram auth |
| `_deploy/configs/ah_app.env` | ah API | Node.js env | PORT=3000, API_TOKEN |
| `_deploy/configs/pta.gavrilov.kz.env` | (unused?) | Node.js env | PORT=3000, ANTHROPIC_KEY |

### CakePHP Config Structure (`app_local.php`)

All PHP apps follow this structure:
```php
return [
    'debug' => false,
    'Security' => ['salt' => '<hash>'],
    'Datasources' => [
        'default' => [
            'host' => '<docker_service_name>',  // e.g., 'crm_db'
            'port' => '3306',
            'username' => '<db_user>',
            'password' => 'mypassword',
            'database' => '<db_name>',
        ],
    ],
    // App-specific: Telegram, CORS, URLs, Wazzup, etc.
];
```

### Frontend .env Structure

**CRM Frontend** (`crm.gavrilov.kz.env`):
```
VITE_TELEGRAM_BOT_USERNAME=mini_crm_testing_bot
VITE_TELEGRAM_REDIRECT_URL=https://api.gavrilov.kz/users/telegram-login-callback
VITE_API_BASE_URL=https://api.gavrilov.kz
PORT=300
VITE_WS_URL=https://api.gavrilov.kz/websocket
```

**Tracker Frontend** (`tracker_app.env`):
```
VITE_API_BASE_URL=/api
VITE_BYPASS_TELEGRAM_AUTH=false
```

---

## 14. Cron Jobs

### Active Crontab

```cron
# -=[ crm.gavrilov.kz ][==-
*/1 * * * * /usr/local/bin/update_crm_front.sh >> /var/log/crm_github_pull.log 2>&1

# -=[ bot.gavrilov.kz ]==-
*/1 * * * * /usr/local/bin/check_for_updates_bot.sh >> /var/log/update_bot.log 2>&1

# -=[ tracker.gavrilov.kz ]=-
*/1 * * * * /usr/local/bin/check_kaspi_api_updates.sh >> /var/log/update_tracker_api.log 2>&1

# -=[ tracker.gavrilov.kz front end app ]=-
*/1 * * * * /usr/local/bin/check_kaspi_app_updates.sh >> /var/log/tracker_app_updates.log 2>&1

# -=[ bomb royale ]=-
*/1 * * * * /usr/local/bin/cron_update_checker_bomb_royale_prod.sh >> /var/log/update_bomb_royale.log 2>&1

# -=[ gavrilov.kz ]=-
*/1 * * * * /usr/local/bin/cron_update_checker_gavrilov_prod.sh >> /var/log/update_gavrilov.log 2>&1

# -=[ ah.gavrilov.kz ]=-
*/1 * * * * /usr/local/bin/check_ah_updates.sh >> /var/log/update_ah.log 2>&1

# -=[ otaustore.gavrilov.kz ]=-
*/1 * * * * /usr/local/bin/check_otaustore_updates.sh >> /var/log/update_otaustore.log 2>&1

# -=[ Database backup ]=-
0 3 * * * /usr/local/bin/db-backup-rotate >> /var/log/db_backup.log 2>&1

# -=[ monitoring ]=-
*/1 * * * * /usr/local/bin/site_monitor.sh
```

### Log Files

| Log | Purpose |
|-----|---------|
| `/var/log/update_bot.log` | Bot deployment |
| `/var/log/api_github_pull.log` | CRM API deployment |
| `/var/log/crm_github_pull.log` | CRM frontend deployment |
| `/var/log/update_tracker_api.log` | Tracker API deployment |
| `/var/log/tracker_app_updates.log` | Tracker frontend deployment |
| `/var/log/update_ah.log` | AH API deployment |
| `/var/log/update_otaustore.log` | Otaustore deployment |
| `/var/log/update_bomb_royale.log` | Bomb Royale deployment |
| `/var/log/update_gavrilov.log` | gavrilov.kz deployment |
| `/var/log/db_backup.log` | Database backup |
| `/var/log/nginx/*.access.log` | Nginx access (per domain) |
| `/var/log/nginx/error.log` | Nginx errors (global) |

---

## 15. Security Notes

### Sudo Workflow

Claude creates drafts in `/var/www/_temp/`. User deploys with sudo:
```
_temp/ (staging) --sudo cp--> /usr/local/bin/ (scripts)
_temp/ (staging) --sudo cp--> /etc/nginx/ (nginx configs)
```

### Database Passwords

All databases currently use `mypassword` as the password.

### SSH Keys

GitHub deployment uses SSH keys. Verify: `ssh -T git@github.com`

### File Permissions

- Scripts in `/usr/local/bin/` should be 755 (executable)
- Application directories owned by user `yg`
- Docker runs as user `yg`
- Nginx system process runs as `www-data`

### Sensitive Files

Config files containing credentials are stored in:
- `/var/www/configs/` (legacy)
- `/var/www/_deploy/configs/` (new)

These should NOT be committed to git repositories.

---

## Appendix: Technology Stack Summary

| Component | Technology | Version/Details |
|-----------|-----------|-----------------|
| OS | Ubuntu Linux | 5.4.0-167-generic |
| Web Server | Nginx | System-level + Docker containers |
| PHP | PHP-FPM | Latest via Docker |
| Framework | CakePHP | PHP backend framework |
| Database | MariaDB | 10.6 |
| Task Scheduler | Ofelia | Docker-native cron |
| Frontend Build | Vite | Vue.js / React |
| Node.js | Node.js | 20 (Alpine) |
| Containerization | Docker + Docker Compose | System-installed |
| SSL | Let's Encrypt | Certbot auto-renewal |
| Version Control | Git | GitHub remotes |
| CI/CD | Cron + Shell Scripts | Every-minute polling |
| Monitoring | Custom bash script | Telegram alerts |
| Backup | Custom bash script | Daily with rotation |

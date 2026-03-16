---
name: vps-ssh-setup
description: One-time setup to give the devops agent SSH + scoped sudo access to a VPS. Follow once per VPS, then record connection details in PROJECT_CONTEXT.md.
---

## What this produces

- An SSH key pair the agent uses to connect
- A sudoers drop-in that allows passwordless execution of deployment operations only
- A verified, working connection recorded in `PROJECT_CONTEXT.md`

---

## Step 1 — Generate SSH key (local machine)

Use a dedicated key — do not reuse your personal key:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/claude_devops -C "claude-devops"
```

---

## Step 2 — Authorize on the VPS

This adds the Claude key to your **existing** VPS user's `authorized_keys` — no new user needed.

```bash
ssh-copy-id -i ~/.ssh/claude_devops.pub <user>@<vps-ip>
```

Verify (should return immediately, no password):

```bash
ssh -i ~/.ssh/claude_devops <user>@<vps-ip> echo OK
```

---

## Step 3 — Sudoers rule on the VPS

Create a drop-in (safer than editing sudoers directly):

```bash
sudo visudo -f /etc/sudoers.d/claude-devops
```

Paste — replace `<user>` with the SSH user:

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

Verify (no password prompt expected):

```bash
sudo nginx -t
```

---

## Step 4 — Record in PROJECT_CONTEXT.md

Add one line to the **Infra** section:

```
VPS: <user>@<ip> | key: ~/.ssh/claude_devops | workdir: /var/www/
```

---

## Rules

- Never commit the private key
- One key per VPS — all projects on that VPS share it
- Extend the sudoers list only when a new operation type is genuinely needed

---
name: workflow-vps-ssh-setup
type: workflow
description: One-time setup to give the devops agent SSH + scoped sudo access to a VPS. Follow once per VPS, then record connection details in PROJECT_CONTEXT.md.
---

## What this produces

- An SSH key pair the agent uses to connect
- A sudoers drop-in that allows passwordless execution of deployment operations only
- A verified, working connection recorded in `PROJECT_CONTEXT.md`

## Steps (summary)

1. **Generate SSH key** (local machine) — dedicated ed25519 key, not personal key
2. **Authorize on the VPS** — `ssh-copy-id` to existing user's `authorized_keys`
3. **Sudoers rule on the VPS** — scoped drop-in granting passwordless access to deploy/nginx/certbot operations only
4. **Record in PROJECT_CONTEXT.md** — `VPS: <user>@<ip> | key: ~/.ssh/claude_devops | workdir: /var/www/`

## Rules

- Never commit the private key
- One key per VPS — all projects on that VPS share it
- Extend the sudoers list only when a new operation type is genuinely needed

For full step instructions with commands, see `./SKILL-DETAILS.md`.

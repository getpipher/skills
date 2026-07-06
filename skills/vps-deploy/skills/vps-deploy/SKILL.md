---
name: vps-deploy
description: Use when setting up new VPS projects, auditing VPS health, reviewing deploy workflows, debugging deployment issues, or hardening server security. Covers Docker, nginx, SSL, SSH, and CI/CD patterns for shared VPS infrastructure.
user-invocable: true
---

# VPS Deployment & Operations

## Infrastructure

**SSH**: Always use `~/.ssh/config` host aliases, never raw IPs. See config for current VPS details.
**Domains**: Port registry at `~/.ssh/vps-port-registry.md`.
**Account isolation**: 1 project = 1 Linux user.

## Deploy Pattern

All projects use Docker. No exceptions.

```
GitHub push → Actions build → GHCR image → SSH to VPS → docker compose pull → up -d → image prune -f
```

## Deploy Style

Before setting up deployment, ask which style is needed:

| Style | Description | Use When |
|-------|-------------|----------|
| **Simple** | Single container, `docker compose up -d` replaces in-place | Most projects. Brief downtime (~2-5s) acceptable. |
| **Blue-Green** | Two containers (blue/green), nginx switches upstream after health check | Zero-downtime needed. Currently: pNode Pulse only. |

Default recommendation: **Simple** (less complexity, sufficient for most apps).

## New Project Checklist

1. Create Linux user on VPS (`adduser <project>`)
2. Add to docker group: `sudo usermod -aG docker <project>`
3. Setup SSH access for new user:
   - `sudo mkdir -p /home/<project>/.ssh`
   - Copy the shared deploy key: `sudo cp /root/.ssh/authorized_keys /home/<project>/.ssh/` (or append the key from `~/.ssh/id_ed25519.pub`)
   - `sudo chown -R <project>:<project> /home/<project>/.ssh && sudo chmod 700 /home/<project>/.ssh && sudo chmod 600 /home/<project>/.ssh/authorized_keys`
4. Add host entry to local `~/.ssh/config`
5. Reserve port in `~/.ssh/vps-port-registry.md`
6. `docker-compose.yml` with `name: <project>` at top level
7. GitHub repo secrets (`VPS_SSH_KEY`, `VPS_HOST`, `VPS_USER`, `VPS_APP_PATH`). `VPS_SSH_KEY` = private key matching the public key in step 3.
8. **Create DNS record** at registrar (A record → VPS IP, or CNAME → existing domain), then verify: `dig <domain> +short` must return VPS IP before proceeding
9. nginx reverse proxy + `certbot --nginx -d <domain>` (certbot fails if DNS doesn't resolve yet)
10. **Verify** deploy.yml uses `appleboy/ssh-action` (never raw SSH heredocs — tilde/variable expansion breaks in GitHub Actions)
11. **Verify** deploy.yml includes `docker image prune -f` (not `docker system prune`)
12. **Verify** DB/cache ports bind to `127.0.0.1`
13. **Verify** SSH password auth is disabled (including drop-in overrides)
14. **Verify** UFW allows only 22, 80, 443
15. **If root domain**: Add www DNS CNAME at registrar + www → non-www 301 redirect in nginx

## Docker Rules

### Image Cleanup
- Every deploy replaces the running image, leaving the old one as `<none>:<none>` (dangling)
- Deploy workflows MUST run `docker image prune -f` after `docker compose up -d`
- NEVER use `docker system prune` on shared VPS (removes other projects' stopped containers, networks)
- Weekly cron (`/etc/cron.weekly/docker-cleanup`) catches stragglers

### Port Binding Security
- DB/cache ports MUST bind to localhost: `"127.0.0.1:port:port"`
- Containers connect via internal Docker bridge network, not host ports
- Only web services (behind nginx) bind to `0.0.0.0`

**Audit command:**
```bash
ssh <vps> "docker ps --format '{{.Names}}\t{{.Ports}}' | grep -E '(5432|6379|3306|27017)'"
```
Any result showing `0.0.0.0` on a DB/cache port = vulnerability.

### Compose File
- Always set `name: <project>` at top level
- Use named volumes with explicit `name:` to prevent prefix conflicts
- Use explicit named networks

## GHCR Authentication

Org packages on GHCR default to private even if the repo is public. VPS users need docker auth to pull.

**Setup**: Copy existing auth from another project user:
```bash
sudo cp /home/<existing-user>/.docker/config.json /home/<new-user>/.docker/config.json
sudo chown <new-user>:<new-user> /home/<new-user>/.docker/config.json
```

Or login directly: `docker login ghcr.io -u <github-user>`

## SSH Hardening

- Key-only auth enforced. Password auth MUST be disabled.
- **Cloud-init trap**: `/etc/ssh/sshd_config.d/50-cloud-init.conf` silently re-enables `PasswordAuthentication yes` after VPS rebuild/reimage. Drop-in files override main config.

**Verify effective config:**
```bash
ssh <vps> "sudo sshd -T | grep passwordauthentication"
# Must show: passwordauthentication no
```

**After any VPS rebuild/reimage**: Re-check SSH config immediately.

## nginx + SSL

- One site config per service in `/etc/nginx/sites-available/`, symlinked to `sites-enabled/`
- SSL via `certbot --nginx -d <domain>`
- Let's Encrypt auto-renewal (certbot timer)

### www → non-www Redirect (Root Domains Only)

Root domains MUST have www → non-www 301 redirects for SEO. Subdomains (docs.*, pulse.*) don't need this.

**Requirements:**
1. DNS: CNAME `www` → apex domain at registrar
2. SSL cert must cover both (`certbot --nginx -d example.com -d www.example.com`)
3. Separate nginx server block for the redirect

**nginx pattern** (3 server blocks per root domain):
```nginx
# 1. HTTP → HTTPS (both www and non-www)
server {
    listen 80;
    server_name example.com www.example.com;
    location /.well-known/acme-challenge/ { root /var/www/html; }
    location / { return 301 https://example.com$request_uri; }
}

# 2. HTTPS www → non-www redirect
server {
    listen 443 ssl;
    server_name www.example.com;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    return 301 https://example.com$request_uri;
}

# 3. HTTPS main (non-www only)
server {
    listen 443 ssl;
    server_name example.com;
    # ... ssl + proxy config ...
}
```

## GitHub Actions Deploy Workflow Template

Always use `appleboy/ssh-action`. Never use raw SSH heredocs (`ssh user@host << 'EOF'`) — variable/tilde expansion breaks unpredictably in GitHub Actions.

```yaml
- name: Deploy to VPS
  uses: appleboy/ssh-action@v1.0.3
  with:
    host: ${{ secrets.VPS_HOST }}
    username: ${{ secrets.VPS_USER }}
    key: ${{ secrets.VPS_SSH_KEY }}
    script: |
      cd ${{ secrets.VPS_APP_PATH }}
      docker compose pull
      docker compose up -d
      docker image prune -f
```

**Secret naming pitfall**: If `VPS_USER` value is a common word (e.g. "core"), GitHub masks ALL occurrences in logs. Cosmetic only — commands still execute correctly. But avoid relying on log output for debugging.

## Health Check Commands

```bash
# Full health check
ssh <vps> "uptime && df -h / && free -h && docker ps --format 'table {{.Names}}\t{{.Status}}'"

# Docker disk usage
ssh <vps> "docker system df"

# Check for zombie processes
ssh <vps> "ps aux | awk '\$8 ~ /Z/ {print}'"

# Firewall status
ssh <vps> "sudo ufw status"

# Fail2ban status
ssh <vps> "sudo fail2ban-client status"
```

## Security Audit Checklist

- [ ] SSH: `passwordauthentication no` (effective, including drop-ins)
- [ ] SSH: `PermitRootLogin prohibit-password`
- [ ] UFW: Only 22, 80, 443 open
- [ ] Fail2ban: sshd jail active
- [ ] Docker: No DB/cache ports on `0.0.0.0`
- [ ] Docker: All deploy workflows use `docker image prune -f` (not `system prune`)
- [ ] SSL: All certs valid, auto-renewal working
- [ ] SSL: No orphaned certs (dead projects cleaned up)
- [ ] nginx: Config test passes
- [ ] nginx: Root domains have www → non-www redirect
- [ ] Authorized keys: Each project user has the GitHub Actions deploy key

## Lessons Learned

- `docker system prune -f` on shared VPS nukes other projects' resources. Always `docker image prune -f`.
- Cloud-init drop-in (`/etc/ssh/sshd_config.d/50-cloud-init.conf`) silently re-enables SSH password auth after VPS rebuild. Always check effective config with `sshd -T`.
- DB/cache ports default to `0.0.0.0` in many docker-compose examples. Always explicitly bind `127.0.0.1`.
- Raw SSH heredocs in GitHub Actions cause unpredictable shell expansion (`~` → `/tmp`). Always use `appleboy/ssh-action`.
- `www.<domain>` needs both a DNS record AND nginx redirect. Missing either = broken or duplicate content.
- Certbot fails silently if DNS doesn't resolve to the VPS yet. Always `dig <domain>` first.
- GHCR org packages default to private. VPS users need `~/.docker/config.json` with auth to pull images.
- `docker/metadata-action@v5` auto-lowercases image tags — use it when org names have uppercase (e.g. ACME-Labs).

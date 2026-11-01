# Self-host n8n: Docker Compose kit

Caddy (automatic HTTPS) + n8n + PostgreSQL on one small VPS. Three containers, one command.
▶️ Video walkthrough: https://www.youtube.com/watch?v=jn7h7wNESLw · Channel: https://www.youtube.com/@BlueprintSystemsAI

| File | What it does |
|------|--------------|
| `docker-compose.yml` | The three services. Only ports 80/443 are public. |
| `Caddyfile` | Reverse proxy + free Let's Encrypt certificate for your domain. |
| `.env.example` | The only file you edit. Copy it to `.env`. |

Pinned versions: n8n `2.43.7`, PostgreSQL `17`, Caddy `2`.

## Requirements
- A Linux VPS (Ubuntu 24.04 recommended) with Docker + Docker Compose v2.
  Idle footprint of the whole stack is roughly 0.4 GB of RAM; give it 2 GB or more for real workflows.
- A (sub)domain, e.g. `n8n.yourdomain.com`, with an **A record pointing to the VPS IP**.
  Check from the server with `getent hosts n8n.yourdomain.com` (or `nslookup` from your computer) before you start.
- Ports 80 and 443 reachable from the internet (VPS firewall and `ufw`).

## Setup
```bash
# 1. Firewall (skip if you manage it elsewhere)
sudo ufw allow OpenSSH && sudo ufw allow 80/tcp && sudo ufw allow 443 && sudo ufw --force enable

# 2. Docker, from Docker's apt repository (Ubuntu 24.04). Docker doesn't recommend its
#    convenience script for production. Adapted from https://docs.docker.com/engine/install/ubuntu/
#    Paste ONE line at a time (long multi-line pastes can get scrambled in some terminals).
sudo apt update && sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
docker compose version   # should print v2.x or newer

# 3. Put the three kit files in a folder
mkdir -p ~/n8n && cd ~/n8n
# ...copy docker-compose.yml, Caddyfile and .env.example here...

# 4. Configure: copy the template, generate the two secrets, then set your domain and email
cp .env.example .env
sed -i "s|^POSTGRES_PASSWORD=.*|POSTGRES_PASSWORD=$(openssl rand -hex 24)|" .env
sed -i "s|^N8N_ENCRYPTION_KEY=.*|N8N_ENCRYPTION_KEY=$(openssl rand -hex 32)|" .env
nano .env   # set N8N_FQDN and SSL_EMAIL (and GENERIC_TIMEZONE)

# 5. Start
docker compose up -d
docker compose logs -f caddy   # wait for "certificate obtained successfully", then Ctrl+C
```
Open `https://n8n.yourdomain.com`, create the owner account, done.

> **Back up `.env`.** `N8N_ENCRYPTION_KEY` encrypts every credential you save in n8n. Lose it and they are unrecoverable.

## Update n8n
1. Pick the new version from https://hub.docker.com/r/n8nio/n8n/tags (use the plain `X.Y.Z` tags) and read the release notes.
2. Back up first (below). Database migrations can't be undone.
3. Set `N8N_VERSION=X.Y.Z` in `.env`, then:
```bash
docker compose pull && docker compose up -d
```
Do not change the `postgres:17-alpine` tag to a different major version. That needs a dump and restore.

## Quick backup
```bash
docker compose exec -T postgres sh -c 'pg_dump -U "$POSTGRES_USER" "$POSTGRES_DB"' | gzip > n8n-db-$(date +%F).sql.gz
cp .env env-backup-$(date +%F)   # keep both files OFF the server
```

## Stop / restart / remove
```bash
docker compose stop            # stop, keep everything
docker compose down            # remove containers, KEEP data volumes
docker compose down -v         # also DELETES all data. Irreversible.
```

## Troubleshooting
| Symptom | Likely cause |
|---------|--------------|
| `required variable ... is missing a value` | `.env` is missing or a variable is empty. The message names which one. |
| Caddy never reports a certificate | DNS A record not pointing at this VPS yet, or ports 80/443 blocked. Behind Cloudflare, set the record to **DNS only** (grey cloud). |
| Caddy log says `no valid A records found for <name>` | `N8N_FQDN` in `.env` is not the hostname you created in DNS (check for a typo or a missing prefix). Run `grep N8N_FQDN .env` and compare it with the record; fix `.env`, then `docker compose up -d`. |
| Webhook or OAuth URLs look wrong | Check `N8N_FQDN` in `.env`, and that `N8N_WEBHOOK_URL` / `N8N_EDITOR_BASE_URL` are still set in the compose file. |
| `n8n's address '::' is not available` | The host kernel has IPv6 disabled. Add `N8N_LISTEN_ADDRESS: 0.0.0.0` under n8n's `environment:`. |
| Mismatching encryption keys | `N8N_ENCRYPTION_KEY` in `.env` differs from the one first used. Restore the original value. |

## Optional hardening (recommended before you store real credentials)
- Create a non-root user: `sudo adduser deploy && sudo usermod -aG sudo,docker deploy`, then use it instead of `root`.
  (Members of the `docker` group have root-equivalent access. Add only people you trust.)
- Log in with SSH keys, then set `PasswordAuthentication no` in `/etc/ssh/sshd_config` and `sudo systemctl restart ssh`.
  Test a second SSH session before closing the first one.
- Turn on automatic security updates: `sudo apt install unattended-upgrades`.

## Things you will see in the logs (normal)
- Caddy: two `warn` lines saying HTTP/2 and HTTP/3 are skipped on port 80. That port only redirects to HTTPS.
- Caddy: an `info` line saying it "failed to sufficiently increase receive buffer size" (UDP buffer, for HTTP/3). It is harmless for this setup.
- Caddy, first start only: lines about "creating new account because no account for configured email is known". Caddy is registering your Let's Encrypt account.
- n8n: a deprecation notice for the internal task runner mode. It is n8n's default today. Because the version is
  pinned, nothing changes until you update. Check the release notes when you do.

## Not covered here
Queue mode / workers, SSO, offsite backup automation, and moving from SQLite. Those get their own videos.

## License note
Self-hosted n8n runs under n8n's Sustainable Use License. Free for running your own business; you may build automations
for clients on your instance as long as they can't create or edit workflows. Hosting n8n as a service where clients
build their own workflows is not allowed. Read the official FAQ for your case:
https://docs.n8n.io/n8n-community-license/community-license/license-faq

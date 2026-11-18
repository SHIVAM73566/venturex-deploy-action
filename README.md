# VentureX ERP & CRM — GitHub Action (v1.3.1)

Deploy **VentureX ERP & CRM v1.3.1** to your own Ubuntu 22.04 or 24.04 server in one workflow run. This composite action provisions PHP 8.4, MySQL 8, Nginx, Composer, Node.js 22, the application, a queue worker, the Laravel scheduler and optional HTTPS via Let's Encrypt.

[![GitHub Action](https://img.shields.io/badge/GitHub-Action-blue)](https://github.com/features/actions)
![Version](https://img.shields.io/badge/Version-1.3.1-blue)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20%7C%2024.04-orange)
![PHP](https://img.shields.io/badge/PHP-8.4-purple)
![MySQL](https://img.shields.io/badge/MySQL-8-blue)
![License](https://img.shields.io/badge/License-MIT-green)

> **Marketplace status:** GitHub Marketplace approval is **pending**. This repository is a standard GitHub Action; no Marketplace availability badge is claimed.

> **What it is:** one self-hosted Laravel 13 application combining CRM, ERP, procurement, inventory, finance, logistics, HR, support and 84 enterprise RAG pipelines. The action installs the infrastructure and application, runs migrations, installs a queue worker and scheduler, and leaves first-administrator creation to the one-time `/install` wizard. It never creates a known default password.

---

## Platform highlights

- **HR suite** — employees, attendance, leave, payroll, performance reviews and onboarding.
- **AERO X Agentic Engine** — auditable, transactional automation rules with rollback on failure.
- **84 RAG pipelines** across CRM, procurement, inventory, finance, logistics, executive and HR domains, with sources, confidence and latency.
- **Key-only AI onboarding** — NVIDIA NIM, Google Gemini, OpenAI or Anthropic Claude keys are configured inside the app.
- **AERO X Super Intelligence Brain** — encrypted credentials, prompt/PII sanitisation, threat detection, rate limiting and audit telemetry.
- **Invoice branding** — logo, accent colour, PDF typography, tax IDs, footer, terms and watermark.

---

## What this action does

| Step | Result |
|---|---|
| Installs dependencies | PHP **8.4** + required extensions, MySQL **8**, Nginx, Node.js 22, Composer |
| Creates the database | MySQL database and user from GitHub Secrets |
| Deploys the app | Idempotently checks out `VentureX-ERP-CRM` (fetch/reset or shallow clone); installs Composer/npm packages and builds assets |
| Configures the environment | Writes/escapes `.env` values without interpolating secrets into shell command lines; generates `APP_KEY` if empty; sets `APP_ENV`, `APP_DEBUG=false`, `APP_URL` and DB/cache/session/queue values |
| Prepares the database | Runs `php artisan migrate --force`; optional demo data seeding is disabled by default and never creates login accounts |
| Optimises | Storage link and `php artisan optimize` |
| Serves it | Nginx virtual host with `public/` as document root, 100 MB upload limit and optional Let's Encrypt HTTPS |
| Runs background work | Installs a `systemd` **queue worker** and a cron entry for `php artisan schedule:run` |
| Verifies | Checks Nginx, PHP-FPM and MySQL are active, then performs a loopback `/up` health check |

The web root is always the application's `public/` directory. The root `.htaccess` in the source repository is only a fallback and is not needed with this action's Nginx configuration.

---

## Prerequisites

- Ubuntu **22.04 or 24.04**
- A domain pointed to the server IP
- SSH access (root, or a user with passwordless `sudo -n`)
- Ports 80 and 443 open
- Only Debian/Ubuntu with `apt-get` is supported

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `server-host` | Yes | — | Server hostname or IP address. |
| `server-user` | No | `root` | SSH username; must have passwordless sudo if not root. |
| `server-key` | Yes | — | SSH private key. Store in GitHub Secrets. |
| `server-port` | No | `22` | SSH port. |
| `domain` | Yes | — | Domain pointing at the server. |
| `ssl-email` | No | `''` | Let's Encrypt email. Empty skips TLS. |
| `db-name` | No | `venturex_erp` | MySQL 8 database name. |
| `db-user` | No | `venturex_user` | MySQL 8 database user. |
| `db-password` | Yes | — | MySQL password. Store in GitHub Secrets. |
| `install-path` | No | `/var/www/venturex` | Parent directory for the checkout. |
| `php-version` | No | `8.4` | PHP version; must be 8.4 or newer. |
| `app-env` | No | `production` | Laravel `APP_ENV`. |
| `run-seeders` | No | `false` | Optional demo data. Never creates login accounts. |
| `known-hosts` | No | `''` | Pinned SSH host key. Empty uses `ssh-keyscan` with an explicit MITM warning. |

---

## Quick start

Add this to `.github/workflows/deploy.yml`:

```yaml
name: Deploy VentureX ERP

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy VentureX ERP v1.3.1
        uses: SHIVAM73566/venturex-deploy-action@v1.3.1
        with:
          server-host: ${{ secrets.SERVER_HOST }}
          server-user: ${{ secrets.SERVER_USER }}
          server-key: ${{ secrets.SERVER_KEY }}
          server-port: '22'
          domain: ${{ secrets.DOMAIN }}
          ssl-email: ${{ secrets.SSL_EMAIL }}
          db-name: venturex_erp
          db-user: venturex_user
          db-password: ${{ secrets.DB_PASSWORD }}
          install-path: /var/www/venturex
          php-version: '8.4'
          app-env: production
          run-seeders: 'false'
          known-hosts: ${{ secrets.KNOWN_HOSTS }}
```

> **Tag status:** the `@v1.3.1` tag is pending creation during central integration. Until it exists, pin the action to the full verified commit SHA:
> `uses: SHIVAM73566/venturex-deploy-action@<full-commit-sha>`
> Do not use `@main` or a moving branch for production.

### Examples

**Fresh VPS**

```yaml
      - uses: SHIVAM73566/venturex-deploy-action@v1.3.1
        with:
          server-host: ${{ secrets.DO_SERVER_IP }}
          server-user: root
          server-key: ${{ secrets.DO_SSH_KEY }}
          domain: erp.example.com
          db-password: ${{ secrets.DB_PASSWORD }}
```

**Custom SSH port and non-root user**

```yaml
      - uses: SHIVAM73566/venturex-deploy-action@v1.3.1
        with:
          server-host: erp.example.com
          server-user: deploy
          server-key: ${{ secrets.SERVER_KEY }}
          server-port: '2222'
          domain: erp.example.com
          db-password: ${{ secrets.DB_PASSWORD }}
          known-hosts: ${{ secrets.KNOWN_HOSTS }}
```

---

## Installer mode behavior

The action prepares infrastructure and runs migrations, but intentionally creates **no administrator account**. On the first visit:

1. Open `https://<domain>/install` (or `http://` when `ssl-email` is empty).
2. Choose **Clean Production Mode** — recommended. It uses the already-migrated schema and creates only the administrator you specify; no demo users or sample records are added by the installer.
3. If you deliberately set `run-seeders: 'true'` for a disposable evaluation, sample records may exist, but there are still no known demo login credentials. Prefer `false` for customer and production installs.
4. The installer writes `storage/.installed` and is then disabled. It never publishes a default password.

The application is checked out to `<install-path>/venturex-erp` (for example `/var/www/venturex/venturex-erp`). Re-running the workflow updates that checkout and preserves `.env`.

---

## Security notes

- Store SSH keys, database passwords and host keys in GitHub Secrets. Never commit them.
- Prefer `known-hosts` pinning; without it the action warns that `ssh-keyscan` does not protect the first connection against MITM.
- Pin to a full commit SHA for reproducibility. The v1.3.1 tag is pending central integration.
- Secrets are base64-encoded and applied by a server-side PHP helper; they are not interpolated into shell command lines or printed after deployment.
- The installer creates the first administrator over HTTPS; do not share or commit the chosen password.

---

## Setup GitHub Secrets

| Secret | Value |
|---|---|
| `SERVER_HOST` | Server IP or hostname |
| `SERVER_USER` | `root` or a passwordless-sudo user |
| `SERVER_KEY` | Full SSH private key including header/footer |
| `DOMAIN` | e.g. `erp.example.com` |
| `SSL_EMAIL` | Let's Encrypt email (optional) |
| `DB_PASSWORD` | MySQL 8 password |
| `KNOWN_HOSTS` | Optional pinned host key line(s) |

---

## Verification and troubleshooting

- Verifies `nginx`, `mysql` and `php*-fpm` are active.
- Performs a loopback `/up` health check with retries; server-side DNS is not required.
- If assets fail, confirm Node.js 22 is available on the runner.
- If migrations fail, check `storage/logs/laravel.log` on the server.
- If a non-root user cannot sudo, grant passwordless `sudo -n`.
- Without `ssl-email`, the site is served over HTTP only.

| Symptom | Fix |
|---|---|
| `Permission denied (publickey)` | Check `SERVER_KEY` includes the full private key. |
| Nginx 502 | Confirm PHP 8.4 FPM is installed and running. |
| Blank page | Check `storage/logs/laravel.log` and permissions on `storage` / `bootstrap/cache`. |
| Certificate failed | Confirm DNS points to the server, then re-run. |
| Queue jobs not running | Check `systemctl status venturex-queue.service`. |

## What this action does not do

- Docker or shared-hosting/cPanel deployment
- Managed hosting or guaranteed outcomes
- GitHub Marketplace approval (pending)

## Support

- Product documentation: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/tree/main/documentation>
- Security report: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/blob/main/SECURITY-AUDIT-REPORT.md>
- Issues: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/issues>
- Maintainer: **Shivam Chaturvedi** — chaturvedishivam179@gmail.com
- PayPal checkout / sales: chaturvedishivam179@gmail.com

## License

MIT License — see [LICENSE](LICENSE). Copyright © 2026 Shivam Chaturvedi.

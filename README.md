# VentureX ERP & CRM — GitHub Action

Deploy **VentureX ERP & CRM** — an AI-powered business operating system — to your own Ubuntu server in one workflow run. This composite action provisions PHP 8.4, MySQL, Nginx, Node.js 22, configures Laravel (including `.env`, migrations, caching, storage linking), installs a queue worker and scheduler, and can optionally enable HTTPS with Let's Encrypt.

[![GitHub Marketplace](https://img.shields.io/badge/Available%20on-GitHub%20Marketplace-blue)](https://github.com/marketplace/actions/venturex-erp-crm-deploy)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

> **What it is:** one Laravel application that combines CRM, ERP, procurement, inventory, finance, logistics, HR and 84 enterprise RAG (retrieval-augmented generation) pipelines. This action installs it end-to-end: PHP, MySQL, Nginx, Composer, Node.js, migrations, caching and optional HTTPS via Let's Encrypt. It never creates a login account — you create the first administrator through the one-time installer.

---

## Platform Highlights

The action deploys the full VentureX ERP & CRM platform, including:

- **HR suite** - employee directory, attendance, leave, payroll runs, performance
  reviews and automated onboarding.
- **AERO X Agentic Engine** - autonomous, auditable rules that draft purchase
  orders on low stock, flag overdue invoices, self-assign support tickets and
  seed onboarding checklists. Every run is transactional and rolled back on
  failure.
- **84 RAG pipelines** across seven domains (CRM, procurement, inventory,
  finance, logistics, executive and HR) with sources, confidence and latency.
- **Key-only AI onboarding** - paste a key for NVIDIA NIM, Google Gemini, OpenAI
  or Anthropic Claude; endpoints and models are wired server-side.
- **AERO X Super Intelligence Brain** - AES-256 encrypted credentials, prompt/PII
  sanitisation, threat detection, rate limiting and audit telemetry.
- **Invoice branding** - upload a logo and set accent colour, PDF typography,
  tax/registration IDs, footer, terms and watermark from Settings.

---

## What this action does

| Step | Result |
|---|---|
| Installs dependencies | PHP 8.4 + required extensions, MySQL, Nginx, Node.js 22, Composer |
| Creates the database | MySQL database and user with the credentials you provide |
| Deploys the app | Idempotently checks out `VentureX-ERP-CRM` (fetch/reset or shallow clone), installs Composer/npm packages, builds assets |
| Configures the environment | Generates `APP_KEY` if empty, sets `APP_ENV`, `APP_DEBUG=false`, `APP_URL`, database/cache/session/queue values via base64-encoded secret handling |
| Prepares the database | Runs migrations; optional demo data seeding via `run-seeders` (never creates login accounts) |
| Optimises | Storage link, `php artisan optimize` (config/route/view caches) |
| Serves it | Nginx virtual host with a 100 MB upload limit + optional HTTPS via Let's Encrypt; reloads Nginx and PHP-FPM |
| Verifies | Checks Nginx, PHP-FPM and MySQL are active, then performs a loopback `/up` health check (no server-side DNS required) |

---

## Features

- Installs PHP 8.4 with required extensions (mbstring, xml, curl, gd, zip, bcmath, intl, fileinfo, tokenizer, pdo_mysql, sqlite3, dom, simplexml)
- Installs and configures MySQL, Nginx, Composer, Node.js 22
- Idempotent application checkout (`git fetch/reset` if already cloned, otherwise `git clone --depth 1`)
- Secure secret handling: database and environment values are passed to the server base64-encoded and applied by a small PHP script (no secrets interpolated into shell command lines)
- Creates/updates MySQL database and user idempotently
- Runs `composer install --no-dev --optimize-autoloader`, `npm ci && npm run build`
- Generates `.env` from `.env.example` only if missing; sets `APP_ENV`, `APP_DEBUG=false`, `APP_URL`, generates `APP_KEY` if empty, configures database/cache/session/queue for database-backed defaults
- Runs migrations, links `public/storage` if missing, runs `php artisan optimize`
- Sets correct ownership/permissions on `storage` and `bootstrap/cache`
- Installs a `systemd` queue worker unit and a cron entry for `php artisan schedule:run`
- Configures Nginx for the domain with `fastcgi_pass unix:/run/php/php8.4-fpm.sock` (using the selected `php-version`), a 100 MB upload limit, and adds TLS via `certbot --nginx` only when `ssl-email` is provided
- Reloads Nginx and PHP-FPM after each deploy
- Verifies `nginx`, `mysql`, and `php*-fpm` are active, then performs a health check against `/up` via loopback (with `--resolve` or a `Host` header), so server-side DNS is not required
- Install-path validation (absolute, dedicated directory, no `..` or whitespace) and input validation for host, user, port, domain, email, database names and PHP version
- Demo seeding is disabled by default; when `run-seeders: 'true'` it seeds demo *data* but never creates demo login accounts

## Prerequisites

- Ubuntu 22.04 or 24.04
- A domain pointed to your server IP
- SSH access to the server (root or a user with passwordless `sudo -n`)
- This action only supports Debian/Ubuntu with `apt-get` (it will fail clearly if unsupported)

## Security Notes

- Store all sensitive values in GitHub Secrets. Never commit SSH keys or database passwords.
- If `known-hosts` is not provided, the action uses `ssh-keyscan` and explicitly warns that the host key is not pinned (MITM risk).
- Prefer pinning `known-hosts` to the server's host key.
- For reproducibility in production, pin this action to a full commit SHA rather than `@main`.
- The action never creates a known administrator password and prints no credentials after deployment.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `server-host` | Yes | - | Server hostname or IP address. |
| `server-user` | No | `root` | SSH username. Use `root` or a user with passwordless `sudo`. |
| `server-key` | Yes | - | SSH private key for authentication. |
| `server-port` | No | `22` | SSH port number. |
| `domain` | Yes | - | Domain name that points at the server. |
| `ssl-email` | No | `''` | Email for Let's Encrypt. If empty, skip TLS and serve HTTP only. |
| `db-name` | No | `venturex_erp` | MySQL database name. |
| `db-user` | No | `venturex_user` | MySQL database username. |
| `db-password` | Yes | - | MySQL database password. Store in GitHub Secrets. |
| `install-path` | No | `/var/www/venturex` | Directory on the server that will contain the application checkout. |
| `php-version` | No | `8.4` | PHP version to install. Requires PHP 8.4 or newer. |
| `app-env` | No | `production` | Laravel `APP_ENV` value. |
| `run-seeders` | No | `false` | Seed demo data (`true`/`false`). Never creates demo login accounts. |
| `known-hosts` | No | `''` | Pinned SSH host key line(s) written to `~/.ssh/known_hosts`. If empty, `ssh-keyscan` is used and the host key is not pinned. |

## Quick start

Add this to `.github/workflows/deploy.yml` in your repository:

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
      - name: Deploy VentureX ERP
        uses: SHIVAM73566/venturex-deploy-action@v1
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

**Note:** For reproducibility and security, pin to a specific commit SHA (e.g., `uses: SHIVAM73566/venturex-deploy-action@<commit-sha>`) instead of `@main` or a moving tag.

### Quick examples

**1. Deploy to a fresh DigitalOcean droplet**

```yaml
      - uses: SHIVAM73566/venturex-deploy-action@v1
        with:
          server-host: ${{ secrets.DO_SERVER_IP }}
          server-user: root
          server-key: ${{ secrets.DO_SSH_KEY }}
          domain: erp.yourdomain.com
          db-password: ${{ secrets.DB_PASSWORD }}
```

**2. Deploy to a custom SSH port with a non-root user**

```yaml
      - uses: SHIVAM73566/venturex-deploy-action@v1
        with:
          server-host: erp.yourdomain.com
          server-user: deploy
          server-key: ${{ secrets.SERVER_KEY }}
          server-port: '2222'
          domain: erp.yourdomain.com
          db-password: ${{ secrets.DB_PASSWORD }}
```

**3. Pin a PHP version and install path**

```yaml
      - uses: SHIVAM73566/venturex-deploy-action@v1
        with:
          server-host: ${{ secrets.SERVER_HOST }}
          server-user: root
          server-key: ${{ secrets.SERVER_KEY }}
          domain: erp.yourdomain.com
          db-password: ${{ secrets.DB_PASSWORD }}
          php-version: '8.4'
          install-path: /var/www/venturex
```

## After the first deployment

The action never creates a login account, so it cannot leak a known password.
Open `https://<domain>/install` (or `http://<domain>/install` when `ssl-email`
is empty) on the first visit and complete the installer. It creates the
`super_admin` account, runs migrations idempotently, and then disappears.

If you seed demo data with `run-seeders: 'true'`, the installer still creates
the administrator; demo data contains no usable credentials.

The application is checked out to `<install-path>/venturex-erp` (for example,
`/var/www/venturex/venturex-erp`). Re-running the workflow fast-forwards that
checkout to `origin/main`; the `.env` file is preserved.

### First 10 minutes in the product

1. Open **Help → Getting Started** for the guided product tour.
2. Open the floating **Product guide** (bottom-right on every page) for the introduction and a searchable list of all 84 RAG pipelines, plus questions matched to the page you are on.
3. Open **AI Center → Enterprise RAG → Open the pipeline tour** to see all 84 pipelines.
4. Index a domain (for example Finance) and ask: *"Which invoices are more than 30 days overdue?"*
5. Review the answer's sources, confidence and latency, then act in the module.
6. Invite your team and assign roles.

---

## How companies use it

| Company | Typical use |
|---|---|
| Small business (5–50) | Owner runs sales, stock, invoices and cash from one login; no separate CRM/ERP subscriptions. |
| Wholesale distributor | Supplier reliability, reorder planning and dead-stock reduction from live stock and PO data. |
| Manufacturer | Material requirement forecasts, lead-time tracking and landed-cost control. |
| Importer / exporter | HS-code guidance, demurrage risk, freight benchmarking and customs-document readiness. |
| Services agency | Pipeline, quote conversion and receivables insight. |
| Group / enterprise | Company-scoped data, role permissions, audit logs and executive scorecards. |

### What makes it different

- **84 focused RAG pipelines**, not one generic chatbot — each pipeline knows its domain and source records.
- **Guided from inside the product** — a floating Product guide on every page carries the introduction, all 84 pipelines and questions matched to the current screen.
- **Local-first retrieval** — embeddings and similarity run on your server; no external vector database required.
- **Explainable answers** — every response includes sources, confidence and latency.
- **Human approval stays in control** — purchases, payments, exports and permission changes are never autonomous.
- **One codebase, full stack** — CRM + ERP + AI + logistics + finance, self-hosted on your infrastructure.

---

## Setup GitHub Secrets

Go to **Settings → Secrets and variables → Actions → New repository secret** and create:

| Secret | Value |
|--------|-------|
| `SERVER_HOST` | Your server IP or hostname |
| `SERVER_USER` | SSH username (`root` or a user with passwordless sudo) |
| `SERVER_KEY` | Your SSH private key (PEM/OpenSSH format) |
| `DOMAIN` | Your domain (e.g., `erp.example.com`) |
| `SSL_EMAIL` | Email for Let's Encrypt (only needed if you want TLS) |
| `DB_PASSWORD` | MySQL password |
| `KNOWN_HOSTS` | (Optional) Pinned host key line(s) from `ssh-keyscan` or `ssh -o FingerprintHash=sha256 ...` |

---

## Requirements

- Ubuntu 20.04+ (22.04/24.04 recommended)
- Root or passwordless-sudo SSH access
- A domain pointed at the server IP
- Ports 80 and 443 open

## Verification & Troubleshooting

- The action verifies `nginx`, `mysql`, and `php*-fpm` are active.
- It performs a health check against `/up` over loopback (using `--resolve` for HTTPS or a `Host` header for HTTP) with retries and fails if the endpoint does not return success.
- If assets fail to build, confirm Node.js 22 is available on the runner (the action fails loudly if `node`/`npm` are missing or too old).
- If migrations fail, check Laravel logs on the server under `storage/logs/`.
- If passwordless sudo fails for a non-root user, the action will fail with a clear message.
- If you do not provide `ssl-email`, the site will be served over HTTP only.

| Symptom | Fix |
|---|---|
| `Permission denied (publickey)` | Check `SERVER_KEY` is the full private key including header/footer. |
| Nginx 502 | Confirm PHP-FPM is installed for the chosen `php-version`. |
| Blank page after deploy | Check `storage/logs/laravel.log` and file permissions on `storage` / `bootstrap/cache`. |
| Certificate failed | Confirm DNS already points at the server, then re-run the workflow. |

## What This Action Does NOT Do

- Docker deployment (not handled by this action)
- Shared hosting or cPanel
- Guaranteed outcomes or managed hosting

## Marketplace

Pricing information may appear on the GitHub Marketplace listing; this README describes functionality only. No guarantees of pricing or availability are implied.

## Support

- Product documentation: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/tree/main/documentation>
- Getting Started: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/blob/main/documentation/GETTING-STARTED.md>
- 84-pipeline RAG guide: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/blob/main/documentation/RAG-GUIDE.md>
- Issues: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/issues>
- Maintainer: Shivam Chaturvedi — chaturvedishivam179@gmail.com

## License

MIT License — see [LICENSE](LICENSE) for details.

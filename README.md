# VentureX ERP & CRM — GitHub Action

Deploy **VentureX ERP & CRM** — an AI-powered business operating system — to your own Ubuntu server in one workflow run.

[![GitHub Marketplace](https://img.shields.io/badge/Available%20on-GitHub%20Marketplace-blue)](https://github.com/marketplace/actions/venturex-erp-crm-deploy)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

> **What it is:** one Laravel application that combines CRM, ERP, procurement, inventory, finance, logistics and 80 enterprise RAG (retrieval-augmented generation) pipelines. This action installs it end-to-end: PHP, MySQL, Nginx, Composer, Node.js, migrations, demo data and a free Let's Encrypt certificate.

---

## What this action does

| Step | Result |
|---|---|
| Installs dependencies | PHP 8.4 + required extensions, MySQL, Nginx, Node.js 20, Composer |
| Creates the database | MySQL database and user with the credentials you provide |
| Deploys the app | Clones `VentureX-ERP-CRM`, installs Composer/npm packages, builds assets |
| Configures the environment | Generates `APP_KEY`, sets `APP_ENV=production`, `APP_DEBUG=false`, `APP_URL`, database credentials |
| Prepares the database | Runs migrations and seeds demo/master data |
| Optimises | Storage link, config/route/view caches |
| Serves it | Nginx virtual host + optional HTTPS via Let's Encrypt |
| Verifies | Checks Nginx, PHP-FPM and MySQL are running and prints the login URL |

---

## Quick start

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
          domain: ${{ secrets.DOMAIN }}
          db-name: venturex_erp
          db-user: venturex_user
          db-password: ${{ secrets.DB_PASSWORD }}
```

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

---

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `server-host` | Yes | — | Server hostname or IP |
| `server-user` | No | `root` | SSH username |
| `server-key` | Yes | — | SSH private key |
| `server-port` | No | `22` | SSH port |
| `domain` | Yes | — | Domain name pointed at the server |
| `db-name` | No | `venturex_erp` | MySQL database name |
| `db-user` | No | `venturex_user` | MySQL username |
| `db-password` | Yes | — | MySQL password |
| `install-path` | No | `/var/www/html` | Parent installation directory |
| `php-version` | No | `8.4` | PHP version to install |

## Required secrets

Go to **Settings → Secrets and variables → Actions → New repository secret**:

| Secret | Value |
|---|---|
| `SERVER_HOST` | Server IP or hostname |
| `SERVER_USER` | SSH username |
| `SERVER_KEY` | Private SSH key with access to the server |
| `DOMAIN` | Domain pointing at the server |
| `DB_PASSWORD` | MySQL password to create |

---

## After deployment

Your instance is available at `https://yourdomain.com`.

**Demo login**

- Email: `demo_admin@example.com`
- Password: `Demo_Admin_2026!`

Change this password immediately on a production deployment.

### First 10 minutes in the product

1. Open **Help → Getting Started** for the guided product tour.
2. Open the floating **Product guide** (bottom-right on every page) for the introduction and a searchable list of all 80 RAG pipelines, plus questions matched to the page you are on.
3. Open **AI Center → Enterprise RAG → Open the pipeline tour** to see all 80 pipelines.
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

- **80 focused RAG pipelines**, not one generic chatbot — each pipeline knows its domain and source records.
- **Guided from inside the product** — a floating Product guide on every page carries the introduction, all 80 pipelines and questions matched to the current screen.
- **Local-first retrieval** — embeddings and similarity run on your server; no external vector database required.
- **Explainable answers** — every response includes sources, confidence and latency.
- **Human approval stays in control** — purchases, payments, exports and permission changes are never autonomous.
- **One codebase, full stack** — CRM + ERP + AI + logistics + finance, self-hosted on your infrastructure.

---

## Requirements

- Ubuntu 20.04+ (22.04/24.04 recommended)
- Root or passwordless-sudo SSH access
- A domain pointed at the server IP
- Ports 80 and 443 open

## Troubleshooting

| Symptom | Fix |
|---|---|
| `Permission denied (publickey)` | Check `SERVER_KEY` is the full private key including header/footer. |
| Nginx 502 | Confirm PHP-FPM is installed for the chosen `php-version`. |
| Blank page after deploy | Check `storage/logs/laravel.log` and file permissions on `storage` / `bootstrap/cache`. |
| Certificate failed | Confirm DNS already points at the server, then re-run the workflow. |

## Support

- Product documentation: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/tree/main/documentation>
- Getting Started: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/blob/main/documentation/GETTING-STARTED.md>
- 80-pipeline RAG guide: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/blob/main/documentation/RAG-GUIDE.md>
- Issues: <https://github.com/SHIVAM73566/VentureX-ERP-CRM/issues>

## License

MIT License — see [LICENSE](LICENSE) for details.

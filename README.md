# VentureX ERP & CRM - GitHub Action

Deploy VentureX ERP & CRM - AI-Powered Business Operating System to your server in one click.

[![GitHub Marketplace](https://img.shields.io/badge/Available%20on-GitHub%20Marketplace-blue)](https://github.com/marketplace/actions/venturex-erp-crm-deploy)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Price](https://img.shields.io/badge/Price-%2489.99%2Fmonth-orange)](https://github.com/marketplace/actions/venturex-erp-crm-deploy)

## Pricing Plans

| Plan | Price | Features |
|------|-------|----------|
| **Single** | $89.99/month | Deploy to 1 server - Limited features, AI included, 5-day free AI trial |
| **Professional** | $150.22/month | Deploy to 5 servers - Full business features, priority support |
| **Enterprise** | $350/month | Unlimited servers - Full updates for company/big business, dedicated support |

> **Enterprise Discount:** Use code `SAVE29` at checkout for $29 off your first month!

## What This Action Does

This GitHub Action automatically deploys VentureX ERP & CRM to your server. It:

- Installs all required dependencies (PHP 8.3, MySQL, Nginx, Node.js)
- Sets up the database
- Deploys the application
- Configures Nginx web server
- Enables HTTPS with Let's Encrypt SSL
- Seeds demo data

## Quick Start

Add this to your `.github/workflows/deploy.yml`:

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
        uses: SHIVAM73566/venturex-deploy-action@main
        with:
          server-host: ${{ secrets.SERVER_HOST }}
          server-user: ${{ secrets.SERVER_USER }}
          server-key: ${{ secrets.SERVER_KEY }}
          domain: ${{ secrets.DOMAIN }}
          db-name: venturex_erp
          db-user: venturex_user
          db-password: ${{ secrets.DB_PASSWORD }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `server-host` | Yes | - | Your server hostname or IP |
| `server-user` | No | `root` | SSH username |
| `server-key` | Yes | - | SSH private key |
| `server-port` | No | `22` | SSH port |
| `domain` | Yes | - | Your domain name |
| `db-name` | No | `venturex_erp` | MySQL database name |
| `db-user` | No | `venturex_user` | MySQL username |
| `db-password` | Yes | - | MySQL password |
| `install-path` | No | `/var/www/html` | Installation path |
| `php-version` | No | `8.3` | PHP version |
| `use-docker` | No | `false` | Use Docker deployment |

## Setup GitHub Secrets

Go to your repo → Settings → Secrets and variables → Actions → New repository secret

| Secret Name | Value |
|-------------|-------|
| `SERVER_HOST` | Your server IP (e.g., 192.168.1.100) |
| `SERVER_USER` | SSH username (e.g., root) |
| `SERVER_KEY` | Your SSH private key |
| `DOMAIN` | Your domain (e.g., erp.yourdomain.com) |
| `DB_PASSWORD` | MySQL password |

## Example: Deploy to DigitalOcean

```yaml
name: Deploy to DigitalOcean

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy VentureX ERP
        uses: SHIVAM73566/venturex-deploy-action@main
        with:
          server-host: ${{ secrets.DO_SERVER_IP }}
          server-user: root
          server-key: ${{ secrets.DO_SSH_KEY }}
          domain: erp.yourdomain.com
          db-name: venturex_erp
          db-user: venturex_user
          db-password: ${{ secrets.DB_PASSWORD }}
```

## Example: Deploy with Docker

```yaml
name: Deploy with Docker

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy VentureX ERP
        uses: SHIVAM73566/venturex-deploy-action@main
        with:
          server-host: ${{ secrets.SERVER_HOST }}
          server-user: root
          server-key: ${{ secrets.SERVER_KEY }}
          domain: erp.yourdomain.com
          db-name: venturex_erp
          db-user: venturex_user
          db-password: ${{ secrets.DB_PASSWORD }}
          use-docker: 'true'
```

## What Gets Installed

- PHP 8.3 with all required extensions
- MySQL 8.0
- Nginx web server
- Node.js 20
- Composer 2
- Let's Encrypt SSL certificate

## Post-Deployment

After deployment, your VentureX ERP & CRM will be available at:

```
https://yourdomain.com
```

**Demo Login:**
- Email: demo_admin@example.com
- Password: Demo_Admin_2026!

## Requirements

- Ubuntu 20.04+ server
- Root SSH access
- Domain pointed to your server IP

## Support

- Documentation: /documentation/
- Issues: GitHub Issues
- Email: support@venturexerp.com

## License

MIT License - see [LICENSE](LICENSE) for details.

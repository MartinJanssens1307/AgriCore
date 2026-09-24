# AgriCore

AgriCore is a Django-based ERP project for Belgian farmers. It is currently in an early, learning-oriented stage, but it already contains a foundation for company management, customers, products, transactions, and cadastral/field management.

## Project goal

The project aims to become a modern farm-management platform that helps farmers manage their operations from one central system. The long-term vision is ambitious: it should eventually support farm administration, field operations, contracting work, inventory, workers, machinery, reporting, and AI-assisted decision support.

## Current status

The repository already contains:

- A Django project (`farmerp/`)
- A core ERP app (`ERP/`)
- A field/cadastral app (`Fields/`)
- Basic company, customer, partner, address, product, and transaction models
- Transaction validation logic
- Factory-based tests in the `Fields` app
- Docker configuration for local development

The project is not yet a polished product. It is best understood as a sandbox and evolving platform, with a growing foundation for actual farm use cases.

## Tech stack

- Python
- Django
- PostgreSQL-ready configuration via `dj-database-url`
- SQLite fallback for local development
- Docker / Docker Compose
- Gunicorn / WhiteNoise
- HTML templates

## Repository structure

```text
AgriCore/
├── ERP/                # Core business domain app
├── Fields/             # Field / cadastral domain app
├── farmerp/            # Django project config
├── .devcontainer/
├── Dockerfile
├── docker-compose.yml
├── build.sh
├── manage.py
├── requirements.txt
├── package.json
├── package-lock.json
├── .gitignore
├── README.md
└── docs/
```

## Quick start

```bash
# clone
git clone https://github.com/MartinJanssens1307/AgriCore.git
cd AgriCore

# install dependencies
pip install -r requirements.txt

# run migrations
python manage.py migrate

# start development server
python manage.py runserver
```

For Docker-based usage:

```bash
docker-compose up --build
```

## Documentation

See the documentation folder for the current project direction and development roadmap.

## Important notes

- The project is currently Belgium-first, but the model already anticipates international customer/supplier data.
- Peppol support exists in the codebase but should be treated as exploratory/deferred rather than a current early-release requirement.
- The project is intentionally incremental: the goal is to build small, useful pieces, validate them, and refine the architecture as the business context becomes clearer.

## Long-term vision

The long-term ambition includes:

- farm master data management
- fields and crop tracking
- contracting/service workflows
- machinery and employee management
- stock, harvests, and product flows
- reporting, dashboards, and AI-assisted decision support
- external ecosystem partnerships in agriculture

This document is intentionally simple and intentionally evolving.

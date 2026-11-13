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
- Cadastral parcel view tests in the `Fields` app
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
    ├── architecture.md
    ├── development.md
    └── decisions/
        ├── 001-tenant-model-and-data-scoping.md
        ├── 002-transaction-lifecycle-and-status-rules.md
        ├── 003-product-snapshot-strategy.md
        ├── 004-vat-null-and-money-precision.md
        ├── 005-fields-and-erp-boundary.md
        ├── 006-peppol-scope-and-prioritization.md
        └── adr-template.md
```

## Quick start

The verified development workflow uses VS Code Dev Containers and Docker Desktop. Install the Dev Containers extension, open the repository in VS Code, and choose **Reopen in Container**. The first time, run `python manage.py migrate` in a container terminal; Django starts automatically when the container attaches.

See the [development setup guide](docs/development.md) for prerequisites, Tailwind watch, tests, troubleshooting, and production-status notes.

## Documentation

- [Development setup](docs/development.md)
- [Architecture overview](docs/architecture.md)
- [Tenant model and data scoping ADR](docs/decisions/001-tenant-model-and-data-scoping.md)
- [Transaction lifecycle and status rules ADR](docs/decisions/002-transaction-lifecycle-and-status-rules.md)
- [Product snapshot strategy ADR](docs/decisions/003-product-snapshot-strategy.md)
- [VAT null handling and money precision ADR](docs/decisions/004-vat-null-and-money-precision.md)
- [Fields and ERP domain boundary ADR](docs/decisions/005-fields-and-erp-boundary.md)
- [Peppol scope and prioritization ADR](docs/decisions/006-peppol-scope-and-prioritization.md)
- [ADR template](docs/decisions/adr-template.md)

Documentation is being built incrementally from confirmed project decisions. Future guides can be added here as their setup and behavior are verified.

## Important notes

- The project is currently Belgium-first, but the model already anticipates international customer/supplier data.
- The Peppol customer lookup is exploratory, not production-ready, and remains the lowest-priority feature until user adoption or profitability justifies revisiting its API and usage costs. See the [Peppol scope ADR](docs/decisions/006-peppol-scope-and-prioritization.md).
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

# Architecture overview

AgriCore is a Django application currently organized around two Django apps: `ERP` for company and commercial workflows, and `Fields` for cadastral parcel management. The intended boundary is for ERP to remain the shared tenant/platform core while Fields owns field-specific workflows; see [ADR 005: Fields and ERP domain boundary](./decisions/005-fields-and-erp-boundary.md). This page distinguishes current code from that intended direction.

## Runtime and configuration

- [`farmerp/`](../farmerp/) is the Django project package. Its settings, root URL configuration, ASGI entry point, and WSGI entry point live there.
- [`manage.py`](../manage.py) is the local Django management entry point.
- [`farmerp/settings.py`](../farmerp/settings.py) installs `ERP` and `Fields`, configures middleware, templates, authentication, and static files. It uses `DATABASE_URL` when present and otherwise falls back to SQLite.
- Production settings require `SECRET_KEY`; `DEBUG` and `ALLOWED_HOSTS` are environment-configured. Static files are served through WhiteNoise when debug mode is disabled.
- [`requirements.txt`](../requirements.txt) lists Python dependencies. [`package.json`](../package.json) provides Tailwind CSS build and watch scripts for the templates and static assets.

## Applications and domain

### ERP (current code; shared core direction)

The [`ERP` app](../ERP/) currently contains company and commercial domain models, views, forms, templates, migrations, and static assets. The intended direction is for it to remain the shared company/tenant and platform core. CRM and sales are still part of this app; extracting them is deferred until their independent workflows justify a separate app boundary.

Its principal models in [`ERP/models.py`](../ERP/models.py) are:

- `Company`: a farm or business, including legal and address details.
- `CompanyAccess`: a user-to-company membership with a role value.
- `BusinessPartner` and `Customer`: partner/customer records belonging to a company.
- `Address`: an address belonging to a business partner.
- `Product`: a company-owned product or service.
- `Transaction` and `TransactionLineItem`: sales/purchase records, their lines, totals, and transaction snapshots.

The `ERP/Views/` modules implement company, customer, address, product, and transaction workflows. Templates are under `ERP/templates/ERP/`.

### Fields

The [`Fields` app](../Fields/) currently manages cadastral parcels. `CadastralParcel` in [`Fields/models.py`](../Fields/models.py) belongs to an ERP `Company` and stores parcel details, surface values, and GeoJSON data. Its views in [`Fields/views.py`](../Fields/views.py) list, create, display, and delete parcels, and provide endpoints that query Wallonia's cadastral map service. The intended Fields domain also includes field geodata and, as they are developed, crop planning and field work. Customer/contract associations and links to products, services, or transactions are future cross-domain relationships, not current workflows.

## Request and URL flow

The root URL configuration is [`farmerp/urls.py`](../farmerp/urls.py):

- `/admin/` uses Django admin.
- `/accounts/` includes Django authentication routes.
- Root-level ERP routes are included from [`ERP/urls.py`](../ERP/urls.py), including the landing page, login, registration, and company setup.
- `/app/` includes the application routes from [`farmerp/urls_app.py`](../farmerp/urls_app.py). ERP routes are mounted there, and field routes are mounted under `/app/fields/`.

`TenantSecurityMiddleware` in [`ERP/middleware.py`](../ERP/middleware.py) protects `/app/` routes. It requires an authenticated user, resolves the active company, and attaches it to the request as `request.tenant`. Outside `/app/`, it sets `request.tenant` to `None`.

`get_current_company` in [`ERP/Views/utils.py`](../ERP/Views/utils.py) currently reads the active company ID from the browser session and checks that the user has a corresponding `CompanyAccess` record. When no valid session selection exists, it falls back to the first membership returned by the database. `set_current_company` validates membership before storing a new selection in the session.

Many ERP and Fields views filter records using `request.tenant`; addresses are scoped through their business partner. Tenant isolation therefore depends on each relevant query and related-object lookup applying the company scope. The current view/query patterns are not a centralized authorization layer.

## Tenant access: current behavior versus intended policy

The intended membership, read/write, company-selection, and revocation rules are described in [ADR 001](./decisions/001-tenant-model-and-data-scoping.md). In the current implementation:

- `CompanyAccess` stores Owner, Employee, and Accountant role values, but role-specific write/read permissions were not found in the inspected application paths.
- Company creation associates the current user with the company as Owner.
- The active company is stored in the browser session, not as a persistent per-user preference across sessions and devices.
- If a user's selected company membership is no longer valid, the helper clears that selection and automatically falls back to another membership if one exists. This differs from the ADR's decision not to silently switch tenants.
- An invitation and acceptance workflow was not found in the inspected application paths.

## External services and integrations

- The Fields views call the Wallonia ArcGIS cadastral service for map tiles and parcel identification.
- [`ERP/services_peppol.py`](../ERP/services_peppol.py) contains an exploratory Ibanity-backed Peppol customer lookup. It is the lowest-priority feature for now and is not production-ready or a complete invoicing workflow; see [ADR 006](./decisions/006-peppol-scope-and-prioritization.md).

## Tests and schema changes

Django migrations are maintained in each app's `migrations/` directory. The currently visible app test module is [`Fields/tests.py`](../Fields/tests.py), which tests parcel view behavior. Use Django's test runner (`python manage.py test`) to run the test suite.

## Related documentation

- [Development setup](./development.md)
- [Tenant model and data scoping ADR](./decisions/001-tenant-model-and-data-scoping.md)
- [Transaction lifecycle and status rules ADR](./decisions/002-transaction-lifecycle-and-status-rules.md)
- [Product snapshot strategy ADR](./decisions/003-product-snapshot-strategy.md)
- [VAT null handling and money precision ADR](./decisions/004-vat-null-and-money-precision.md)
- [Fields and ERP domain boundary ADR](./decisions/005-fields-and-erp-boundary.md)
- [Peppol scope and prioritization ADR](./decisions/006-peppol-scope-and-prioritization.md)
- [ADR template](./decisions/adr-template.md)
- [README quick start](../README.md#quick-start)

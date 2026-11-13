# Development setup

This guide documents AgriCore's current, verified development workflow: VS Code Dev Containers with Docker Desktop. It is the only setup the project owner has personally used and validated. Native setup instructions are intentionally omitted rather than presented as a tested alternative.

## Prerequisites

- Docker Desktop, running before opening the development container.
- Visual Studio Code.
- The **Dev Containers** extension for VS Code.
- Git, to clone the repository.

On Windows, use a WSL/Linux filesystem location for the repository rather than a Windows-mounted path. This improves file performance and helps Tailwind's file watcher detect changes reliably.

## First-time setup

1. Clone the repository and open its folder in VS Code.
2. When prompted, choose **Reopen in Container**. If the prompt does not appear, run **Dev Containers: Reopen in Container** from the Command Palette.
3. Wait for the development container to build. The Dev Container uses Docker Compose to start the application and PostgreSQL services. Its post-create command installs Python and Node dependencies.
4. The Django development server starts automatically when the container attaches. Open [http://localhost:8000](http://localhost:8000).
5. In a VS Code terminal connected to the container, apply database migrations:

   ```bash
   python manage.py migrate
   ```

   Repeat this after pulling changes that add or update migrations.

## Daily development

### Django server

The Dev Container starts `python manage.py runserver 0.0.0.0:8000` on attach. If you stop that terminal process, restart it from a container terminal:

```bash
python manage.py runserver 0.0.0.0:8000
```

### Tailwind CSS

CSS watch is optional if you are not changing templates or styles. To rebuild the CSS automatically as you edit files, open a second VS Code terminal in the container and run:

```bash
npm run css:watch
```

For a one-time CSS build instead:

```bash
npm run css:build
```

### Tests and admin account

Run the Django test suite from a container terminal:

```bash
python manage.py test
```

To create a local Django admin account:

```bash
python manage.py createsuperuser
```

Then visit [http://localhost:8000/admin/](http://localhost:8000/admin/).

## Configuration and data

- The Dev Container uses the PostgreSQL service in `docker-compose.yml`.
- `DEBUG=1`, the database URL, and local host settings are supplied by the Compose configuration. No separate `.env` setup is required for this workflow.
- Compose database credentials are local development defaults only. Do not reuse them for a public or production database.
- The Compose PostgreSQL data is stored in the `postgres_data` named volume and persists when containers stop.

## Common workflow checks

- **The site does not load:** make sure Docker Desktop is running and the VS Code window is attached to the Dev Container. Check the Django terminal for errors.
- **Database tables are missing:** run `python manage.py migrate` in a container terminal.
- **CSS changes do not appear:** start `npm run css:watch` in a second container terminal and keep it running. On Windows, confirm the repository is in the WSL/Linux filesystem rather than under a Windows-mounted path.
- **Tests fail after pulling changes:** apply migrations, then rerun `python manage.py test`.

## Production status

The application currently runs on Render's free tier, but production deployment is not yet a mature, documented operating procedure. The project may move to a VPS from a provider such as OVH or Hetzner. These development instructions are not production deployment guidance; document a VPS deployment separately once its environment and operational requirements are decided.

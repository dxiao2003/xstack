# Supported Stacks

This file is guidance, not a template source. Check current official documentation online before scaffolding any selected framework.

## Recommended Defaults

- Frontend: React + Vite + TypeScript
- Backend: Python + FastAPI
- Database: Postgres
- Local development: Docker Compose
- Frontend package manager: `pnpm`
- Frontend component framework: Material UI
- Frontend routing: TanStack Router
- Backend data layer: SQLAlchemy + Alembic

## Documentation Sources To Prefer

- Vite: official Vite guide and CLI docs.
- React: official React docs for app structure and framework guidance.
- FastAPI: official FastAPI docs for application layout, Uvicorn reload, testing, and Docker.
- SQLAlchemy and Alembic: official docs for engine URLs, sessions, models, migrations, and async/sync tradeoffs.
- Postgres Docker image: official image docs for environment variables, volumes, and health checks.
- Material UI: official install and framework integration docs.
- TanStack Router: official install and Vite/React examples.
- Package managers: official `pnpm`, `npm`, or `yarn` docs for lockfiles, stores, and container usage.

## Docker Compose Patterns

Generate Compose files for the exact selected stack. For the recommended stack, the expected shape is:

- `frontend`: build from `frontend/`, mount `./frontend:/app`, keep `/app/node_modules` in a named volume, run the documented dev server with host binding, and expose the Vite port.
- `backend`: build from `backend/`, mount `./backend:/app`, keep Python package caches or virtualenvs in named volumes when the chosen tool supports it, run Uvicorn with reload, and expose the API port.
- `db`: use the official Postgres image, set local-only credentials through environment variables or an env file, attach a named data volume, and add a health check.

Bind mounts should map directly to the repo directories the user edits. Named volumes should cover paths that are produced inside containers and should not be overwritten by the host mount, especially `node_modules`, package stores, virtualenvs, caches, and database data.

## Smoke Test Surface

For the recommended FastAPI + Postgres stack, create a backend health endpoint that obtains a connection through the generated app's database layer and executes a simple query such as `SELECT 1`. Return JSON that distinguishes API process health from database health, and fail with a non-2xx status if the query fails.

For the recommended Vite + React stack, make the initial landing page fetch that health endpoint from the browser and render the returned status. The page must also render network, HTTP, parse, and backend-reported errors so local setup problems are visible without opening devtools.

## Reload And Watch Checklist

- Bind dev servers to `0.0.0.0` inside containers.
- Publish stable host ports.
- Enable the documented reload/watch mode for each framework.
- Add polling watch options when the selected framework needs them under Docker Desktop, WSL, or remote filesystems.
- Verify reload by editing a mounted source file and confirming the service reflects the change without rebuilding the image.

## Validation Checklist

- Docker can build every service from a clean checkout.
- `docker compose up -d` starts all services.
- The database-backed backend health endpoint returns success.
- The frontend landing page loads in a browser-capable validation step and displays a successful backend/database health result.
- Frontend lint/test/build commands run inside the frontend container.
- Backend lint/test commands run inside the backend container.
- Database migrations can run from a clean database when a migration tool is selected.
- `docker compose down -v` cleans up local state after validation.

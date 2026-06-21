---
name: bootstrap-webapp-project
description: Use when starting a new full-stack web app from scratch. Scaffolds frontend and backend from current framework docs into a Docker-first local dev environment with hot reload, a database-backed health check, and a validated hello-world baseline, then reinitializes git and optionally creates a GitHub repo.
---

# Bootstrap Webapp Project

## Philosophy

Use this skill to create a new web app project from current framework guidance, not from pre-created product templates bundled with the skill.

The skill's value is:

1. Check the selected frameworks' official documentation online before scaffolding.
2. Generate the project structure, framework files, Dockerfiles, Compose file, validation script, and decision log dynamically for the chosen stack.
3. Make local development reproducible with Docker Compose, bind-mounted source code, persistent dependency/cache volumes, and automatic reload when mounted files change.

## Workflow

1. Confirm that the current working directory is the repository to initialize.
2. Ask for stack decisions one by one unless the user requests defaults.
3. Recommend a boring default stack unless the user has stronger requirements:
   - Frontend: React + Vite + TypeScript
   - Backend: Python + FastAPI
   - Database: Postgres
   - Frontend component framework: Material UI when compatible with React
   - Frontend routing: TanStack Router when compatible with React
   - Backend data layer: SQLAlchemy + Alembic when compatible with the backend and database
   - Local development: Docker Compose for all services, with bind-mounted source and hot reload
4. For every selected framework/tool, check current official documentation online before generating files. Prefer official docs, release notes, and CLI references over blog posts or memory.
5. Record the chosen scaffold commands, source links, versions, and key docs-derived decisions in `docs/decisions/bootstrap.md`.
6. Generate the project using the best documented bootstrap path for the selected stack. Prefer official CLI scaffolds where they exist, such as framework create commands, then edit the generated files to integrate the full stack.
7. Add a scaffold smoke-test surface:
   - A backend health endpoint that uses the application's configured database connection and executes a real lightweight query, such as `SELECT 1`, before returning success.
   - A frontend hello-world landing page that calls the backend health endpoint from the browser and displays the returned status or the full error state.
   - Tests for the backend health behavior and the frontend rendering/error handling where the selected frameworks make this practical.
8. Create Docker development files for the exact generated stack:
   - `compose.yaml`
   - service-specific `Dockerfile`s or documented dev container commands
   - `.dockerignore` files
   - project-specific environment examples
   - `scripts/validate_scaffold.sh`
9. Configure bind mounts so host edits update the running containers without rebuilding:
   - Mount the frontend source directory into the frontend service workdir.
   - Mount the backend source directory into the backend service workdir.
   - Use named volumes for dependency directories that should not be shadowed by host mounts, such as `node_modules`, package manager stores, virtual environments, Python caches, and database data.
   - Set file-watching environment variables or polling modes when needed for Docker Desktop, WSL, or network filesystems.
   - Run framework dev servers with reload/watch flags enabled.
10. Configure pre-commit hooks for the selected stack:
    - Choose a hook runner compatible with the selected languages. Default to `pre-commit` when Python is in the stack (it handles multi-language repos). Default to `husky` + `lint-staged` when the stack is Node.js-only.
    - Install and configure hooks that cover the lint, format, and type-check commands documented for the selected frameworks. See the "Pre-Commit Hooks" section for per-stack defaults.
    - Install the hook runner and required hook tools on the host so hooks fire at commit time. Document the install commands in `README.md` under a "Developer Setup" section.
    - Run hooks against all current files (`pre-commit run --all-files` or equivalent) to confirm the scaffold passes cleanly before Docker validation.
    - Record hook configuration files and install commands in `docs/decisions/bootstrap.md`.
11. Validate the scaffold through Docker Compose before final cleanup, including the browser-visible hello-world page success state.
12. During cleanup, ask whether to remove the health endpoint and hello-world landing page. If the user says yes, remove them and any tests/docs/validation checks that only exist for those scaffold smoke surfaces.
13. After validation and the smoke-surface cleanup pass, present all potentially destructive handoff steps to the user together as a single approval, rather than asking about each one separately. List the steps and exactly what each will remove or change:
    - Copy the project guidance template from `assets/AGENTS.md` to `AGENTS.md` in the root directory of the generated project, then create a `CLAUDE.md` symlink pointing to it (`ln -s AGENTS.md CLAUDE.md`). Do this right before removing the bootstrap code and reinitializing git, so the template ships with the new project.
    - Remove bootstrap-only files (list what will be deleted).
    - Remove validation-only artifacts that real development will not use, such as host virtual environments, dependency directories, caches, or packages/scripts/tools installed just to run validation.
    - Reinitialize git, including deleting any existing `.git` history.
    - Create the initial commit.

    Ask the user to approve the whole set, and tell them they can approve only a subset by naming which steps to skip. Perform only the approved steps and skip the rest. Then offer GitHub upstream creation with `gh` only when requested, confirming again at the point of repo creation.

## Questions To Ask

Ask these individually, not as a bulk questionnaire:

1. Project name.
2. Frontend choice. Nudge to React + Vite + TypeScript.
3. Backend choice. Nudge to Python + FastAPI.
4. Database choice. Nudge to Postgres.
5. Local development environment. Strongly recommend Docker Compose.
6. Frontend package manager. Nudge to `pnpm`.
7. Frontend component framework. Nudge to Material UI when compatible with React.
8. Frontend routing. Nudge to TanStack Router when compatible with React.
9. Backend database migration system / ORM. Nudge to SQLAlchemy + Alembic when compatible.
10. Whether to offer GitHub upstream creation with `gh` after cleanup and git initialization.

## Documentation Check

Before creating files, browse the official documentation for each selected framework/tool. Capture enough detail to justify the scaffold:

- Official scaffold command or starter template recommendation.
- Current package manager guidance.
- Development server host binding and reload/watch settings.
- Docker/container guidance when provided by the framework.
- Test, lint, format, build, and migration commands.
- Version constraints and compatibility notes for selected integrations.

Use exact docs-derived commands when they are suitable. If official docs provide multiple routes, choose the path that is simplest, actively maintained, and compatible with Docker bind-mounted development. Clearly state when a choice is an inference from docs rather than an explicit recommendation.

## Smoke Test Surface

Create a temporary end-to-end proof that the generated stack is wired correctly:

- Backend health endpoint:
  - Use a conventional path such as `/health` or `/api/health`.
  - Connect through the same database configuration and driver the application will use.
  - Execute a lightweight query that proves authentication, networking, database selection, and driver configuration work. Do not treat an opened socket or ORM object construction as sufficient.
  - Return structured JSON with at least service status and database status. Include enough error detail for local debugging, but do not leak secrets.
  - Return a non-2xx status when the database check fails.
- Frontend hello-world landing page:
  - Render immediately with a clear loading state.
  - Fetch the backend health endpoint from the browser through the configured local URL/proxy.
  - Display the success payload when healthy.
  - Display HTTP, network, JSON parsing, and backend-reported errors when unhealthy.
  - Keep the implementation simple and easy to remove after scaffolding.

Record these files in `docs/decisions/bootstrap.md` so cleanup can remove them accurately if the user chooses.

## Docker Development Contract

Make Docker Compose the default local environment. Generate service configuration that is reproducible and pleasant for iterative development:

- Bind service source into containers with paths matching the generated repo layout, for example `./frontend:/app` and `./backend:/app`.
- Keep generated dependency/cache directories in named volumes, for example `/app/node_modules`, pnpm store, uv cache, Python virtualenv, and Postgres data.
- Bind dev servers to `0.0.0.0` inside containers and publish stable host ports.
- Enable automatic reload:
  - Vite: run the documented dev command with `--host 0.0.0.0`; set polling watch options when needed.
  - FastAPI/Uvicorn: run with `--reload --host 0.0.0.0`.
  - Other frameworks: use the official watch/reload mode from the documentation.
- Use non-root users or ownership repair only where needed to avoid host-owned artifact problems.
- Put secrets in environment files or local-only examples, not committed real values.

For stack-specific Docker details, read `references/supported-stacks.md`.

## Pre-Commit Hooks

Add pre-commit hooks so every commit on the new project is gated by the same linters and checks CI would run.

### Hook Runner Selection

Choose a hook runner suited to the primary runtime(s) in the selected stack:

- **Multi-language or Python-containing stacks**: use `pre-commit` (https://pre-commit.com). It is runtime-agnostic and handles mixed repos with a single config file.
- **Node.js-only stacks**: use `husky` + `lint-staged`. Configure `lint-staged` to scope checks to staged files for speed.
- **Other single-runtime stacks**: prefer the hook runner standard for that ecosystem, or use `pre-commit` as a safe default when none is established.

### What Hooks To Configure

For each language and framework in the selected stack, identify and configure hooks for:

1. **Linting** — static analysis and bug detection appropriate for the language (e.g., `eslint`, `ruff`, `golangci-lint`, `rubocop`).
2. **Formatting** — enforce a consistent style, either auto-applying or checking (e.g., `prettier`, `ruff format`, `gofmt`, `rustfmt`).
3. **Type checking** — when the selected language has a static type system (e.g., `tsc --noEmit` for TypeScript, `mypy` for Python; the compiler itself for Go, Rust, or Java).

Check the official documentation and community standards for each selected language before pinning tool versions. Do not use remembered version numbers — fetch current releases from the tool's PyPI page, npm registry, or release notes.

**Always include regardless of language:**
- Trailing whitespace and end-of-file newline enforcement
- YAML and JSON syntax validation
- Merge-conflict marker detection

See `references/supported-stacks.md` for common language-to-tool mappings and a configuration example.

### Installation

Install the hook runner on the host so hooks fire at every `git commit`:

- For `pre-commit`: `pip install pre-commit` (or `pipx install pre-commit`), then `pre-commit install` in the repo root.
- For `husky`: `pnpm add -D husky lint-staged`, then `pnpm exec husky init`.
- For other runners: follow the official installation guide for the selected runner.

Document these commands in `README.md` under a "Developer Setup" section. Also add a note to the generated `AGENTS.md` / `CLAUDE.md` project guidance so coding agents know to run setup after cloning.

### Validation

After installing hooks, run them against all files in the scaffold to confirm they pass cleanly:

- `pre-commit`: `pre-commit run --all-files`
- `husky` + `lint-staged`: run each lint and format command directly against all relevant files.

Fix any issues surfaced before proceeding to Docker validation. Do not commit a scaffold that fails its own hooks.

## Decision Log

Create `docs/decisions/bootstrap.md` in the generated project. Include:

- Project name and creation timestamp.
- Selected frontend, backend, database, package manager, component framework, router, data layer, and local environment.
- Official documentation links consulted, with retrieval date.
- Scaffold commands run.
- Docker Compose service layout, bind mounts, named volumes, ports, and reload/watch settings.
- Pre-commit hook runner chosen, hooks configured, hook versions pinned, and install commands.
- Validation commands and results.

Keep this decision log in the final project.

## Validation

Generate `scripts/validate_scaffold.sh` for the selected stack instead of copying a bundled script. Prefer a Docker validation path:

1. `docker compose down -v`
2. `docker compose build`
3. `docker compose up -d`
4. Wait for the database-backed backend health endpoint to return success.
5. Wait for the frontend dev server.
6. Load the frontend hello-world landing page with a browser-capable check, such as Playwright when available, and assert that it displays a successful backend/database health result. A raw `curl` of the frontend HTML is not enough for this check because it does not prove the browser executed the health request.
7. Run backend tests, lint, type checks, migrations, or health checks as applicable.
8. Run frontend lint, tests, and production build as applicable.
9. Run pre-commit hooks against all files (`pre-commit run --all-files` or equivalent) and assert they pass cleanly.
10. `docker compose down -v` during cleanup unless the user wants the stack left running.

If Docker is unavailable, report that clearly and ask whether to install/use Docker or adapt to a non-Docker validation flow. Do not silently treat bare-metal validation as equivalent to the Docker-first target.

## Compatibility Audit

After validation passes and before cleanup/git handoff, verify that selected frameworks and generated files are mutually compatible:

- Generated imports are backed by direct dependency entries.
- Dependency files do not include unselected optional modules.
- Package-manager commands match the selected package manager everywhere.
- Docker Compose commands match the generated framework scripts.
- Database URLs, migration tools, and drivers match the selected database.
- The backend health endpoint uses the same database connection configuration as application code and executes a real query.
- The frontend hello-world page calls the backend through the same URL/proxy path users will use locally.
- Reload/watch configuration works with bind-mounted source.
- Pre-commit hooks are installed and pass cleanly against the scaffold (`pre-commit run --all-files` or equivalent exits 0).
- Hook configuration files reference the correct linter/formatter versions for the selected stack.
- Validation exercises the selected framework integrations.

Fix generated project files and rerun validation when incompatibilities exist.

## Cleanup Choices

Before final git handoff, ask the user whether to keep or remove the scaffold smoke-test surface:

- If the user keeps it, leave the health endpoint, hello-world landing page, related tests, and validation checks in place.
- If the user removes it, delete the endpoint and landing page, remove tests that only cover those scaffold surfaces, update validation so it no longer expects them, and leave any reusable database connectivity utilities that the real application still needs.
- If removing the landing page would leave the frontend without a routable page, replace it with the selected framework's minimal app shell rather than leaving a broken route.
- Ask whether to remove validation-only artifacts that will not be used for actual future development, such as host-side virtual environments, dependency directories, caches, and any packages, tools, or scripts that were installed on the host purely to run validation. Distinguish these from artifacts the running stack genuinely needs:
  - Remove things that only existed to validate the scaffold, for example a host `.venv` or `node_modules` created outside Docker, browser binaries or test runners installed just for the smoke check, and throwaway helper scripts.
  - Keep dependency manifests, lockfiles, Dockerfiles, `compose.yaml`, and the in-container dependency setup that real development depends on.
  - **Never remove pre-commit hook configuration.** Hook runner config files and any lint, format, or type-check tool configuration added to project manifests are permanent project infrastructure, not scaffold-only artifacts.
  - When unsure whether an artifact is validation-only or needed for development, ask rather than deleting it.
- Update `docs/decisions/bootstrap.md` with the cleanup decision and files removed.

## Git And GitHub

The default handoff is:

1. Copy `assets/AGENTS.md` to `AGENTS.md` in the root of the generated project, then symlink `CLAUDE.md` to it (`ln -s AGENTS.md CLAUDE.md`) so both Claude Code and other agents pick up the same guidance. Do this right before removing the bootstrap code and reinitializing git so the project guidance ships with the new project.
2. Remove bootstrap-only files that should not live in the product repo. Do not remove skills that may be used beyond the bootstrap phase.
3. Remove any template `.git` history only after explicit confirmation.
4. Run `git init`.
5. Commit generated project files.
6. Offer `gh repo create`, defaulting to private visibility. Prompt the user for the repo name.

Do not create a GitHub repository without explicit confirmation at the point of creation.

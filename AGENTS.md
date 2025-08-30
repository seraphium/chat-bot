# Repository Guidelines

## Project Structure & Module Organization
- Root: `bolt-chatbot-ui/` (Next.js + TypeScript), `langchain_backend/` (FastAPI + LangChain).
- Backend: `app/` (api, chains, config, core, models, services, tasks, utils), `tests/`, `docs/`, `examples/`.
- Frontend: `app/` (App Router), `components/`, `lib/`, `hooks/`, `config/`.

## Build, Test, and Development Commands
- Backend — run API: `cd langchain_backend && poetry run python -m app.main`.
- Backend — migrate DB: `poetry run alembic upgrade head`.
- Backend — tests: `poetry run pytest` (markers: `unit`, `integration`, `e2e`; coverage: `--cov=app`).
- Backend — workers: `poetry run python start_celery_worker.py`; monitor: `poetry run python start_flower.py`.
- Frontend — dev: `cd bolt-chatbot-ui && npm install && npm run dev`.
- Frontend — build/start: `npm run build && npm start`; lint: `npm run lint`.

## Coding Style & Naming Conventions
- Python: Black + Flake8 (`poetry run black .` / `poetry run flake8`). 4-space indent, `snake_case` for vars/functions, `PascalCase` for classes, modules in `snake_case`.
- TypeScript/React: ESLint (`npm run lint`). 2-space indent, `camelCase` for vars/functions, `PascalCase` for components, files `kebab-case.tsx/ts` or `PascalCase.tsx` for components.
- API routes under `app/api/routes`; keep FastAPI routers small and typed; colocate Pydantic schemas in `app/models/schemas`.

## Testing Guidelines
- Backend: place tests in `langchain_backend/tests/{unit,integration,e2e}`; files `test_*.py`, classes `Test*`, functions `test_*` (see `pytest.ini`). Run with `poetry run pytest -q` and add `--cov=app` for coverage reports.
- Frontend: no test runner is configured; prefer adding lightweight component or integration tests if introduced later. Keep types clean and run `npm run lint` before PRs.

## Commit & Pull Request Guidelines
- Commits: concise, imperative subject (≤72 chars). Examples: `fix ui delete`, `add enhancement plan`. Group related changes.
- PRs: include purpose/summary, linked issue(s), testing notes, migration notes (if DB), and screenshots/GIFs for UI changes. Ensure CI passes, lint clean, and backend tests green.

## Security & Configuration Tips
- Never commit secrets. Use `.env` (root, frontend, backend). If adding a setting, update `langchain_backend/.env.example` and reference it in docs.
- Services: Redis required for caching/Celery; default DB is SQLite (PostgreSQL in prod). Document new environment or service dependencies in `README.md`.


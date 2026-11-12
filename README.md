# ApplyTrack

![CI](https://github.com/MohanaSrinivas/applytrack/actions/workflows/ci.yml/badge.svg?branch=main)

A full-stack job application tracker for working-student, internship, and graduate-role searches. It brings application records and pipeline status into one personal workspace and demonstrates a React client, authenticated REST API, relational persistence, and a repeatable local Docker setup.

**Portfolio scope:** a learning and demonstration project, not a hosted service or production multi-tenant product. No hosted demo is currently provided.

## At a glance

- Register and sign in with a password-hashed account and signed JWT access token.
- Create, view, update, and delete applications through the API; retrieve status totals from the statistics endpoint.
- Keep each account’s application queries scoped to its authenticated user ID.
- Explore the API through generated OpenAPI documentation.

## Architecture

```mermaid
flowchart LR
  Person[User] --> Browser[Browser]
  Browser --> UI[React 19 and TypeScript<br/>Vite development server]
  UI -->|JSON and bearer token| API[FastAPI<br/>Pydantic and SQLAlchemy]
  API -->|SQL| DB[(PostgreSQL 16<br/>Docker Compose)]
  API --> Health[/health]
```

In isolated backend tests, SQLite replaces PostgreSQL. The Compose setup runs PostgreSQL, the API, and the Vite development server on the local machine.

## Stack and checks

React 19 · TypeScript · Vite · Vitest · Python 3.12 · FastAPI · SQLAlchemy · Pydantic · PostgreSQL · Docker Compose · pytest · Ruff · GitHub Actions

The `main` CI workflow checks the backend with Ruff and pytest and the frontend with `npm ci`, tests, build, and npm audit. Frontend tests cover registration/token persistence, saved-application rendering/sign-out, and API request headers, success, and error handling. Backend tests use SQLite and do not require a PostgreSQL service.

## Run with Docker Compose

Requirements: Docker Desktop or Docker Engine with the Compose v2 plugin.

```bash
cp .env.example .env
```

On Windows PowerShell, use `Copy-Item .env.example .env` instead. For local use, replace the example `POSTGRES_PASSWORD` and `JWT_SECRET` values in `.env` before starting.

```bash
docker compose up --build
```

Open the app at [http://localhost:5173](http://localhost:5173), the API docs at [http://localhost:8000/docs](http://localhost:8000/docs), and the health check at [http://localhost:8000/health](http://localhost:8000/health). Stop the stack with `docker compose down`; add `-v` only when you intend to remove the local database volume.

## Run without Docker

Requirements: Python 3.12, Node.js 20 or newer, and npm.

From the repository root, create and activate a Python virtual environment, then install the backend requirements:

```bash
python -m venv .venv
# macOS/Linux: source .venv/bin/activate
# Windows PowerShell: .\.venv\Scripts\Activate.ps1
python -m pip install -r backend/requirements.txt
```

Start the backend with a local SQLite database:

```bash
# macOS/Linux
DATABASE_URL=sqlite:///./applytrack.db JWT_SECRET=local-development-secret python -m uvicorn app.main:app --app-dir backend --reload
# Windows PowerShell
$env:DATABASE_URL = "sqlite:///./applytrack.db"; $env:JWT_SECRET = "local-development-secret"; python -m uvicorn app.main:app --app-dir backend --reload
```

In a second terminal, install and run the frontend:

```bash
cd frontend
npm ci
npm run dev
```

## Run checks

Backend, from the repository root:

```bash
cd backend
python -m pip install -r requirements.txt
ruff check app tests
```

Run the backend tests:

```bash
# macOS/Linux/Git Bash
PYTHONPATH=. pytest -q
# Windows PowerShell
$env:PYTHONPATH = "."; python -m pytest -q
```

Frontend:

```bash
cd frontend
npm ci
npm test
npm run build
npm audit
```

## API surface

- `POST /auth/register` and `POST /auth/login`
- `GET` and `POST /applications`
- `GET`, `PATCH`, and `DELETE /applications/{id}`
- `GET /applications/statistics`
- `GET /health`
- OpenAPI UI: `/docs`; schema: `/openapi.json`

Application statuses are `saved`, `applied`, `interview`, `offer`, and `rejected`. SQLAlchemy currently creates the schema at startup; there is no migration system. The access token is stored in browser `localStorage`, so an XSS vulnerability could expose it. Password reset, email verification, refresh-token rotation, rate limiting, pagination, and a production deployment are not implemented.

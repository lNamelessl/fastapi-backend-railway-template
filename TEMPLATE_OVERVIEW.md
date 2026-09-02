# FastAPI Backend Template

A production-ready, backend-only [FastAPI](https://fastapi.tiangolo.com) API template on [Railway](https://railway.app), based on the [official Full Stack FastAPI Template](https://github.com/tiangolo/full-stack-fastapi-template) by [@tiangolo](https://github.com/tiangolo) with all frontend elements removed.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.app/new?github_url=https://github.com/lNamelessl/fastapi-backend-railway-template)

## What's Included

* ⚡ **FastAPI** with automatic interactive docs (Swagger UI / ReDoc).
  * 🧰 **SQLModel** for typed SQL database interactions (ORM).
  * 🔍 **Pydantic** for data validation and settings management.
  - 💾 **PostgreSQL** as the SQL database, with [Alembic](https://alembic.sqlalchemy.org) migrations.
* 🔒 Secure password hashing (Argon2/bcrypt via pwdlib).
* 🔑 JWT authentication with login, registration, password recovery, and user management endpoints.
* 📫 Email-based password recovery (pluggable SMTP).
* 🐘 Automatic database migrations and superuser seeding on every deploy.
* 🩺 A `/health` endpoint wired to Railway's health check.

## One-Click Deploy

Click the deploy button above. Railway clones this repository and provisions two services:

1. **PostgreSQL** — a Railway database plugin.
2. **backend** — this repo, built with the root `Dockerfile` and started with the `railway.json` start command.

On every container start, `backend/scripts/prestart.sh` runs `alembic upgrade head` and seeds the first superuser before the server binds Railway's `PORT`. Railway polls `GET /health` and marks the deployment healthy once the API responds.

After the deploy, generate a public domain for the backend service (Settings > Networking > Generate Domain). The API is then live at `https://<your-domain>/docs`.

# Deploy and Host

Deploy a backend-only FastAPI API with PostgreSQL on Railway in one click: the template provisions a **backend** service (Docker, Python 3.14 + uv) and a **PostgreSQL** database, wires `DATABASE_URL` to the database reference, runs Alembic migrations plus superuser seeding automatically on each deploy, binds to Railway's dynamic `PORT`, and passes the `GET /health` health check before going live.

## About Hosting

Hosting this FastAPI backend on Railway gives you:

* A **backend** service built from the repo's root `Dockerfile` (no Nixpacks/Buildpack guesswork), started with `bash scripts/prestart.sh && exec fastapi run --host 0.0.0.0 --port ${PORT:-8000} --workers 4`, health-checked at `/health` with an ON_FAILURE restart policy (10 retries).
* A **PostgreSQL** database plugin with a connection URL exposed as the `DATABASE_URL` reference variable. The backend validates and rewrites it for the psycopg driver automatically.
* Automatic migrations and seeding: `prestart.sh` runs `alembic upgrade head` and `app/initial_data.py` (creates the superuser from `FIRST_SUPERUSER` / `FIRST_SUPERUSER_PASSWORD`) on every deployment start, so schema and code ship together.
* Environment variables required on the backend service: `DATABASE_URL` (reference: `${{Postgres.DATABASE_URL}}`), `SECRET_KEY`, `FIRST_SUPERUSER`, `FIRST_SUPERUSER_PASSWORD`, `PROJECT_NAME`. Optional: `FRONTEND_HOST` (CORS + email links), `SMTP_*` / `EMAILS_FROM_EMAIL` (outbound email), `SENTRY_DSN`.
* Post-deploy step: generate a Railway domain for the backend service and point `FRONTEND_HOST` (and your own frontend) at the API URL.

## Why Deploy

* The repo is backend-only: no frontend build step, no reverse proxy, no Node toolchain — deploys are a single small Docker image.
* Config-as-code: `railway.json` in the repo pins the builder, start command, health check, and restart policy, so every deploy behaves identically whether triggered by the CLI or by a git push.
* The official template it derives from is battle-tested by tiangolo and the FastAPI team; the Railway packaging fixes (dynamic `PORT`, container-start migrations, redirect-free `/health`) are already applied.
* Deploys are repeatable and stateless; the database is the only stateful service, managed by Railway with daily backups and easy resets.

## Common Use Cases

* REST API backends for mobile apps, SPAs, or separately hosted frontends.
* Internal tools and admin APIs with JWT auth and a ready-made user/role model.
* Rapid prototyping of product APIs with typed ORM models and Alembic migrations already wired.
* A starting point for microservices that need authentication, email flows, and PostgreSQL.

## Dependencies for

### Deployment Dependencies

* **PostgreSQL** (Railway database plugin) — required. Provides `DATABASE_URL` via the `${{Postgres.DATABASE_URL}}` reference. The backend connects with the psycopg driver and runs Alembic migrations against it on boot.
* **backend service environment variables** — required: `DATABASE_URL`, `SECRET_KEY`, `FIRST_SUPERUSER`, `FIRST_SUPERUSER_PASSWORD`, `PROJECT_NAME`. Optional: `FRONTEND_HOST`, `SMTP_HOST`, `SMTP_USER`, `SMTP_PASSWORD`, `EMAILS_FROM_EMAIL`, `SENTRY_DSN`. Do not set `FASTAPI_ENV`.
* **Railway networking** — a generated public domain (or custom domain) for the backend service to expose the API and docs.

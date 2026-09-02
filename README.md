# FastAPI Backend Template (Railway)

A backend-only API template based on the [official Full Stack FastAPI Template](https://github.com/tiangolo/full-stack-fastapi-template) by [@tiangolo](https://github.com/tiangolo), with all frontend elements removed and the deployment adapted for [Railway](https://railway.app).

[![Test Backend](../../actions/workflows/test-backend.yml/badge.svg)](../../actions/workflows/test-backend.yml)

## Technology Stack and Features

- ⚡ [**FastAPI**](https://fastapi.tiangolo.com) for the Python backend API.
  - 🧰 [SQLModel](https://sqlmodel.tiangolo.com) for the Python SQL database interactions (ORM).
  - 🔍 [Pydantic](https://docs.pydantic.dev), used by FastAPI, for the data validation and settings management.
  - 💾 [PostgreSQL](https://www.postgresql.org) as the SQL database.
- 🚄 One-click deployment to [Railway](https://railway.app), with [`railway.json`](./railway.json) config-as-code, health checks, and automatic database migrations on every deploy.
- 🐋 [Docker Compose](https://www.docker.com) for local services (PostgreSQL, Adminer, Mailpit).
- 🔒 Secure password hashing by default.
- 🔑 JWT (JSON Web Token) authentication.
- 📫 Email-based password recovery.
- 📬 [Mailpit](https://mailpit.axllent.org) for local email testing during development.
- ✅ Tests with [Pytest](https://pytest.org).
- 🏭 CI (continuous integration) based on GitHub Actions.

## Deploy to Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.app/new?github_url=https://github.com/lNamelessl/fastapi-backend-railway-template)

1. Click the deploy button above (or go to [railway.app/new](https://railway.app/new) and select this repository).
2. Railway provisions a **PostgreSQL** database and a **backend** service, builds it with the repo's `Dockerfile`, and starts it.
3. On every start (and before every deploy is marked healthy), the container runs `scripts/prestart.sh`, which applies Alembic migrations and seeds the first superuser from `FIRST_SUPERUSER` / `FIRST_SUPERUSER_PASSWORD`.
4. Railway probes `GET /health` until the service is up, then you can generate a public domain.

After deploying, see [deployment.md](./deployment.md) for the environment variable reference and post-deploy steps.

## How to Use It

Click the **Use this template** button at the top of this page to create a new repository, or deploy it directly to Railway with the button above.

## Backend Development

Backend docs: [backend/README.md](./backend/README.md).

## Deployment

Railway deployment guide: [deployment.md](./deployment.md).

## Development

General development docs: [development.md](./development.md).

This includes the local FastAPI workflow, Docker Compose services, `.env` configuration, and more.

## License

The original Full Stack FastAPI Template is licensed under the terms of the MIT license. This backend-only adaptation keeps the same license.

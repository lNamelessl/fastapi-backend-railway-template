# FastAPI Project - Deployment

Deploy the project to [Railway](https://railway.app). The repo ships a `railway.json` (config-as-code) and a root `Dockerfile` so Railway can build and run the backend without extra configuration.

## One-Click Deploy

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.app/new?github_url=https://github.com/lNamelessl/fastapi-backend-railway-template)

Clicking the button (or opening `https://railway.app/new?github_url=https://github.com/lNamelessl/fastapi-backend-railway-template`) creates a new Railway project from this repository and prompts you to provision:

* **backend**: this repo, built with the `Dockerfile` and started with the `railway.json` `startCommand`.
* **PostgreSQL**: a Railway database plugin. Its `DATABASE_URL` reference is injected into the backend automatically.

## What Happens on Deploy

1. Railway builds the Docker image from the repo root `Dockerfile` (Python 3.14 + [uv](https://docs.astral.sh/uv/)).
2. On container start, `backend/scripts/prestart.sh` runs **before** the server binds the port. It applies Alembic migrations (`alembic upgrade head`) and seeds the first superuser (`app/initial_data.py`) using `FIRST_SUPERUSER` and `FIRST_SUPERUSER_PASSWORD`.
3. The server binds to Railway's injected `PORT` environment variable.
4. Railway polls the `healthcheckPath` (`GET /health`) and marks the deployment healthy once it returns `200`.

## Environment Variables

The backend service needs these variables:

* `DATABASE_URL`: set it to the database reference `DATABASE_URL=${{Postgres.DATABASE_URL}}` so it always points at your Railway Postgres.
* `SECRET_KEY`: a secret key used to sign security tokens. Generate one with `openssl rand -hex 32`.
* `FIRST_SUPERUSER`: the email address of the first superuser, e.g. `admin@example.com`.
* `FIRST_SUPERUSER_PASSWORD`: the password of the first superuser. Generate a secure value with `openssl rand -base64 24`.
* `PROJECT_NAME`: the name of the project, used in the API documentation and emails.

Optional variables:

* `FRONTEND_HOST`: the URL of the (separate) frontend that will call this API. Used for CORS and for the links inside emails. Defaults to `http://localhost:5173`; set it to your frontend's public URL.
* `SMTP_HOST`, `SMTP_USER`, `SMTP_PASSWORD`, `EMAILS_FROM_EMAIL`: to enable outgoing emails.
* `SENTRY_DSN`: to enable Sentry error reporting.

**Note**: do not set `FASTAPI_ENV` on Railway. It only accepts `development` (used for local runs); setting anything else fails validation and crash-loops the container.

**Note**: the seed script only creates the superuser if it does not exist yet. Changing `FIRST_SUPERUSER_PASSWORD` later will not update the already-seeded user; update the user through the API or reset the database volume.

## Post-Deploy Steps

1. In your Railway project, open the **backend** service and generate a public domain (Settings > Networking > Generate Domain).
2. Set `FRONTEND_HOST` to the public URL of your frontend (or to the backend's own domain if you only need the API reachable for browser-based docs) and redeploy if needed.
3. Check the interactive API docs at `https://your-domain/docs`.

## CLI Deployment

You can also deploy from your machine with the [Railway CLI](https://docs.railway.app/develop/cli):

```bash
railway init --name my-project
railway add -d postgres
railway add -s backend
railway variables --set 'DATABASE_URL=${{Postgres.DATABASE_URL}}' \
  --set "SECRET_KEY=$(openssl rand -hex 32)" \
  --set "FIRST_SUPERUSER=admin@example.com" \
  --set "FIRST_SUPERUSER_PASSWORD=$(openssl rand -hex 16)" \
  --service backend
railway up --service backend
railway domain
```

## Continuous Deployment

Connecting the GitHub repository in Railway enables automatic deploys: every push to the tracked branch builds a new image with the `Dockerfile` and deploys it with the `railway.json` settings (start command, health check, restart policy). Migrations run on every deployment start, so schema changes ship together with the code.

## Local Deployment with Docker Compose

For local integration testing you can run the whole stack with Docker Compose (PostgreSQL, Adminer, Mailpit, backend); see [development.md](./development.md).

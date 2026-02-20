# CI/CD + Docker Deploy (Windows Local PC)

This repo uses GitHub Actions to:

- **CI**: build/test backend + frontend on PRs and pushes
- **CD**: on push to `main`, publish Docker images to **GHCR** and (optionally) deploy

The easiest way to deploy to a **Windows local PC** is a **self-hosted runner** (no inbound SSH required).

---

## 1) What CI/CD runs

Workflow: `.github/workflows/ci-cd.yml`

- On `pull_request` to `main`
  - Backend: `npm ci`, typecheck (if present), build, tests (if present)
  - Frontend: `npm ci`, build, tests (if present)

- On `push` to `main`
  - Publishes Docker images to GHCR:
    - `ghcr.io/<owner>/<repo>-backend:latest` and `:<sha>`
    - `ghcr.io/<owner>/<repo>-frontend:latest` and `:<sha>`

- Optional deploy methods:
  - **Self-hosted Windows runner** (best for local PC)
  - **SSH deploy** (only if the target machine is reachable over SSH)

---

## 2) Self-hosted runner deploy (recommended)

### Prereqs on the Windows PC

- Docker Desktop installed and running
- `docker compose` works in PowerShell (`docker compose version`)

### Add a self-hosted runner (one-time)

1. In GitHub repo → **Settings** → **Actions** → **Runners** → **New self-hosted runner**
2. Choose **Windows x64**
3. Follow the displayed commands on your PC
4. When asked for labels, keep defaults (must include `self-hosted` and `windows`)

### Required GitHub Secrets

Repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Required:

- `JWT_SECRET` = a long random secret (min 32 chars)

Optional (only if you want CI to write these into `.env` on the PC):

- `DATABASE_URI`
- `REDIS_URL`
- `FRONTEND_URL` (preferred)
- `CORS_ORIGIN` (fallback; used only if `FRONTEND_URL` is not set)
- `LOG_LEVEL`

### Required GitHub Variables

Repo → **Settings** → **Secrets and variables** → **Actions** → **Variables** → **New repository variable**

- `ENABLE_SELF_HOSTED_DEPLOY` = `true`

### First deploy

- Push a commit to `main`
- Watch **Actions** → workflow run
- The job **Deploy (Self-hosted Windows runner)** will:
  - login to GHCR
  - write `.env` from secrets
  - run `docker compose -f docker-compose.prod.yml pull`
  - run `docker compose -f docker-compose.prod.yml up -d --remove-orphans`

---

## 3) SSH deploy (only for reachable servers)

If you deploy to a Windows/Linux machine reachable by SSH, configure:

- `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_SSH_KEY`
- Optional: `DEPLOY_OS=windows` and `DEPLOY_PATH`

If you also set `JWT_SECRET` (and optional env secrets), the workflow will **write a `.env`** on the server during deploy.

---

## 4) Local manual run (no CI)

On any machine with Docker:

```powershell
# from repo root
# create .env with at least JWT_SECRET, DATABASE_URI (prod), etc.
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d --remove-orphans
```

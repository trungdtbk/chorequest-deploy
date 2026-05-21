# chorequest-deploy

Orchestrates the ChoreQuest backend and frontend services via Docker Compose.

The backend and frontend live in their own repos and are included here as git submodules. This repo contains only deployment config — Docker Compose, nginx, and environment setup.

## Repos

| Repo | Description |
|---|---|
| [chorequest-backend](https://github.com/trungdtbk/chorequest-backend) | FastAPI backend |
| [chorequest-frontend](https://github.com/trungdtbk/chorequest-frontend) | React frontend |
| [chorequest-deploy](https://github.com/trungdtbk/chorequest-deploy) | This repo — deployment config |

## Structure

```
chorequest-deploy/
├── backend/                  # submodule → chorequest-backend
├── frontend/                 # submodule → chorequest-frontend
├── nginx/
│   └── nginx.conf            # reverse proxy config
├── docker-compose.yml        # development / production orchestration
├── .env.example              # required environment variables
├── Makefile                  # convenience shortcuts
└── .gitmodules
```

## Prerequisites

- Docker
- Docker Compose (plugin — `docker compose`, not `docker-compose`)
- Git

## Quick start

### 1. Clone with submodules

```bash
git clone --recurse-submodules https://github.com/trungdtbk/chorequest-deploy
cd chorequest-deploy
```

### 2. Configure environment

```bash
cp .env.example .env
nano .env   # fill in your values
```

See [Environment variables](#environment-variables) below for details.

### 3. Build and run

```bash
docker compose up -d --build
```

The app is now running at `http://localhost:8122`.

## Environment variables

Copy `.env.example` to `.env` and fill in the values:

```bash
# Backend
SECRET_KEY=                          # generate with: python -c "import secrets; print(secrets.token_hex(32))"
TZ=
DATABASE_URL=sqlite:///./data/chores_os.db  # optional
```

## Common commands

```bash
# Start in background
docker compose up -d

# Start with live logs
docker compose up

# Rebuild images (after code changes)
docker compose up -d --build

# Stop everything
docker compose down

# View logs
docker compose logs -f

# View logs for one service
docker compose logs -f backend

# Restart a service
docker compose restart backend

# Open a shell in the backend container
docker compose exec backend bash
```

## Updating to latest code

After new commits are merged in the backend or frontend repos, update the submodule pointers and redeploy:

```bash
# Pull latest commits for all submodules
git submodule update --remote

# Commit the updated pointers
git add backend frontend
git commit -m "chore: bump submodules to latest"
git push

# Redeploy
docker compose up -d --build
```

Or with the Makefile:

```bash
make update
docker compose up -d --build
```

## Deploying to a new server

Assuming Docker and Docker Compose are already installed:

```bash
git clone --recurse-submodules https://github.com/trungdtbk/chorequest-deploy
cd chorequest-deploy
cp .env.example .env
nano .env
docker compose up -d --build
```

## Architecture

```
browser
  │
  ▼
nginx :80  (port 8122 on host)
  ├── /api/*  →  backend:8000  (FastAPI)
  ├── /ws/*   →  backend:8000  (WebSocket)
  └── /*      →  static files  (React build)
```

## Data persistence

The SQLite database is stored in `./data/` on the host and mounted into the backend container. This directory is created automatically on first run and persists across container restarts and rebuilds.

To back up the database:

```bash
cp data/chores_os.db data/chores_os.db.backup
```
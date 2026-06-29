# URL Shortener

Go-based URL shortener with PostgreSQL, Redis, cookie-based auth, and Docker Compose dev/prod setups.

## Quick Start

### Prerequisites

- Docker + Docker Compose

### Development

```bash
# 1. Set up environment
cp .env.example .env

# 2. Start Postgres + Redis (Docker)
make dev-up

# 3. Run migrations
make dev-migrate-up

# 4. Run backend (requires Go + air)
make run-backend
```

> Backend runs on `http://localhost:8080`. Postgres/Redis run in Docker — no local install needed.

### Production

```bash
# 1. Set up production environment
cp .env.example .env.prod
# Edit .env.prod — set DB_HOST_GO=postgres, REDIS_HOST_GO=redis

# 2. Build & start everything in Docker
make prod-up

# 3. Run migrations
make prod-migrate-up

# 4. Watch logs
make prod-logs
```

> Everything runs in Docker — no Go, Postgres, or Redis on the host.

## Environment

| Variable | Dev (`.env`) | Prod (`.env.prod`) |
|---|---|---|
| `DB_HOST_GO` | `localhost` | `postgres` |
| `REDIS_HOST_GO` | `localhost` | `redis` |
| `RESEND_API` | your Resend API key | your Resend API key |

`.env` → local dev (app on host, DB in Docker).  
`.env.prod` → Docker everything (hostnames = Docker service names).

Full variable list in [`.env.example`](.env.example).

## Makefile

| Command | What it does |
|---|---|
| `make dev-up` | Start dev containers (postgres, redis) |
| `make dev-down` / `dev-down-force` | Stop / stop+volumes |
| `make dev-logs` | Stream dev logs |
| `make dev-migrate-up/down/version` | Database migrations |
| `make run-backend` | Run Go app locally with `air` |
| `make lint-backend` / `format-backend` | Lint / format Go code |
| `make prod-up` | Build and start prod stack |
| `make prod-down` / `prod-down-force` | Stop / stop+volumes |
| `make prod-logs` | Follow prod app logs |
| `make prod-migrate-up/down/version` | Prod migrations |

## API Reference

See [API.md](API.md) for full endpoint docs with curl examples and OpenAPI spec.

## Tech Stack

Go, PostgreSQL, Redis, Docker Compose, `rs/cors`, `migrate`, Resend (email).

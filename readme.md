# URL Shortener

A Go-based URL shortener backend with user authentication, email verification, password reset, URL management, Redis, PostgreSQL, and Docker-based development and production setups.

## Features

- User registration, login, logout, refresh tokens
- Email verification
- Forgot-password and reset-password flow
- Create, list, fetch, and delete short URLs
- Redirect from short slug to original URL
- PostgreSQL for persistent storage
- Redis for caching/session-related use cases
- SMTP/email integration through Resend
- Structured logging with `slog`
- CORS support for a frontend running on `http://localhost:3000`
- Docker Compose setups for dev and prod
- DB migrations with `migrate`
- Basic profiling endpoints via `pprof`

## Project Structure

```text
.
├── backend
│   ├── cmd/server            # App entrypoint
│   ├── internal
│   │   ├── app               # Router, middleware, app wiring
│   │   ├── config            # Configuration loading
│   │   ├── domain            # Domain models and errors
│   │   ├── handler           # HTTP handlers
│   │   ├── logger            # Logger setup
│   │   ├── repository        # Data access layer
│   │   └── service           # Business logic
│   ├── migrations            # SQL migrations
│   └── Dockerfile
├── dev-compose.yml
├── prod-compose.yml
├── Makefile
└── .env.example
```

## Tech Stack

- Go
- PostgreSQL
- Redis
- Docker / Docker Compose
- Resend for emails
- `rs/cors` for CORS handling
- `migrate` for database migrations

## Requirements

- Go installed locally if you want to run without Docker
- Docker and Docker Compose
- PostgreSQL
- Redis
- A Resend API key if you use email features

## Environment Variables

Copy `.env.example` to `.env` for development and fill in the values.
For production, use `.env.prod`.

Typical variables used by the project:

```env
APP_PORT=
DB_HOST=
DB_PORT=
DB_USER=
DB_PASSWORD=
DB_NAME=
REDIS_HOST=
REDIS_PORT=
JWT_SECRET=
RESEND_API_KEY=
FROM_EMAIL=
```

> The exact set may vary depending on your `config.go` and app wiring.

## Running Locally

### Using Docker Compose

Start the development stack:

```bash
make dev-up
```

View logs:

```bash
make dev-logs
```

Stop containers:

```bash
make dev-down
```

Stop and remove volumes:

```bash
make dev-down-force
```

### Running the backend directly

```bash
make run-backend
```

This assumes your local environment and dependencies are already set up.

## Database Migrations

Run pending migrations in dev:

```bash
make dev-migrate-up
```

Rollback the last migration:

```bash
make dev-migrate-down
```

Show current migration version:

```bash
make dev-migrate-version
```

The same commands exist for production with the `prod-` prefix.

## Makefile Commands

### Development

- `make dev-up` — start dev containers
- `make dev-down` — stop dev containers
- `make dev-down-force` — stop dev containers and remove volumes
- `make dev-logs` — show dev logs
- `make dev-migrate-up` — run pending migrations
- `make dev-migrate-down` — rollback last migration
- `make dev-migrate-version` — show migration version
- `make run-backend` — run the Go API server locally
- `make lint-backend` — run `golangci-lint`
- `make format-backend` — format Go code with `gofumpt`

### Production

- `make prod-up` — start prod containers
- `make prod-down` — stop prod containers
- `make prod-down-force` — stop prod containers and remove volumes
- `make prod-logs` — follow prod logs
- `make prod-migrate-up` — run pending migrations
- `make prod-migrate-down` — rollback last migration
- `make prod-migrate-version` — show migration version

## API Overview

Base path: `/api/v1`

### Auth

- `POST /api/v1/auth/register` — register a new user
- `POST /api/v1/auth/login` — log in
- `POST /api/v1/auth/logout` — log out
- `POST /api/v1/auth/refresh` — refresh tokens
- `POST /api/v1/auth/verify-email` — verify email
- `POST /api/v1/auth/forgot-password` — request password reset
- `POST /api/v1/auth/reset-password` — reset password
- `GET /api/v1/auth/me` — get current user
- `DELETE /api/v1/auth/me` — delete current user

### URL management

- `POST /api/v1/urls` — create a short URL
- `GET /api/v1/urls` — list user URLs
- `GET /api/v1/urls/{slug}` — get a single URL by slug
- `DELETE /api/v1/urls/{slug}` — delete a URL by slug

### Redirect

- `GET /{slug}` — redirect to the original URL

### Health

- `GET /health` — health check

### Debug

- `GET /debug/pprof/` — pprof index
- `GET /debug/pprof/profile` — CPU profile

## CORS

CORS is configured to allow requests from:

```text
http://localhost:3000
```

Allowed methods include `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, and `PATCH`.

## Notes

- Authentication-protected endpoints are wrapped with middleware.
- The app uses structured logging and middleware around the router.
- Dev and prod compose files both include PostgreSQL, Redis, and migration support.
- The production compose file also builds and runs the Go application containe
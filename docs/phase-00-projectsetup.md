# Phase 0 — Project Setup

> **Status:** Completed
> **Date:** 2026-07-18
>
> Single source of truth for environment setup: tech stack, project
> structure, and development workflow.
>
> For MVP design (API, DB schema, PRD), see
> [phase-01-single-server.md](phase-01-single-server.md).

---

## 1. Technology Decisions

| Decision               | Choice           | Why                                                          |
| ---------------------- | ---------------- | ------------------------------------------------------------ |
| Language               | Go 1.22+         | Compiled, static types (familiar from C++). First-class concurrency for later phases. |
| HTTP Router            | Chi v5           | stdlib-compatible, middleware support, no framework lock-in.  |
| Database               | PostgreSQL 16    | Industry standard. Rich indexing, JSONB, EXPLAIN ANALYZE.    |
| DB Driver              | pgx v5           | Pure Go, high performance, context-aware.                    |
| Migrations             | golang-migrate   | CLI + library. Up/down migrations. Version controlled.       |
| Auth                   | JWT (golang-jwt) | Stateless, simple for MVP. Can add refresh tokens later.     |
| Config                 | Environment vars | 12-factor app. Loaded via .env in dev.                       |
| Containerization       | Docker + Compose | Reproducible. PostgreSQL in a container = no local install.  |
| UUID generation        | google/uuid      | Standard, no DB dependency for ID generation.                |
| Password hashing       | bcrypt           | Industry standard, built into Go stdlib (golang.org/x/crypto).|
| Input validation       | Custom + regex   | Keep it simple. No validation framework for MVP.             |
| API Documentation      | swaggo/swag      | Auto-generates OpenAPI spec from Go annotations. Swagger UI for interactive testing. |
| Swagger UI             | http-swagger     | Serves Swagger UI at `/swagger/`. Test endpoints in browser. |

---

## 2. Repository Structure

```
system-design-lab/
├── cmd/
│   └── server/
│       └── main.go                  # Entry point
├── internal/
│   ├── config/
│   │   └── config.go                # Env-based configuration
│   ├── handler/
│   │   ├── auth.go                  # Register, Login handlers
│   │   ├── monitor.go               # CRUD handlers
│   │   ├── check.go                 # Manual check handler
│   │   └── response.go              # Standardized JSON responses
│   ├── middleware/
│   │   ├── auth.go                  # JWT verification
│   │   ├── logging.go               # Request logging
│   │   └── recovery.go              # Panic recovery
│   ├── model/
│   │   ├── user.go                  # User struct
│   │   ├── monitor.go               # Monitor struct
│   │   └── check_result.go          # CheckResult struct
│   ├── repository/
│   │   ├── user.go                  # User DB operations
│   │   ├── monitor.go               # Monitor DB operations
│   │   └── check_result.go          # CheckResult DB operations
│   ├── service/
│   │   ├── auth.go                  # Auth business logic
│   │   ├── monitor.go               # Monitor business logic
│   │   └── checker.go               # URL health check logic
│   └── validator/
│       └── validator.go             # Input validation helpers
├── migrations/
│   ├── 000001_create_users.up.sql
│   ├── 000001_create_users.down.sql
│   ├── 000002_create_monitors.up.sql
│   ├── 000002_create_monitors.down.sql
│   ├── 000003_create_check_results.up.sql
│   └── 000003_create_check_results.down.sql
├── docs/
│   ├── phase-00-projectsetup.md      # ← You are here
│   ├── architecture/
│   ├── adr/
│   └── journal/
├── challenges/
│   ├── phase-XX-challenge.md          # Question templates (read-only)
│   └── answers/
│       └── phase-XX-answers.md        # Your answers go here
├── prompts/                           # Reusable workflow prompts
├── docker-compose.yml
├── Dockerfile
├── Makefile
├── go.mod
├── go.sum
├── .env.example
├── .gitignore
└── README.md
```

---

## 3. Development Workflow

```
make build        → Compile the Go binary
make run          → docker compose up (app + postgres)
make stop         → docker compose down
make test         → Run all tests
make migrate-up   → Apply pending migrations
make migrate-down → Rollback last migration
make lint         → Run golangci-lint
make swagger      → Regenerate OpenAPI spec from annotations
```

---

## 4. What's Next

1. Scaffold the Go project (`go mod init`, folder structure)
2. Write Dockerfile + docker-compose.yml
3. Write Makefile
4. Create .env.example and .gitignore
5. First commit → Phase 0 complete
6. Attempt [Phase 1 Challenge](../challenges/phase-01-challenge.md)
7. Begin Phase 1 — design and implement the MVP

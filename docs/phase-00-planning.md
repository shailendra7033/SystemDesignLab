# Phase 0 — Planning

> **Status:** In Progress
> **Date Started:** 2026-07-18

---

## 1. Product Requirements Document (PRD)

### What Are We Building?

A **URL Monitoring Platform** — a backend service that lets users register
URLs to monitor and receive visibility into uptime, latency, and availability.

### Who Is It For?

- Developers who want to monitor their side projects
- Small teams who need basic uptime visibility
- (In reality: **ourselves**, as a learning vehicle for system design)

### MVP Scope (Phase 1)

| Feature                  | Priority | Notes                              |
| ------------------------ | -------- | ---------------------------------- |
| User registration/login  | P0       | JWT-based auth                     |
| Create a monitor         | P0       | URL + check interval + expected status |
| List monitors            | P0       | Paginated, filtered by user        |
| Get monitor details      | P0       |                                    |
| Update monitor           | P1       |                                    |
| Delete monitor           | P0       | Soft delete? Hard delete for MVP   |
| Manual health check      | P0       | Hit URL, record result             |
| View check history       | P1       | Last N checks for a monitor        |

### Out of Scope for MVP

- Automated periodic checks (Phase 2)
- Notifications/alerts (Phase 3)
- Dashboard UI (no frontend — API only)
- Multi-region checking
- SSL certificate monitoring
- Custom headers / auth for monitored URLs

### Non-Functional Requirements

| Requirement      | Target (MVP)                          |
| ---------------- | ------------------------------------- |
| Response time    | < 200ms p95 for CRUD endpoints        |
| Availability     | Not a concern for MVP (single server) |
| Data retention   | Keep all check results (no cleanup yet)|
| Auth             | JWT with 24h expiry                   |
| API format       | REST, JSON, versioned (/api/v1/)      |

---

## 2. High-Level Architecture (Phase 1)

```
┌──────────┐       ┌──────────────────┐       ┌──────────────┐
│  Client  │──────▶│  Go API Server   │──────▶│  PostgreSQL  │
│ (curl/   │◀──────│  (Chi router)    │◀──────│              │
│  httpie) │       │  Port 8080       │       │  Port 5432   │
└──────────┘       └──────────────────┘       └──────────────┘
                          │
                          ▼
                   ┌──────────────┐
                   │  URL Check   │  (inline, during
                   │  (net/http)  │   manual check)
                   └──────────────┘
```

**Everything runs in Docker Compose.** The Go app and PostgreSQL each get
their own container. No external dependencies yet.

---

## 3. Technology Decisions

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

## 4. Repository Structure

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
│   ├── phase-00-planning.md         # ← You are here
│   ├── architecture/
│   ├── adr/
│   └── journal/
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

## 5. API Contract (v1)

```
Auth
  POST   /api/v1/auth/register        → 201 Created
  POST   /api/v1/auth/login            → 200 OK (returns JWT)

Monitors (requires auth)
  GET    /api/v1/monitors              → 200 OK (paginated list)
  POST   /api/v1/monitors              → 201 Created
  GET    /api/v1/monitors/{id}         → 200 OK
  PUT    /api/v1/monitors/{id}         → 200 OK
  DELETE /api/v1/monitors/{id}         → 204 No Content

Checks (requires auth)
  POST   /api/v1/monitors/{id}/check   → 200 OK (triggers manual check)
  GET    /api/v1/monitors/{id}/history  → 200 OK (paginated check results)

Health
  GET    /health                        → 200 OK

Docs
  GET    /swagger/*                     → Swagger UI (interactive API docs)
```

### Standardized Error Response

```json
{
  "error": {
    "code": "MONITOR_NOT_FOUND",
    "message": "Monitor with id abc-123 does not exist"
  }
}
```

### Pagination Format

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 42
  }
}
```

---

## 6. Database Schema (Initial)

```sql
-- users
CREATE TABLE users (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email         TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- monitors
CREATE TABLE monitors (
    id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id          UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    url              TEXT NOT NULL,
    name             TEXT NOT NULL,
    check_interval_s INT NOT NULL DEFAULT 300,
    expected_status  INT NOT NULL DEFAULT 200,
    is_active        BOOL NOT NULL DEFAULT true,
    created_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at       TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_monitors_user_id ON monitors(user_id);

-- check_results
CREATE TABLE check_results (
    id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    monitor_id       UUID NOT NULL REFERENCES monitors(id) ON DELETE CASCADE,
    status_code      INT,
    response_time_ms INT NOT NULL,
    is_up            BOOL NOT NULL,
    error            TEXT,
    checked_at       TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_check_results_monitor_checked ON check_results(monitor_id, checked_at DESC);
```

---

## 7. Development Workflow

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

## 8. What's Next

After this planning doc is reviewed:

1. Scaffold the Go project (`go mod init`, folder structure)
2. Write Dockerfile + docker-compose.yml
3. Write Makefile
4. Create migration SQL files
5. Create .env.example and .gitignore
6. First commit → Phase 0 complete
7. Begin Phase 1 — implement the MVP endpoints

---

## 9. Open Questions

- [ ] Should monitor deletion be soft delete (is_deleted flag) or hard delete?
  - **Decision:** Hard delete for MVP. Soft delete adds complexity we don't need yet.
- [ ] Should check history have a retention limit?
  - **Decision:** Keep everything for MVP. We'll address this when storage becomes a measurable problem.
- [ ] JWT secret management?
  - **Decision:** Environment variable for MVP. No vault/secrets manager yet.

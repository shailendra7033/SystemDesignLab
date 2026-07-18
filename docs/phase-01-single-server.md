# Phase 1 — Single Server MVP

> **Status:** Not Started
> **Date:** —
>
> **This is the reference answer for Phase 1.**
> Do NOT read this until you've attempted
> [challenges/phase-01-challenge.md](../challenges/phase-01-challenge.md).

---

## 1. Product Requirements Document (PRD)

### What Are We Building?

A **URL Monitoring Platform** — a backend service that lets users register
URLs to monitor and receive visibility into uptime, latency, and availability.

### Who Is It For?

- Developers who want to monitor their side projects
- Small teams who need basic uptime visibility
- (In reality: **ourselves**, as a learning vehicle for system design)

### MVP Scope

| Feature                  | Priority | Notes                              |
| ------------------------ | -------- | ---------------------------------- |
| User registration/login  | P0       | JWT-based auth                     |
| Create a monitor         | P0       | URL + check interval + expected status |
| List monitors            | P0       | Paginated, filtered by user        |
| Get monitor details      | P0       |                                    |
| Update monitor           | P1       |                                    |
| Delete monitor           | P0       | Hard delete for MVP                |
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

## 2. High-Level Architecture

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

---

## 3. API Contract (v1)

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

## 4. Database Schema

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

## 5. Open Questions

- [x] Should monitor deletion be soft delete or hard delete?
  - **Decision:** Hard delete for MVP. Soft delete adds complexity we don't need yet.
- [x] Should check history have a retention limit?
  - **Decision:** Keep everything for MVP. Address when storage becomes a measurable problem.
- [x] JWT secret management?
  - **Decision:** Environment variable for MVP. No vault/secrets manager yet.

---

## 6. Metrics Baseline

Measure these after Phase 1 is complete — they become the baseline for all future phases:

- Response time for all endpoints (curl + time)
- Manual check latency
- Query times with `EXPLAIN ANALYZE`
- Container resource usage (`docker stats`)

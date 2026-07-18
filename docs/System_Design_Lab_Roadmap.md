# System Design Lab - Roadmap

## Project 01

URL Monitoring Platform

### Problem Statement

Provide a platform where users can monitor websites and receive
visibility into uptime, latency, and availability.

The product itself is intentionally simple. The learning comes from
evolving the architecture as new problems appear.

---

## Phase 0 — Planning & Environment Setup

**Deliverables**

- Product Requirements Document
- High-Level Architecture diagram
- Technology Stack finalized (Go + Chi + PostgreSQL)
- Repository structure with Go modules
- Dockerfile + docker-compose.yml (Go app + PostgreSQL)
- Makefile for common commands (build, run, test, migrate)
- CI pipeline (GitHub Actions: lint + test)
- `.env.example` for configuration

**Learn**

- Requirement Analysis
- HLD vs LLD
- API-first thinking
- Why Docker? (reproducibility, dependency isolation, scaling prep)
- 12-factor app basics (config via env vars)

**System Design Concepts**

- Monolithic architecture
- Environment parity (dev = prod)

---

## Phase 1 — Single Server MVP

**Features**

- User Registration & Login (JWT-based auth)
- Create Monitor (URL, check interval, expected status code)
- List Monitors (with pagination)
- Get Monitor Details
- Update Monitor
- Delete Monitor
- Manual Health Check (hit a URL and return status)
- View Check History for a Monitor

**API Contract (v1)**

```
POST   /api/v1/auth/register
POST   /api/v1/auth/login

GET    /api/v1/monitors
POST   /api/v1/monitors
GET    /api/v1/monitors/{id}
PUT    /api/v1/monitors/{id}
DELETE /api/v1/monitors/{id}

POST   /api/v1/monitors/{id}/check
GET    /api/v1/monitors/{id}/history
```

**Database Schema (Initial)**

```
users
├── id            UUID  PK
├── email         TEXT  UNIQUE NOT NULL
├── password_hash TEXT  NOT NULL
├── created_at    TIMESTAMPTZ
└── updated_at    TIMESTAMPTZ

monitors
├── id                UUID  PK
├── user_id           UUID  FK → users.id
├── url               TEXT  NOT NULL
├── name              TEXT
├── check_interval_s  INT   DEFAULT 300
├── expected_status   INT   DEFAULT 200
├── is_active         BOOL  DEFAULT true
├── created_at        TIMESTAMPTZ
└── updated_at        TIMESTAMPTZ

check_results
├── id            UUID  PK
├── monitor_id    UUID  FK → monitors.id
├── status_code   INT
├── response_time_ms  INT
├── is_up         BOOL
├── error         TEXT
├── checked_at    TIMESTAMPTZ
└── created_at    TIMESTAMPTZ
```

**Project Structure**

```
system-design-lab/
├── cmd/
│   └── server/
│       └── main.go              # Entry point
├── internal/
│   ├── config/                  # Env-based configuration
│   ├── handler/                 # HTTP handlers (controllers)
│   ├── middleware/               # Auth, logging, recovery
│   ├── model/                   # Domain structs
│   ├── repository/              # Database access layer
│   ├── service/                 # Business logic
│   └── checker/                 # URL health check logic
├── migrations/                  # SQL migration files
├── docs/                        # Phase docs + ADRs
│   ├── adr/
│   └── journal/
├── docker-compose.yml
├── Dockerfile
├── Makefile
├── go.mod
├── go.sum
├── .env.example
└── README.md
```

**Standardized Error Response**

```json
{
  "error": {
    "code": "MONITOR_NOT_FOUND",
    "message": "Monitor with id abc-123 does not exist"
  }
}
```

**Learn**

- REST API design & conventions
- Database schema design & normalization
- Layered architecture (handler → service → repository)
- Input validation at the boundary
- Structured error handling
- JWT authentication flow
- Database migrations (versioned, reversible)
- Database indexing basics (add indexes on user_id, monitor_id, checked_at)

**Metrics Baseline**

- Measure response time for all endpoints with curl/httpie
- Record manual check latency
- Note query times with `EXPLAIN ANALYZE`
- This becomes the baseline for future phases

**Architecture**

```
Client → Go API Server → PostgreSQL
```

---

## Phase 2 — Background Worker & Scheduler

**Features**

- Automated periodic URL checks (based on each monitor's check_interval_s)
- Checks run in a background goroutine, not in HTTP request path
- Graceful shutdown (finish in-flight checks before exit)

**Learn**

- Why background workers? (HTTP requests should return fast, not block on external calls)
- Go concurrency: goroutines, sync.WaitGroup, context cancellation
- Ticker-based scheduling vs cron
- Graceful shutdown with OS signals
- Separation of concerns: API server vs worker process

**System Design Concepts**

- Synchronous vs asynchronous processing
- Long-running processes
- Graceful shutdown

**Architecture**

```
Client → Go API Server → PostgreSQL
                              ↑
         Background Worker ───┘
         (goroutine / ticker)
```

---

## Phase 3 — Notifications

**Features**

- Alert when a monitor goes DOWN (status change detection)
- Alert when a monitor comes back UP
- Notification channels: email (SMTP) and/or webhook
- Notification preferences per monitor
- Avoid alert storms (cooldown period)

**Learn**

- Why notifications matter (a monitor without alerts is just a logger)
- Event detection: comparing current state vs previous state
- External service integration (SMTP, webhook POST)
- Idempotency: don't send duplicate alerts
- Cooldown/debounce patterns

**System Design Concepts**

- Event-driven design (state change → trigger)
- External service reliability
- Idempotency

---

## Phase 4 — Worker Pool

**Features**

- Configurable pool of N workers checking URLs concurrently
- Fair distribution of monitors across workers
- Prevent duplicate checks on the same monitor

**Learn**

- Why a worker pool? (single goroutine can't keep up with 1000+ monitors)
- Go channels as work queues
- Bounded concurrency (semaphore pattern)
- Fan-out / fan-in pattern
- Benchmark: single worker vs N workers throughput

**System Design Concepts**

- Concurrency vs parallelism
- Bounded concurrency
- Fan-out / fan-in
- Throughput scaling

---

## Phase 5 — Retry Strategy

**Features**

- Configurable retry count per monitor
- Exponential backoff with jitter
- Mark as DOWN only after all retries exhausted
- Log each retry attempt

**Learn**

- Why retry? (network is unreliable — transient failures are normal)
- Exponential backoff math
- Why jitter? (thundering herd problem)
- Circuit breaker concept (awareness, not necessarily implementation)

**System Design Concepts**

- Transient vs permanent failures
- Exponential backoff + jitter
- Thundering herd
- Circuit breaker (conceptual)

---

## Phase 6 — Message Queue

**Features**

- Replace direct DB writes from workers with queue-based publishing
- Workers publish check results to RabbitMQ
- Consumer process reads from queue and writes to DB
- Dead letter queue for failed messages

**Learn**

- Why a queue? (decouple producers from consumers, handle spikes, survive crashes)
- RabbitMQ basics: exchanges, queues, bindings, acks
- At-least-once vs at-most-once delivery
- Dead letter queues
- What happens when the consumer is down? (durability)
- Benchmark: direct DB write vs queue-based write under load

**System Design Concepts**

- Producer-consumer pattern
- Message durability
- Delivery guarantees
- Decoupling via async messaging
- Backpressure

**Architecture**

```
Client → Go API Server → PostgreSQL
                              ↑
         Worker Pool ──→ RabbitMQ ──→ Consumer ──→ PostgreSQL
```

---

## Phase 7 — Caching

**Features**

- Cache monitor list and dashboard summary in Redis
- Cache invalidation on monitor create/update/delete
- Cache check history with TTL

**Learn**

- Why cache? (measure DB query times first — prove the bottleneck)
- Cache-aside pattern
- TTL-based expiration
- Cache invalidation strategies (write-through, write-behind, cache-aside)
- Cache stampede and how to prevent it
- Benchmark: with cache vs without cache response times

**System Design Concepts**

- Cache-aside, write-through, write-behind
- Cache invalidation (the hard problem)
- TTL strategies
- Cache stampede

---

## Phase 8 — Rate Limiting & Auth Hardening

**Features**

- Rate limit API endpoints (per user, per IP)
- Token refresh flow
- API key support for programmatic access
- API versioning strategy (URL prefix: /api/v1/, /api/v2/)

**Learn**

- Why rate limiting? (protect against abuse, ensure fairness)
- Token bucket vs sliding window algorithms
- Implement rate limiter in middleware
- API versioning: when and how to introduce v2
- Security headers (CORS, HSTS, etc.)

**System Design Concepts**

- Rate limiting algorithms
- API versioning strategies
- Defense in depth

---

## Phase 9 — Observability

**Features**

- Structured logging (JSON logs with request ID, trace context)
- Prometheus metrics endpoint (/metrics)
- Grafana dashboard (request rate, latency percentiles, error rate, queue depth)
- Health check endpoint (/health, /ready)
- Request tracing with correlation IDs

**Learn**

- Why observability? (you can't improve what you can't measure)
- Three pillars: logs, metrics, traces
- RED method (Rate, Errors, Duration)
- USE method (Utilization, Saturation, Errors)
- Prometheus data model (counters, gauges, histograms)
- Alerting basics

**System Design Concepts**

- Observability vs monitoring
- RED and USE methods
- SLIs, SLOs, SLAs (define them for your own service)

---

## Phase 10 — Load Balancer & Horizontal Scaling

**Features**

- Run multiple API server instances behind Nginx
- Sticky sessions vs stateless design
- Database connection pooling across instances
- Docker Compose scale: `docker compose up --scale api=3`

**Learn**

- Why load balancer? (single server is a SPOF and has throughput limits)
- Load balancing algorithms (round-robin, least connections, IP hash)
- Stateless vs stateful services
- Session management in a multi-instance setup
- Connection pooling (PgBouncer)
- Benchmark: 1 instance vs 3 instances under load

**System Design Concepts**

- Single point of failure (SPOF)
- Horizontal vs vertical scaling
- Stateless services
- Connection pooling
- Load balancing algorithms

**Architecture**

```
Client → Nginx (LB) → [API-1, API-2, API-3] → PostgreSQL
                                                    ↑
         Worker Pool ──→ RabbitMQ ──→ Consumer ──→──┘
                                                    ↑
                            Redis ──────────────────┘
```

---

## Phase 11 — Performance Testing

**Features**

- k6 load test scripts for all critical endpoints
- Soak test (sustained load over time)
- Spike test (sudden traffic burst)
- Identify breaking points of current architecture
- Document findings with graphs

**Learn**

- Why performance test? (assumptions about scale are usually wrong)
- Types: load test, stress test, soak test, spike test
- Identifying bottlenecks from metrics (CPU, memory, DB connections, queue depth)
- Capacity planning basics

**System Design Concepts**

- Capacity planning
- Bottleneck analysis
- Latency vs throughput trade-offs

---

## Phase 12 — Architecture Review & Refactoring

**Deliverables**

- Final architecture diagram (before vs after, Phase 1 vs Phase 12)
- Complete ADR collection review
- Engineering journal summary
- List of all system design concepts learned with real examples
- "What I'd do differently" document
- Identify next evolution paths (database replication, sharding, microservices, API gateway)

**Learn**

- Reflective engineering practice
- Architecture evolution documentation
- Identifying when to decompose a monolith
- Planning for 10x and 100x scale

---

## Ground Rules

Before adding any new technology, answer:

1. What problem exists?
2. Can the current system handle it?
3. How was the bottleneck measured? (show numbers)
4. What alternatives were evaluated?
5. Why was this solution chosen?
6. What complexity does it introduce?
7. What is the rollback plan if it fails?

If these questions cannot be answered, the technology will not be added.

---

## Final Goal

The final repository should demonstrate the evolution of a
production-grade backend system while documenting every engineering
decision, measurement, and trade-off.

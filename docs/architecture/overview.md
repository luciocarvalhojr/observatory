# Architecture Overview

## Context

Observatory is a self-hosted observability platform running on a K3s homelab cluster.
It is designed as a microservices reference architecture demonstrating production-grade
patterns including event-driven communication, database-per-service, GitOps deployment,
and a full DevSecOps pipeline.

---

## System Architecture

```
                           ┌─────────────────────────────────────────┐
                           │              K3s Cluster                 │
                           │                                          │
Internet ──► Traefik ──────►         api-gateway :8080               │
              (TLS)        │              │                           │
                           │    ┌─────────┼──────────┐               │
                           │    ▼         ▼          ▼               │
                           │ auth-svc  user-svc  alert-svc           │
                           │ :8081     :8082      :8083               │
                           │  Redis    PG         PG + Redis          │
                           │                        │                 │
                           │              NATS (alert.fired)          │
                           │              ┌──────────┴──────────┐     │
                           │              ▼                     ▼     │
                           │          notify-svc          incident-svc│
                           │          :8084               :8085       │
                           │          (async)             PG          │
                           └─────────────────────────────────────────┘
```

---

## Communication Patterns

### Synchronous (REST)
Used for: user-facing queries, CRUD operations, health checks

```
api-gateway ──REST──► auth-svc      (token validation)
api-gateway ──REST──► user-svc      (user operations)
api-gateway ──REST──► alert-svc     (alert rule management)
api-gateway ──REST──► incident-svc  (incident queries)
```

### Asynchronous (NATS)
Used for: cross-service events, notifications, incident creation

```
alert-svc   ──NATS──► notify-svc    (alert.fired, alert.resolved)
alert-svc   ──NATS──► incident-svc  (alert.fired, alert.resolved)
user-svc    ──NATS──► notify-svc    (user.created)
```

---

## Data Architecture

Each service owns its data store. No shared databases.

| Service | Store | Justification |
|---|---|---|
| auth-svc | Redis | Session store, token blacklist, fast TTL |
| user-svc | PostgreSQL | Relational, durable user records |
| alert-svc | PostgreSQL + Redis | Rules in PG, dedup state in Redis |
| notify-svc | None (stateless) | Consumes events, no persistence needed |
| incident-svc | PostgreSQL | Audit trail, timeline, state machine |

---

## Service Boundaries

```
┌─────────────────────────────────────────────────────────────┐
│ api-gateway                                                  │
│  - No business logic                                         │
│  - Rate limiting (per IP, per user)                          │
│  - Auth token validation (delegates to auth-svc)             │
│  - Circuit breaker per downstream service                    │
│  - Request/response logging + tracing                        │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ auth-svc                                                     │
│  - JWT issue + refresh + revoke                              │
│  - OIDC integration (Headlamp IDP)                           │
│  - JWKS endpoint for token verification                      │
│  - Token blacklist via Redis                                 │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ user-svc                                                     │
│  - User CRUD                                                 │
│  - API key management                                        │
│  - Notification preferences                                  │
│  - Publishes: user.created, user.deleted                     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ alert-svc                                                    │
│  - Alert rule CRUD                                           │
│  - Prometheus polling worker (30s interval)                  │
│  - Rule evaluation engine                                    │
│  - Deduplication via Redis (prevent alert storms)            │
│  - Publishes: alert.fired, alert.resolved                    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ notify-svc                                                   │
│  - Subscribes: alert.fired, alert.resolved, user.created     │
│  - Delivery: Slack, email, webhook                           │
│  - Retry with exponential backoff                            │
│  - Dead-letter queue for failed deliveries                   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ incident-svc                                                 │
│  - Subscribes: alert.fired, alert.resolved                   │
│  - Auto-creates incidents from alerts                        │
│  - State machine: open → acknowledged → resolved             │
│  - Timeline events per incident                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Cross-Cutting Concerns

Every service implements:

```go
// Health endpoints
GET /healthz   → liveness probe  (is the process alive?)
GET /readyz    → readiness probe (is the service ready to serve traffic?)

// Observability
- Structured JSON logging (zerolog)
- Prometheus metrics (/metrics)
- OpenTelemetry traces (→ Jaeger)

// Security
- Graceful shutdown on SIGTERM
- Read-only filesystem (distroless)
- Runs as nonroot:nonroot
- No shell in container
```

---

## Deployment Architecture

```
helm-charts repo (GitHub Pages)
        │
        │  helm pull
        ▼
ArgoCD Application
        │
        │  kubectl apply
        ▼
K3s Cluster
  ├── Namespace: observatory
  │     ├── api-gateway
  │     ├── auth-svc
  │     ├── user-svc
  │     ├── alert-svc
  │     ├── notify-svc
  │     └── incident-svc
  ├── Namespace: observatory-data
  │     ├── PostgreSQL clusters (CloudNativePG)
  │     ├── Redis
  │     └── NATS
  └── Namespace: observatory-infra
        ├── Jaeger
        └── (shared with monitoring stack)
```

---

## Related Documents

- [ADR-001: Go as the service language](../adr/001-go-language.md)
- [ADR-002: NATS as message broker](../adr/002-nats-broker.md)
- [ADR-003: Database per service](../adr/003-database-per-service.md)
- [ADR-004: Distroless container images](../adr/004-distroless-images.md)
- [ADR-005: Keyless image signing with Cosign](../adr/005-cosign-signing.md)

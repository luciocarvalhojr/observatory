# api-gateway

## Overview
Single entry point for all external traffic to the Observatory platform.
Validates JWT tokens via auth-svc introspect, then reverse-proxies requests to the
appropriate upstream service. No business logic lives here.

**Status:** 🔨 In Progress — scaffolded, not yet released
**Repository:** [observatory-api-gateway](https://github.com/luciocarvalhojr/observatory-api-gateway)

---

## Responsibilities
- TLS termination (via Traefik Ingress in front of the gateway)
- JWT validation — delegates to `auth-svc /auth/introspect` on every protected request
- Reverse proxy routing to upstream services
- Request/response structured logging
- Identity header injection (`X-User-Subject`, `X-User-Email`) for downstream services
- Rate limiting (planned)
- Circuit breaker per upstream (planned)

---

## Routing Table

| Path prefix    | Upstream     | Auth required |
|----------------|--------------|---------------|
| `/auth/*`      | auth-svc     | No            |
| `/users/*`     | user-svc     | Yes           |
| `/alerts/*`    | alert-svc    | Yes (future)  |
| `/incidents/*` | incident-svc | Yes (future)  |
| `/healthz`     | gateway self | No            |
| `/readyz`      | gateway self | No            |

---

## Auth Flow

```
Client ──► POST /auth/login ──► auth-svc (public, no token needed)
                                    │
                                    ▼ JWT issued

Client ──► GET /users/123
           Authorization: Bearer <token>
                │
                ▼
        api-gateway calls auth-svc /auth/introspect
                │
          ┌─────┴──────┐
          │ invalid     │ valid
          ▼             ▼
        401         proxy to user-svc
                    + X-User-Subject header
                    + X-User-Email header
```

---

## Environment Variables

| Variable           | Description                  | Default              |
|--------------------|------------------------------|----------------------|
| `PORT`             | HTTP listen port             | `8080`               |
| `AUTH_SVC_URL`     | auth-svc base URL            | required             |
| `USER_SVC_URL`     | user-svc base URL            | required             |
| `ALERT_SVC_URL`    | alert-svc base URL           | optional             |
| `INCIDENT_SVC_URL` | incident-svc base URL        | optional             |
| `OTLP_ENDPOINT`    | OpenTelemetry collector      | `http://jaeger:4318` |
| `ENV`              | `development`/`production`   | `production`         |

---

## Dependencies
- auth-svc (required — token validation on every protected request)
- user-svc, alert-svc, incident-svc (upstream proxies)
- No database, no Redis — fully stateless

---

## Helm Chart
```bash
helm install observatory-api-gateway luciocarvalhojr/observatory-api-gateway \
  --namespace observatory \
  --values environments/prod/api-gateway-values.yaml
```

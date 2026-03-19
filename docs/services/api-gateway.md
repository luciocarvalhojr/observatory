# API Gateway

## Overview

The API Gateway is implemented via **Traefik**, which is already the ingress controller
in the K3s cluster. No custom service is deployed. Traefik handles routing, JWT validation,
and identity header injection natively.

---

## How It Works

Every protected route uses Traefik's `forwardAuth` middleware, which calls
`auth-svc /auth/introspect` internally before forwarding the request upstream.

```
Client request (Bearer token)
        │
        ▼
     Traefik
        │
        ├── /auth/* ──────────────────────────────────► auth-svc:8081
        │   (public — no middleware)
        │
        ├── /users/*  ──► [forwardAuth middleware] ──► auth-svc:8081/auth/introspect
        │                        │
        │               ┌────────┴────────┐
        │            200 OK           401
        │         (token valid)   (returned to client)
        │               │
        │   inject X-User-Subject, X-User-Email headers
        │               │
        │               └────────────────────────────► user-svc:8082
        │
        └── /alerts/* /incidents/* (same pattern, future services)
```

---

## Kubernetes Config

### Middleware (defined once, reused by all protected services)

```yaml
# apps/observatory/traefik-middleware-auth.yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: observatory-auth
  namespace: observatory
spec:
  forwardAuth:
    address: http://auth-svc.observatory.svc.cluster.local:8081/auth/introspect
    authResponseHeaders:
      - X-User-Subject
      - X-User-Email
```

### Ingress annotation for protected services

```yaml
annotations:
  traefik.ingress.kubernetes.io/router.middlewares: observatory-observatory-auth@kubernetescrd
```

---

## Routing Table

| Path prefix     | Hostname                           | Service      | Auth required |
|-----------------|------------------------------------|--------------|---------------|
| `/*`            | `observatory-auth.capihome.xyz`    | auth-svc     | No            |
| `/users/*`      | `api.capihome.xyz`                 | user-svc     | Yes           |
| `/alerts/*`     | `api.capihome.xyz`                 | alert-svc    | Yes (future)  |
| `/incidents/*`  | `api.capihome.xyz`                 | incident-svc | Yes (future)  |

> **TODO:** Consolidate `observatory-auth.capihome.xyz` → `api.capihome.xyz/auth/*`
> once all services are stable. Requires coordinating Authentik OIDC client config
> and `OIDC_REDIRECT_URL` in auth-svc-values.yaml.

---

## Why Traefik Instead of a Custom Service

- Traefik is already the ingress controller — a custom proxy adds an unnecessary hop
- `forwardAuth` covers JWT validation + header injection natively
- Rate limiting (`rateLimit`) and circuit breaker (`circuitBreaker`) are built-in middleware
- Zero additional service to build, test, deploy, or maintain

---

## Files in k8s-home-lab

```
apps/observatory/
  traefik-middleware-auth.yaml    ← forwardAuth middleware (this is the gateway config)

# Per-service ingress (in each service's Helm chart values):
apps/observatory/
  user-svc-values.yaml            ← ingress with observatory-auth middleware annotation
  alert-svc-values.yaml           ← (future)
  incident-svc-values.yaml        ← (future)
```

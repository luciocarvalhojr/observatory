# auth-svc

## Overview
Handles authentication and authorization for the Observatory platform.
Issues and validates JWT tokens, integrates with the cluster OIDC provider (Headlamp IDP),
and manages token revocation via Redis.

**Status:** 🟡 Planned
**Repository:** [observatory-auth-svc](https://github.com/luciocarvalhojr/observatory-auth-svc)

---

## Responsibilities
- Issue JWT access tokens + refresh tokens
- Validate tokens (used by api-gateway on every request)
- OIDC integration for SSO with the homelab IDP
- Token revocation / blacklist via Redis
- JWKS endpoint (`/.well-known/jwks.json`) for public key distribution

---

## API Endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/auth/login` | Issue tokens from username/password |
| `POST` | `/auth/refresh` | Refresh access token |
| `POST` | `/auth/logout` | Revoke token (add to blacklist) |
| `GET` | `/.well-known/jwks.json` | Public keys for token verification |
| `GET` | `/healthz` | Liveness probe |
| `GET` | `/readyz` | Readiness probe |

---

## Data Store
**Redis** — session store and token blacklist
- Access tokens: TTL 15 minutes
- Refresh tokens: TTL 7 days
- Revoked tokens: stored until natural expiry

---

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `PORT` | HTTP listen port | `8081` |
| `JWT_SECRET` | Signing key (SealedSecret) | — |
| `JWT_ACCESS_TTL` | Access token TTL | `15m` |
| `JWT_REFRESH_TTL` | Refresh token TTL | `168h` |
| `REDIS_URL` | Redis connection string | — |
| `OIDC_ISSUER` | OIDC provider URL | — |

---

## Dependencies
- Redis (auth-svc namespace)
- OIDC provider (idp namespace)

---

## Helm Chart
```bash
helm install auth-svc luciocarvalhojr/auth-svc \
  --namespace observatory \
  --values environments/prod/auth-svc-values.yaml
```

---

## Runbook
- [Token validation failing](../runbooks/auth-token-validation.md)
- [Redis connection issues](../runbooks/redis-connection.md)

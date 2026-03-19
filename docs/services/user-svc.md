# user-svc

## Overview
Handles user management for the Observatory platform.
Provides user CRUD via REST, persists to PostgreSQL, and publishes domain events to NATS
so downstream services (notify-svc, incident-svc) can react to user lifecycle changes.

**Status:** 🔨 In Progress — scaffolded, not yet released
**Repository:** [observatory-user-svc](https://github.com/luciocarvalhojr/observatory-user-svc)

---

## Responsibilities
- User CRUD (create, read, update, delete)
- API key management (future)
- Notification preferences (future)
- Publishes: `user.created`, `user.deleted` to NATS

---

## API Endpoints

| Method   | Path         | Description              |
|----------|--------------|--------------------------|
| `POST`   | `/users`     | Create a user            |
| `GET`    | `/users/:id` | Get user by ID           |
| `PUT`    | `/users/:id` | Update user name         |
| `DELETE` | `/users/:id` | Delete user              |
| `GET`    | `/healthz`   | Liveness probe           |
| `GET`    | `/readyz`    | Readiness probe (PG ping)|

---

## Data Store
**PostgreSQL** — durable user records
- Table: `users (id UUID, email TEXT UNIQUE, name TEXT, created_at, updated_at)`
- Schema applied at first start via `dev/init.sql`; production migrations via goose (planned)

---

## NATS Events Published

| Subject        | Trigger                | Payload                          |
|----------------|------------------------|----------------------------------|
| `user.created` | `POST /users` success  | `{id, email, name}`              |
| `user.deleted` | `DELETE /users/:id`    | `{id}`                           |

---

## Environment Variables

| Variable        | Description               | Default              |
|-----------------|---------------------------|----------------------|
| `PORT`          | HTTP listen port          | `8082`               |
| `DATABASE_URL`  | PostgreSQL connection URL | required             |
| `NATS_URL`      | NATS connection URL       | required             |
| `OTLP_ENDPOINT` | OpenTelemetry collector   | `http://jaeger:4318` |
| `ENV`           | `development`/`production`| `production`         |

---

## Dependencies
- PostgreSQL (observatory-data namespace)
- NATS (observatory-data namespace)

---

## Local Dev

```bash
docker compose up
curl localhost:8082/healthz
curl -X POST localhost:8082/users \
  -H 'Content-Type: application/json' \
  -d '{"email":"alice@example.com","name":"Alice"}'
```

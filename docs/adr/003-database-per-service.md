# ADR-003: Database per service

## Status
Accepted

## Context
Microservices can either share a database or each own their data store.
Shared databases create tight coupling between services and make independent
deployment and scaling impossible.

## Decision
Each service owns its own PostgreSQL database cluster provisioned by CloudNativePG.
No service accesses another service's database directly — all cross-service data
access happens via API calls or NATS events.

## Consequences
- Services can be deployed, scaled, and migrated independently
- Schema changes in one service cannot break another
- CloudNativePG operator handles HA, backups, and connection pooling
- Higher resource usage (multiple PG clusters vs one shared)
- Data that needs to be joined across services requires API composition in api-gateway

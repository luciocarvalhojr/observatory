# ADR-002: NATS as the message broker

## Status
Accepted

## Context
Services need to communicate asynchronously for alert events and notifications.
We need a broker that is lightweight enough to run on a homelab K3s cluster,
supports at-least-once delivery, and has a dead-letter queue pattern.

## Decision
Use NATS with JetStream enabled for persistent, at-least-once delivery.

## Alternatives considered
- **Kafka** — too heavy for a homelab, overkill for 6 services
- **RabbitMQ** — heavier operational burden, less cloud-native
- **Redis Streams** — Redis is already used for caching; mixing concerns
- **NATS** — single binary, low memory, JetStream adds persistence, native K8s operator

## Consequences
- NATS JetStream provides durable consumers and dead-letter queues
- Single binary deployment (~20MB), very low memory footprint
- `notify-svc` can replay missed events after restart
- All publishers/consumers use the same NATS connection pool

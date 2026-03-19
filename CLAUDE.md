# Observatory

Self-hosted observability platform: alert rules evaluated against Prometheus → notifications via NATS → incident tracking. Runs on K3s homelab with ArgoCD GitOps.

This repo is the **platform library** (shared configs, docs, ADRs). Application code lives in separate `observatory-*-svc` repos.

## Current State

- **Version:** v1.0.0 (tagged). HEAD is 2 commits ahead (b734e0c).
- **Pipeline:** GitHub Actions with semantic-release. Triggers on push to `main` and PRs (dry-run).
- **Deployed?** Infrastructure is deployed (K3s, ArgoCD, Traefik, NATS, PostgreSQL via CloudNativePG, Redis, Kyverno, Falco, Jaeger, Prometheus/Grafana). No application services have been released yet — all service badges show unreleased.

## In Progress

- **Release tooling migration:** Attempting to migrate from semantic-release to release-please (ADR-000). Currently blocked — the `extends` URL approach for semantic-release was tried and reverted (last 4 commits). Migration to release-please has not started.
- **Shared config distribution:** `configs/` holds single-source-of-truth configs (`.golangci.yml`, `.releaserc.yaml`, `renovate/base.json`) intended to be extended by all services. The `extends` mechanism for `.releaserc.yaml` is currently not working (reverted to inline config).
- **Service implementation:** All 5 services (auth, user, alert, notify, incident) and the api-gateway are planned but none are released. Only `auth-svc` has documentation written.
- **Roadmap phases 1–3** are in progress; phases 4–5 are planned.

## Known Issues

- **`.go-version` is empty** — Go version pinning is documented (1.24) but the file at `configs/.go-version` has no content.
- **semantic-release `extends` via URL doesn't work** — the attempt to centralize `.releaserc.yaml` using a raw GitHub URL for `extends` failed and was reverted. Services must manually copy the config or use Renovate to keep them in sync (drift risk).
- **Only `auth-svc` is documented** — `docs/services/` has one file; all other services lack documentation.
- **Roadmap is almost entirely unchecked** — ROADMAP.md shows phases 1–3 as "in progress" but most checkboxes remain empty.

## Next Steps

1. Resolve release strategy: complete migration from semantic-release → release-please (see ADR-000 for step-by-step plan).
2. Add `configs/.goreleaser.base.yaml` as shared GoReleaser config.
3. Implement and release `auth-svc` first (it's the migration validation target).
4. Roll out remaining services (user, alert, notify, incident, api-gateway).
5. Fix `configs/.go-version` — add `1.24` content.
6. Write service docs for all services (following `docs/services/auth-svc.md` as template).
7. Phase 4: OpenTelemetry instrumentation + Jaeger trace integration in services.
8. Phase 5: HPA, PodDisruptionBudget, NetworkPolicy, full SealedSecrets rollout.

## Key Decisions

- **Go 1.24 + Gin Gonic** (ADR-001): static binaries, fast compile, strong concurrency for microservices.
- **NATS JetStream** (ADR-002): chosen over Kafka/RabbitMQ for lightweight homelab operation; at-least-once delivery via persistent streams.
- **Database-per-service** (ADR-003): PostgreSQL via CloudNativePG operator; independent scaling per service.
- **Distroless images** (ADR-004): ~5MB images, no shell, minimal CVE surface; built via ko or multi-stage Docker.
- **Keyless Cosign signing** (ADR-005): GitHub OIDC → Sigstore; zero secret management; Kyverno enforces signature verification at admission.
- **80% test coverage gate** enforced in CI; PRs blocked if coverage drops below threshold.
- **Conventional Commits** required; semantic-release (and future release-please) derive versions from commit messages.

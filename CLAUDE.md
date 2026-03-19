# Observatory

Self-hosted observability platform: alert rules evaluated against Prometheus → notifications via NATS → incident tracking. Runs on K3s homelab with ArgoCD GitOps.

This repo is the **platform library** (shared configs, docs, ADRs). Application code lives in separate `observatory-*-svc` repos.

## Current State

- **Version:** v1.0.0 (tagged). HEAD is 2 commits ahead (b734e0c).
- **Pipeline:** GitHub Actions with semantic-release. Triggers on push to `main` and PRs (dry-run).
- **Infrastructure:** Deployed — K3s, ArgoCD, Traefik, cert-manager, Authentik, Sealed Secrets, Prometheus/Grafana, NATS, PostgreSQL via CloudNativePG, Redis, Kyverno, Falco, Jaeger.

## Service Status

| Service       | Status                          | Version | Notes                              |
|---------------|---------------------------------|---------|------------------------------------|
| auth-svc      | 🟢 Deployed                     | v1.6.0  | Helm chart 0.1.3, ArgoCD synced    |
| user-svc      | 🔨 Scaffolded — not released    | —       | Pushed to GitHub, needs Helm chart + ArgoCD |
| api-gateway   | ✅ Implemented via Traefik      | —       | `forwardAuth` middleware in k8s-home-lab; no custom service |
| alert-svc     | ⬜ Not started                  | —       |                                    |
| notify-svc    | ⬜ Not started                  | —       |                                    |
| incident-svc  | ⬜ Not started                  | —       |                                    |

## In Progress

- **user-svc scaffolded** — needs Helm chart in `helm-charts` and ArgoCD Application in `k8s-home-lab`.
- **Release tooling migration:** Attempting to migrate from semantic-release to release-please (ADR-000). Currently blocked — the `extends` URL approach was tried and reverted. Migration to release-please has not started.

## Known Issues

- **`.go-version` is empty** — `configs/.go-version` has no content; should be `1.26.1`.
- **semantic-release `extends` via URL doesn't work** — services must manually copy `.releaserc.yaml` (drift risk).
- **Coverage gate at 0%** on all services — threshold must be raised to 80% once tests are written.
- **JWT_SECRET not sealed** in k8s-home-lab — `apps/observatory/auth-svc-sealed-secret.yaml` has it commented out.
- **Plaintext OIDC clientSecret** in k8s-home-lab `bootstrap/argocd/values.yaml` and `apps/my-headlamp/values.yaml`.

## Next Steps

1. Add Helm chart for `user-svc` in `helm-charts` repo.
2. Add ArgoCD Application in `k8s-home-lab` for `user-svc`.
3. Seal `JWT_SECRET` in `k8s-home-lab` (see `apps/observatory/README.md:38`).
4. Write tests for `auth-svc` — raise coverage gate from 0% to 80%.
5. Fix `configs/.go-version` — add `1.26.1`.
6. Write service docs for `alert-svc`, `notify-svc`, `incident-svc` (once started).
7. Phase 2: implement `alert-svc` → `notify-svc` → `incident-svc`.

## Key Decisions

- **Go 1.26.1 + Gin Gonic** (ADR-001): static binaries, fast compile, strong concurrency for microservices.
- **NATS JetStream** (ADR-002): chosen over Kafka/RabbitMQ for lightweight homelab operation; at-least-once delivery via persistent streams.
- **Database-per-service** (ADR-003): PostgreSQL via CloudNativePG operator; independent scaling per service.
- **Distroless images** (ADR-004): ~5MB images, no shell, minimal CVE surface; built via multi-stage Docker.
- **Keyless Cosign signing** (ADR-005): GitHub OIDC → Sigstore; zero secret management; Kyverno enforces signature verification at admission.
- **80% test coverage gate** enforced in CI; PRs blocked if coverage drops below threshold.
- **Conventional Commits** required; semantic-release (and future release-please) derive versions from commit messages.
- **Single unified devsecops.yml** per service: all CI/CD stages in one file; release/deploy jobs gated on main branch push.

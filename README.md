# 🔭 Observatory

> A self-hosted observability platform built as a microservices reference architecture.
> Designed to demonstrate production-grade DevSecOps, GitOps, and cloud-native patterns on a K3s homelab.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
[![Stack](https://img.shields.io/badge/stack-Go%20%7C%20K3s%20%7C%20ArgoCD-blueviolet)](#tech-stack)

---

## 📖 Overview

Observatory is a platform that allows users to define alert rules against Prometheus metrics,
receive notifications, and track incidents — all running on a self-hosted K3s cluster.

This project exists to demonstrate:

- **Microservices architecture** with real domain logic
- **Event-driven patterns** using NATS as a message broker
- **Full DevSecOps pipeline** from commit to runtime
- **GitOps deployment** with ArgoCD and Helm
- **Supply chain security** with Cosign image signing and Kyverno verification
- **Platform engineering** with shared configs, reusable workflows, and drift prevention

---

## 🗺️ Quick Navigation

| Document | Description |
| --- | --- |
| [Architecture Overview](docs/architecture/overview.md) | System design and decisions |
| [Services](docs/services) | Per-service documentation |
| [DevSecOps Pipeline](docs/devsecops/pipeline.md) | CI/CD and security pipeline |
| [Infrastructure](docs/infrastructure/cluster.md) | K3s cluster and tooling |
| [ADRs](docs/adr) | Architecture Decision Records |
| [Roadmap](docs/ROADMAP.md) | Project phases and milestones |
| [Release Strategy](docs/release-strategy.md) | Versioning and release process |
| [Contributing](CONTRIBUTING.md) | How to contribute |

---

## 🏗️ Services

| Service | Description | Tech | Version |
| --- | --- | --- | --- |
| [api-gateway](docs/services/api-gateway.md) | Entry point, routing, rate limiting | Traefik + Kong | ![version](https://img.shields.io/github/v/release/luciocarvalhojr/observatory-api-gateway?label=&color=lightgrey) |
| [auth-svc](docs/services/auth-svc.md) | JWT auth, OIDC integration | Go + Gin + Redis | ![version](https://img.shields.io/github/v/release/luciocarvalhojr/observatory-auth-svc?label=&color=blue) |
| [user-svc](docs/services/user-svc.md) | User management | Go + Gin + PostgreSQL | ![version](https://img.shields.io/github/v/release/luciocarvalhojr/observatory-user-svc?label=&color=lightgrey) |
| [alert-svc](docs/services/alert-svc.md) | Alert rules engine | Go + Gin + PostgreSQL + Redis | ![version](https://img.shields.io/github/v/release/luciocarvalhojr/observatory-alert-svc?label=&color=lightgrey) |
| [notify-svc](docs/services/notify-svc.md) | Notification delivery | Go + Gin + NATS | ![version](https://img.shields.io/github/v/release/luciocarvalhojr/observatory-notify-svc?label=&color=lightgrey) |
| [incident-svc](docs/services/incident-svc.md) | Incident tracking | Go + Gin + PostgreSQL | ![version](https://img.shields.io/github/v/release/luciocarvalhojr/observatory-incident-svc?label=&color=lightgrey) |

> Badges show the latest GitHub release. Grey = not yet released.

---

## 🔄 System Flow

```
User defines alert rule
        │
        ▼
alert-svc polls Prometheus every 30s
        │
   threshold breached
        │
        ├──── NATS: alert.fired ────► notify-svc ──► Slack / Email / Webhook
        │
        └──── NATS: alert.fired ────► incident-svc ──► Incident created
```

---

## 🛡️ DevSecOps Pipeline

Every service follows the same pipeline:

```
Commit → lint → test → SAST → SCA → secret scan
                                          │
                              Tag → build image → trivy scan
                                          │
                              SBOM → cosign sign → GitHub Release
                                          │
                              ArgoCD sync → kyverno verify → deploy
                                          │
                              Runtime: falco + trivy-operator
```

> See [Release Strategy](docs/release-strategy.md) for how versioning and releases work across services.

---

## 🧰 Tech Stack

### Application

| Layer | Technology |
| --- | --- |
| Language | Go 1.24 |
| Framework | Gin Gonic |
| Message Broker | NATS |
| Databases | PostgreSQL (CloudNativePG) + Redis |
| API Docs | Swagger (swaggo) |

### Platform

| Layer | Technology |
| --- | --- |
| Cluster | K3s |
| GitOps | ArgoCD |
| Packaging | Helm |
| Ingress | Traefik |
| API Gateway | Kong (planned) |
| Identity Provider | authentik |
| TLS | cert-manager + Let's Encrypt |
| Storage | NFS CSI |

### DevSecOps

| Layer | Technology |
| --- | --- |
| Lint | golangci-lint |
| SAST | gosec |
| SCA | govulncheck |
| Secret Scan | Gitleaks |
| Image Scan | Trivy |
| SBOM | Syft (anchore) |
| Image Signing | Cosign (keyless) |
| Policy Enforcement | Kyverno |
| Runtime Security | Falco |
| Tracing | OpenTelemetry + Jaeger |
| Dependency Updates | Renovate |

---

## 📦 Repositories

### Platform

| Repository | Description |
| --- | --- |
| [observatory](https://github.com/luciocarvalhojr/observatory) | This repo — docs, shared configs & platform library |
| [helm-charts](https://github.com/luciocarvalhojr/helm-charts) | Helm chart registry (GitHub Pages) |
| [k8s-home-lab](https://github.com/luciocarvalhojr/k8s-home-lab) | K3s cluster GitOps (ArgoCD apps-of-apps) |

### Templates

| Repository | Description |
| --- | --- |
| [go-api](https://github.com/luciocarvalhojr/go-api) | Golden template — production-ready Go API with full DevSecOps pipeline |

### Services

| Repository | Description |
| --- | --- |
| [observatory-auth-svc](https://github.com/luciocarvalhojr/observatory-auth-svc) | Auth service |
| [observatory-api-gateway](https://github.com/luciocarvalhojr/observatory-api-gateway) | API Gateway service |
| [observatory-user-svc](https://github.com/luciocarvalhojr/observatory-user-svc) | User service |
| [observatory-alert-svc](https://github.com/luciocarvalhojr/observatory-alert-svc) | Alert service |
| [observatory-notify-svc](https://github.com/luciocarvalhojr/observatory-notify-svc) | Notify service |
| [observatory-incident-svc](https://github.com/luciocarvalhojr/observatory-incident-svc) | Incident service |

---

## 🚀 Getting Started

### Prerequisites

- K3s cluster running
- ArgoCD installed
- cert-manager + ClusterIssuer configured
- Helm 3.x

### Deploy via ArgoCD (recommended)

```bash
kubectl apply -f https://raw.githubusercontent.com/luciocarvalhojr/k8s-home-lab/main/argocd/argocd-apps-cleaned.yaml
```

### Deploy via Helm

```bash
helm repo add luciocarvalhojr https://luciocarvalhojr.github.io/helm-charts
helm repo update
helm install auth-svc luciocarvalhojr/go-api \
  --namespace observatory \
  --create-namespace \
  -f ./observatory-auth-svc/values.yaml
```

---

## 📄 License

MIT — see [LICENSE](./LICENSE)
# 🔭 Observatory

> A self-hosted observability platform built as a microservices reference architecture.
> Designed to demonstrate production-grade DevSecOps, GitOps, and cloud-native patterns on a K3s homelab.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Services](https://img.shields.io/badge/services-6-green.svg)](#services)
[![Stack](https://img.shields.io/badge/stack-Go%20%7C%20K3s%20%7C%20ArgoCD-blueviolet.svg)](#tech-stack)

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

---

## 🗺️ Quick Navigation

| Document | Description |
|---|---|
| [Architecture Overview](docs/architecture/overview.md) | System design and decisions |
| [Services](docs/services/) | Per-service documentation |
| [DevSecOps Pipeline](docs/devsecops/pipeline.md) | CI/CD and security pipeline |
| [Infrastructure](docs/infrastructure/cluster.md) | K3s cluster and tooling |
| [ADRs](docs/adr/) | Architecture Decision Records |
| [Roadmap](docs/ROADMAP.md) | Project phases and milestones |
| [Contributing](CONTRIBUTING.md) | How to contribute |

---

## 🏗️ Services

| Service | Description | Tech | Status |
|---|---|---|---|
| [api-gateway](docs/services/api-gateway.md) | Entry point, routing, rate limiting | Traefik or Kong | 🟡 Planned |
| [auth-svc](docs/services/auth-svc.md) | JWT auth, OIDC integration | Go + Gin + Redis | 🟡 Planned |
| [user-svc](docs/services/user-svc.md) | User management | Go + Gin + PostgreSQL | 🟡 Planned |
| [alert-svc](docs/services/alert-svc.md) | Alert rules engine | Go + Gin + PostgreSQL + Redis | 🟡 Planned |
| [notify-svc](docs/services/notify-svc.md) | Notification delivery | Go + Gin + NATS | 🟡 Planned |
| [incident-svc](docs/services/incident-svc.md) | Incident tracking | Go + Gin + PostgreSQL | 🟡 Planned |

**Status:** 🟢 Live · 🔵 In Progress · 🟡 Planned · 🔴 Deprecated

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
                              SBOM → cosign sign → helm-charts PR
                                          │
                              ArgoCD sync → kyverno verify → deploy
                                          │
                              Runtime: falco + trivy-operator
```

---

## 🧰 Tech Stack

### Application
| Layer | Technology |
|---|---|
| Language | Go 1.24 |
| Framework | Gin Gonic |
| Message Broker | NATS |
| Databases | PostgreSQL (CloudNativePG) + Redis |
| API Docs | Swagger (swaggo) |

### Platform
| Layer | Technology |
|---|---|
| Cluster | K3s |
| GitOps | ArgoCD |
| Packaging | Helm |
| Ingress | Traefik |
| TLS | cert-manager + Let's Encrypt |
| Storage | Longhorn / NFS CSI |

### DevSecOps
| Layer | Technology |
|---|---|
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

---

## 📦 Repositories

| Repository | Description |
|---|---|
| [observatory](https://github.com/luciocarvalhojr/observatory) | This repo — docs & planning |
| [observatory-api-gateway](https://github.com/luciocarvalhojr/observatory-api-gateway) | API Gateway service |
| [observatory-auth-svc](https://github.com/luciocarvalhojr/observatory-auth-svc) | Auth service |
| [observatory-user-svc](https://github.com/luciocarvalhojr/observatory-user-svc) | User service |
| [observatory-alert-svc](https://github.com/luciocarvalhojr/observatory-alert-svc) | Alert service |
| [observatory-notify-svc](https://github.com/luciocarvalhojr/observatory-notify-svc) | Notify service |
| [observatory-incident-svc](https://github.com/luciocarvalhojr/observatory-incident-svc) | Incident service |
| [helm-charts](https://github.com/luciocarvalhojr/helm-charts) | Helm chart registry |
| [k8s-home-lab](https://github.com/luciocarvalhojr/k8s-home-lab) | K3s cluster GitOps |

---

## 🚀 Getting Started

### Prerequisites
- K3s cluster running
- ArgoCD installed
- cert-manager + ClusterIssuer configured
- Helm 3.x

### Deploy the Platform

```bash
# Add helm registry
helm repo add luciocarvalhojr https://luciocarvalhojr.github.io/helm-charts
helm repo update

# Or via ArgoCD (recommended)
kubectl apply -f https://raw.githubusercontent.com/luciocarvalhojr/k8s-home-lab/main/argocd/observatory.yaml
```

---

## 📄 License

MIT — see [LICENSE](LICENSE)

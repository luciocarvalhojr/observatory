# Cluster Infrastructure

## Overview

Observatory runs on a self-hosted K3s cluster managed via GitOps with ArgoCD.
All infrastructure components are defined in the
[k8s-home-lab](https://github.com/luciocarvalhojr/k8s-home-lab) repository.

---

## Cluster Components

| Component | Namespace | Purpose | Version |
|---|---|---|---|
| K3s | — | Kubernetes distribution | latest |
| ArgoCD | argocd | GitOps engine | see argocd/argocd.yaml |
| Traefik | kube-system | Ingress controller + TLS termination | built-in K3s |
| cert-manager | cert-manager | Automatic TLS certificates | v1.19.2 |
| CloudNativePG | cnpg-system | PostgreSQL operator | latest |
| NATS | observatory-data | Message broker (JetStream) | latest |
| Redis | observatory-data | Cache + session store | latest |
| Sealed Secrets | sealed-secrets | Encrypted secret management | v2.18.1 |
| MetalLB | metallb-system | LoadBalancer for bare metal | v0.15.3 |
| Longhorn | longhorn-system | Distributed block storage | latest |
| External DNS | external-dns | Automatic DNS record management | latest |
| Kyverno | kyverno | Policy enforcement + image signing | latest |
| Falco | falco | Runtime security | latest |
| Trivy Operator | trivy-system | Continuous image scanning | latest |
| Jaeger | observability | Distributed tracing | latest |
| Prometheus Stack | monitoring | Metrics + alerting | 82.1.1 |

---

## Namespaces

```
observatory          → all Observatory services
observatory-data     → PostgreSQL, Redis, NATS
observability        → Jaeger, OpenTelemetry collector
monitoring           → Prometheus, Grafana, Alertmanager
argocd               → ArgoCD
cert-manager         → cert-manager
kyverno              → Kyverno
falco                → Falco
trivy-system         → Trivy Operator
sealed-secrets       → Sealed Secrets controller
```

---

## Network Architecture

```
External
  │
  │ :443 (HTTPS)
  ▼
MetalLB (192.168.0.x LoadBalancer IP)
  │
  ▼
Traefik (IngressController)
  │
  ├── observatory.yourdomain.com  → api-gateway:8080
  ├── argocd.yourdomain.com       → argocd-server:443
  ├── grafana.yourdomain.com      → grafana:3000
  └── jaeger.yourdomain.com       → jaeger-query:16686

Internal (ClusterIP only)
  auth-svc:8081
  user-svc:8082
  alert-svc:8083
  notify-svc:8084
  incident-svc:8085
```

---

## Storage

| Store | Type | Used by |
|---|---|---|
| CloudNativePG | Block (Longhorn) | user-svc, alert-svc, incident-svc |
| Redis | In-memory | auth-svc, alert-svc |
| NATS JetStream | Block (Longhorn) | notify-svc, incident-svc |
| NFS (192.168.0.110) | NFS | Media / bulk storage |

---

## TLS

All public endpoints use Let's Encrypt certificates via cert-manager.

```yaml
# ClusterIssuer (in k8s-home-lab)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    solvers:
      - http01:
          ingress:
            class: traefik
```

---

## Secrets Management

All secrets are encrypted with Sealed Secrets before committing to Git.

```bash
# Seal a secret
kubectl create secret generic my-secret \
  --from-literal=key=value \
  --dry-run=client -o yaml | \
  kubeseal --cert sealed-secrets.pem \
  --format yaml > my-secret-sealed.yaml
```

# Roadmap

## Phase 1 — Foundation 🏗️
> Goal: Core services running in cluster with basic pipeline

- [x] `auth-svc` — JWT issue/validate, Redis session store — **deployed v1.6.0**
- [ ] `user-svc` — User CRUD, PostgreSQL, domain events — **scaffolded, not released**
- [x] `api-gateway` — Implemented via Traefik `forwardAuth` middleware (no custom service)
- [x] Helm chart for `auth-svc` (observatory-auth-svc 0.1.3)
- [ ] Helm chart for `user-svc`
- [x] ArgoCD Application for `auth-svc` in k8s-home-lab
- [x] Traefik `forwardAuth` middleware manifest in k8s-home-lab (`apps/observatory/traefik-middleware-auth.yaml`)
- [ ] ArgoCD Application for `user-svc`
- [x] Basic `devsecops.yml` per service (lint + test + trivy) — auth-svc deployed; user-svc scaffolded
- [x] TLS via cert-manager + Traefik Ingress

## Phase 2 — Core Domain 🚨
> Goal: Alert rules engine and notifications working end-to-end

- [ ] `alert-svc` — Rule CRUD + Prometheus polling worker
- [ ] `notify-svc` — NATS consumer + Slack delivery
- [ ] `incident-svc` — Auto-create from alert events
- [ ] NATS deployed in cluster (CloudNativePG operator)
- [ ] End-to-end flow: rule fires → Slack message → incident created
- [ ] Helm charts + ArgoCD for new services

## Phase 3 — Full DevSecOps 🔒
> Goal: Complete security pipeline from commit to runtime

- [x] Gitleaks in all service pipelines (auth-svc deployed; user-svc scaffolded)
- [ ] Coverage gate (80%) enforced in CI — currently 0% on all services
- [x] SBOM generation + attached to GitHub releases (auth-svc)
- [x] Cosign keyless image signing (auth-svc)
- [ ] Kyverno policies: signature verify + approved registry
- [ ] Gatekeeper: no privileged, readOnlyRootFilesystem
- [ ] Falco installed in cluster
- [ ] Trivy Operator for continuous scanning
- [ ] Renovate configured for k8s-home-lab

## Phase 4 — Observability 📊
> Goal: Full visibility into the platform itself

- [ ] OpenTelemetry SDK in all services
- [ ] Jaeger deployed in cluster
- [ ] Distributed traces across Traefik → services
- [ ] Grafana dashboards per service (RED metrics)
- [ ] Grafana dashboard for security violations (Kyverno + Falco)
- [ ] Alertmanager rules for platform health

## Phase 5 — Hardening & Polish ✨
> Goal: Production-grade reliability and documentation

- [ ] PodDisruptionBudget for all services
- [ ] HorizontalPodAutoscaler for alert-svc + user-svc
- [ ] NetworkPolicy — restrict pod-to-pod traffic
- [ ] SealedSecrets for all sensitive config
- [ ] Full API documentation (Swagger) for all services
- [ ] Architecture diagrams (Mermaid)
- [ ] Blog post / writeup

# Roadmap

## Phase 1 — Foundation 🏗️
> Goal: Core services running in cluster with basic pipeline

- [ ] `auth-svc` — JWT issue/validate, Redis session store
- [ ] `user-svc` — User CRUD, PostgreSQL, domain events
- [ ] `api-gateway` — Routing, rate limiting, circuit breaker
- [ ] Helm charts for all 3 services
- [ ] ArgoCD Applications in k8s-home-lab
- [ ] Basic `devsecops.yml` per service (lint + test + trivy)
- [ ] TLS via cert-manager + Traefik Ingress

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

- [ ] Gitleaks in all service pipelines
- [ ] Coverage gate (80%) enforced in CI
- [ ] SBOM generation + attached to GitHub releases
- [ ] Cosign keyless image signing
- [ ] Kyverno policies: signature verify + approved registry
- [ ] Gatekeeper: no privileged, readOnlyRootFilesystem
- [ ] Falco installed in cluster
- [ ] Trivy Operator for continuous scanning
- [ ] Renovate configured for k8s-home-lab

## Phase 4 — Observability 📊
> Goal: Full visibility into the platform itself

- [ ] OpenTelemetry SDK in all services
- [ ] Jaeger deployed in cluster
- [ ] Distributed traces across api-gateway → services
- [ ] Grafana dashboards per service (RED metrics)
- [ ] Grafana dashboard for security violations (Kyverno + Falco)
- [ ] Alertmanager rules for platform health

## Phase 5 — Hardening & Polish ✨
> Goal: Production-grade reliability and documentation

- [ ] PodDisruptionBudget for all services
- [ ] HorizontalPodAutoscaler for api-gateway + alert-svc
- [ ] NetworkPolicy — restrict pod-to-pod traffic
- [ ] SealedSecrets for all sensitive config
- [ ] Full API documentation (Swagger) for all services
- [ ] Architecture diagrams (Mermaid)
- [ ] Blog post / writeup

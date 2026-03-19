# DevSecOps Pipeline

## Overview

Every service in Observatory follows an identical pipeline.
Security is applied at every layer: code, build, deploy, and runtime.

---

## Pipeline Stages

```
┌─────────────────────────────────────────────────────────────┐
│  STAGE 1: Pull Request (every PR to main)                    │
│                                                              │
│  gitleaks ──► golangci-lint ──► gosec ──► govulncheck        │
│                                                              │
│  go test -race -cover (threshold: 80%)                       │
│  swagger verify (docs must be up to date)                    │
└─────────────────────────────────────────────────────────────┘
                          │ merge
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  STAGE 2: Release (semantic-release creates tag)             │
│                                                              │
│  docker build (distroless, multi-stage)                      │
│       │                                                      │
│       ▼                                                      │
│  trivy scan (block on CRITICAL/HIGH)                         │
│       │                                                      │
│       ▼                                                      │
│  push to GHCR (ghcr.io/luciocarvalhojr/<service>:<ver>)      │
│       │                                                      │
│       ▼                                                      │
│  sbom-action (generate SPDX SBOM, attach to release)         │
│       │                                                      │
│       ▼                                                      │
│  cosign sign (keyless via GitHub OIDC — no keys stored)      │
│       │                                                      │
│       ▼                                                      │
│  workflow_dispatch → helm-charts (opens PR)                  │
└─────────────────────────────────────────────────────────────┘
                          │ PR merged
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  STAGE 3: Deploy (ArgoCD detects new chart version)          │
│                                                              │
│  kyverno: verify cosign signature before scheduling pod      │
│  kyverno: enforce approved registry (ghcr.io/luciocarvalhojr)│
│  kyverno: require resource limits on all containers          │
│  gatekeeper: no privileged containers                        │
│  gatekeeper: readOnlyRootFilesystem required                 │
└─────────────────────────────────────────────────────────────┘
                          │ running
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  STAGE 4: Runtime (continuous)                               │
│                                                              │
│  trivy-operator: rescan images on schedule                   │
│  falco: detect anomalous syscalls / shell in container       │
│  prometheus: scrape /metrics from all services               │
│  otel collector: receive traces → jaeger                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Security Tools Reference

| Tool | Stage | What it catches |
|---|---|---|
| Gitleaks | PR | Secrets committed to git history |
| golangci-lint | PR | Code quality, style, bugs |
| gosec | PR | SAST — SQL injection, hardcoded creds, unsafe operations |
| govulncheck | PR | SCA — known CVEs in Go dependencies |
| Trivy (CI) | Release | OS + library CVEs in container image |
| Syft/SBOM | Release | Full inventory of image contents |
| Cosign | Release | Cryptographic proof image came from your pipeline |
| Kyverno | Deploy | Signature verification, policy enforcement |
| Gatekeeper | Deploy | OPA policies — privileged containers, resource limits |
| Trivy Operator | Runtime | Continuous re-scanning of running workloads |
| Falco | Runtime | Anomalous behavior — unexpected shell, file writes |

---

## Workflow Files Per Service Repo

```
.github/workflows/
└── devsecops.yml     # single unified pipeline — PR gates + release + deploy
```

All stages are in one file. Release and deploy jobs are gated on `github.ref == 'refs/heads/main'`
so they only run on merge, not on PRs.

### devsecops.yml job chain

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test-and-lint:      # golangci-lint + go test + coverage gate + swagger verify
  security-scan:      # gitleaks + gosec + govulncheck
  docker-scan:        # build image + trivy scan (needs: test-and-lint, security-scan)
  release:            # semantic-release → creates GitHub tag/release (main only)
  build-and-push:     # docker build + push to GHCR + trivy + SBOM + cosign (main, new release only)
  update-helm-chart:  # opens PR on helm-charts repo to bump image tag (main, new release only)
```

---

## Coverage Gate

All services enforce a minimum 80% test coverage:

```bash
COVERAGE=$(go tool cover -func=coverage.out | grep total | awk '{print $3}' | tr -d '%')
if (( $(echo "$COVERAGE < 80" | bc -l) )); then
  echo "Coverage ${COVERAGE}% below threshold 80%"
  exit 1
fi
```

---

## Image Signing Flow (Cosign Keyless)

No private keys are stored anywhere. Signing uses GitHub OIDC:

```
GitHub Actions (release.yml)
    │
    │  OIDC token (proves: this action ran in this repo on this branch)
    ▼
Sigstore Fulcio (certificate authority)
    │
    │  short-lived certificate
    ▼
Cosign signs image digest
    │
    │  signature + certificate stored in
    ▼
Sigstore Rekor (transparency log)
```

Verification on the cluster (Kyverno):
```yaml
attestors:
  - keyless:
      subject: "https://github.com/luciocarvalhojr/observatory-*/
                .github/workflows/release.yml@refs/heads/main"
      issuer:  "https://token.actions.githubusercontent.com"
```

---

## Semantic Release Convention

All services use Conventional Commits to drive automatic versioning:

| Commit prefix | Version bump | Example |
|---|---|---|
| `fix:` | patch (0.0.X) | `fix: handle nil pointer in alert eval` |
| `feat:` | minor (0.X.0) | `feat: add webhook notification channel` |
| `feat!:` or `BREAKING CHANGE:` | major (X.0.0) | `feat!: redesign alert rule schema` |
| `chore:`, `docs:`, `test:` | no release | `chore: update dependencies` |

---

## Dependabot + Renovate

Each service repo has `dependabot.yaml` watching:
- Go modules (weekly)
- GitHub Actions versions (weekly)
- Dockerfile base images (weekly)

`k8s-home-lab` repo has `renovate.json` watching:
- ArgoCD `targetRevision` in all Application manifests
- Infrastructure Helm chart versions

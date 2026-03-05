# Contributing

## Starting a New Service

1. **Create the repo** — `observatory-<name>-svc` under your GitHub account
2. **Scaffold Go project:**
```bash
mkdir observatory-<name>-svc && cd $_
go mod init github.com/luciocarvalhojr/observatory-<name>-svc
mkdir -p cmd/api internal/handler internal/domain internal/repository
```
3. **Copy workflows** from an existing service (e.g. `auth-svc`)
4. **Add Helm chart** to `helm-charts/charts/<name>-svc/`
5. **Add ArgoCD Application** to `k8s-home-lab/argocd/<name>-svc.yaml`
6. **Add service docs** to `observatory/docs/services/<name>-svc.md`
7. **Open GitHub issue** using the "New Service" template

---

## Commit Convention

This project uses [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add slack notification channel
fix: resolve alert deduplication race condition
chore: update go dependencies
docs: add runbook for token validation
test: add unit tests for rule evaluator
refactor: extract alert polling to separate worker
```

Commits drive semantic-release versioning automatically.

---

## Branch Strategy

```
main          → protected, requires PR + passing CI
feature/*     → new features
fix/*         → bug fixes
chore/*       → maintenance
docs/*        → documentation only
```

---

## ADR Process

For any significant architectural decision:
1. Open an issue using the ADR template
2. Discuss in the issue
3. Once accepted, write the ADR in `docs/adr/NNN-title.md`
4. Reference it from the relevant service docs

---

## Local Development

```bash
# Run any service locally
cd observatory-<service>-svc
go run cmd/api/main.go

# Run with Docker
docker build -t <service>:dev .
docker run -p 808X:808X <service>:dev

# Run all security checks locally
golangci-lint run
gosec ./...
govulncheck ./...
go test -v -race -cover ./...
```

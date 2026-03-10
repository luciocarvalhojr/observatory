# ADR-001: Release Strategy

## Status
`DRAFT` — semantic-release in use today, migration to release-please planned

---

## Context

Each Observatory service needs a consistent, automated release process that:
- Bumps versions from conventional commits
- Generates a CHANGELOG
- Creates a GitHub Release
- Triggers downstream build pipeline (Docker, Helm)

The current approach uses **semantic-release** with a local `.releaserc.yaml`
in each service repo. This works but has two problems:
- Config is duplicated across every service repo (drift risk)
- semantic-release has no native mechanism to share config via URL

---

## Decision

### Current (interim)
Use **semantic-release** with a standard `.releaserc.yaml` copied into each
service at scaffold time. Renovate monitors for drift across repos.

The `.releaserc.yaml` standard is defined once in:
```
observatory/configs/.releaserc.yaml  ← source of truth (reference only)
```
Each service copy must match this file. Any divergence is caught by Renovate.

### Future (planned)
Migrate to **release-please** (Google) with a central manifest in `observatory`:

```
observatory/
  release-please-config.json          ← all services defined here
  .release-please-manifest.json       ← versions tracked here
  .github/workflows/release-please.yml
```

Benefits over semantic-release:
- No Node.js dependency
- Central version tracking across all services in one file
- PR-based releases (review before releasing)
- Native GitHub integration
- Pairs with GoReleaser for the actual build/sign/push step

---

## TODO — migration to release-please

- [ ] Add `release-please-config.json` to `observatory`
- [ ] Add `.release-please-manifest.json` to `observatory`
- [ ] Add `.github/workflows/release-please.yml` to `observatory`
- [ ] Add `configs/.goreleaser.base.yaml` to `observatory` (shared build base)
- [ ] Migrate `observatory-auth-svc` from semantic-release → GoReleaser
- [ ] Validate full flow: commit → Release PR → merge → tag → GoReleaser → GHCR
- [ ] Migrate remaining services as they are built

---

## Consequences

**Now:** simple, working, slight drift risk manageable via Renovate.

**After migration:** zero drift, central visibility of all service versions,
production-grade release pipeline with image signing and SBOM out of the box.
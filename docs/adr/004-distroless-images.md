# ADR-004: Distroless container images

## Status
Accepted

## Context
Container images need to be as small and secure as possible. Traditional base images
(alpine, debian) include shells, package managers, and utilities that are never needed
at runtime and increase the attack surface.

## Decision
Use `gcr.io/distroless/static-debian12:nonroot` as the final stage for all services.

## Consequences
- No shell — impossible to exec into a running container (reduces blast radius)
- No package manager — no way to install tools after deployment
- Runs as nonroot:nonroot by default
- Image size ~5MB vs ~50MB for alpine-based
- Trivy scans report significantly fewer CVEs
- Debugging requires ephemeral debug containers (`kubectl debug`)

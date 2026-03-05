# ADR-001: Go as the primary service language

## Status
Accepted

## Context
We need a language for all six microservices that is fast, has a small memory footprint,
compiles to a static binary (ideal for distroless containers), and has strong support
for building HTTP APIs and concurrent workloads.

## Decision
Use Go 1.24 with Gin Gonic for all services.

## Consequences
- Static binaries compile to ~10-15MB, ideal for distroless images
- Strong concurrency model (goroutines) for the alert polling worker
- Single language across all services reduces cognitive overhead
- Gin is lightweight — no magic, explicit routing
- All tooling (golangci-lint, gosec, govulncheck) is Go-native

# ADR-005: Keyless image signing with Cosign

## Status
Accepted

## Context
We need to cryptographically prove that container images deployed to the cluster
were built by our CI pipeline and not tampered with. Traditional approaches require
managing a private signing key, which creates a secret management burden.

## Decision
Use Cosign keyless signing via GitHub OIDC and Sigstore Fulcio/Rekor.
No private keys are stored anywhere — the GitHub Actions OIDC token proves
the identity of the workflow that signed the image.

## Verification
Kyverno verifies every image before scheduling a pod:
```yaml
attestors:
  - keyless:
      subject: "https://github.com/luciocarvalhojr/observatory-*/.github/workflows/release.yml@refs/heads/main"
      issuer:  "https://token.actions.githubusercontent.com"
```

## Consequences
- Zero secret management — no private keys to rotate or leak
- Signatures are publicly verifiable in Sigstore Rekor transparency log
- Kyverno blocks any unsigned or incorrectly-signed image from running
- Requires internet access from the cluster to verify signatures
- Sigstore Rekor is a public log — image digests are public (acceptable for open source)

# Observatory — Shared Configs

This directory is the **single source of truth** for all tool configurations
shared across Observatory microservices. Any change here propagates to all
services automatically — no manual sync needed.

## Files

| File | Tool | Purpose |
|---|---|---|
| `.golangci.yml` | golangci-lint | Lint ruleset for all Go services |
| `.releaserc.yaml` | semantic-release | Release & changelog config |
| `renovate/base.json` | Renovate | Dependency update policy |
| `.go-version` | Go toolchain | Pinned Go version |

---

## How to use in a service repo

### `.golangci.yml`
```yaml
# observatory-*-svc/.golangci.yml
version: "2"
extends:
  - "https://raw.githubusercontent.com/luciocarvalhojr/observatory/main/configs/.golangci.yml"

# Override only what's service-specific, e.g.:
# linters-settings:
#   wrapcheck:
#     ignorePackageGlobs:
#       - "github.com/some/extra-package"
```

### `.releaserc.yaml`
```yaml
# observatory-*-svc/.releaserc.yaml
extends: "https://raw.githubusercontent.com/luciocarvalhojr/observatory/main/configs/.releaserc.yaml"

# Override only if needed, e.g. add extra assets:
# plugins:
#   - - "@semantic-release/exec"
#     - publishCmd: "some extra publish step"
```

### `renovate.json`
```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>luciocarvalhojr/observatory//configs/renovate/base.json"
  ]
}
```

### `.go-version`
Do **not** copy this file into each service. The Go version is enforced
centrally by the reusable CI workflow, which reads this file from the
`observatory` repo directly.

---

## Updating a config

1. Open a PR in `observatory` with your change
2. After merge, all services pick it up automatically on their next CI run
3. No PRs needed in individual service repos

## Adding a service-level override

If one service genuinely needs a different rule, add it **in that service's
local config file** using the `extends` + override pattern shown above.
Document why in a comment so future maintainers understand the divergence.
# bloqr-actions

Shared GitHub Actions for BloqrAI repositories. This repository is public and contains **no secrets**.

## `load-secrets`

Loads `op://` references from 1Password into environment variables, so a repository's only GitHub Actions secret is `OP_SERVICE_ACCOUNT_TOKEN`.

```yaml
jobs:
    deploy:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@<sha>
            - uses: BloqrAI/bloqr-actions/load-secrets@<sha> # vX.Y.Z
              with:
                  service-account-token: ${{ secrets.OP_SERVICE_ACCOUNT_TOKEN }}
            # NAME=... from .github/op-secrets.env are now masked environment variables for later steps.
```

Pin by full commit SHA, like every other action.

### The reference file

Each consuming repository keeps `.github/op-secrets.env`: references only, never values. Start it with a header that ties it to 1Password so a human can match keys:

```
# 1Password service account: ci-<repo>          (vault: Claude-CI-<repo>)
# Token item (Claude-Dev):   1password/ci-sa-<repo>/dev
# GitHub secret:             OP_SERVICE_ACCOUNT_TOKEN (the only one)
CLOUDFLARE_API_TOKEN=op://Claude-CI-<repo>/cloudflare/credential
```

### Naming (must match in 1Password and in the repo)

| Thing | Name |
|---|---|
| Vault | `Claude-CI-<repo>`, for example `Claude-CI-bloqr-core` (the `Claude-` prefix keeps it inside the vaults the agent may use) |
| Service account | `ci-<repo>`, read access to its own vault only |
| Token item, stored in `Claude-Dev` | `1password/ci-sa-<repo>/dev` |
| Secret items inside the vault | `<system>/<purpose>/<env>` |
| GitHub Actions secret | `OP_SERVICE_ACCOUNT_TOKEN` (sole secret) |

The registry that lists every repository, service account and vault lives in `BloqrAI/bloqr-artifacts` (`strategy/ci-secrets-registry.md`), not here, because vault names do not belong in a public repository.

### Caveats

- Values are exported as environment variables for the rest of the job. They are masked in logs, but every later step in that job, including third-party actions, can read them. Load only in jobs that need them.
- Fork pull requests do not receive repository secrets. Jobs that need secrets must run on trusted events; do not add skip logic that lets them pass without the secret.
- Never run this action in a `pull_request_target` job that checks out pull-request code.

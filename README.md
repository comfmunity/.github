# comfmunity/.github

Shared GitHub configuration for the Comfmunity organisation.

## Reusable workflows

### `node-ci.yml`

Installs with `npm ci` (authenticated to GitHub Packages for the `@comfmunity` scope), then runs `lint`, `typecheck`, `build` and `test` when the package defines them. Libraries also get `npm pack --dry-run`.

```yaml
# .github/workflows/ci.yml in any comfmunity repo
name: CI
on:
  pull_request:
  push:
    branches: [main]
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
jobs:
  ci:
    uses: comfmunity/.github/.github/workflows/node-ci.yml@main
    with:
      is-library: true # false for the bot
    secrets: inherit
```

| Input | Default | |
|---|---|---|
| `node-versions` | `["24"]` | JSON array for the test matrix |
| `is-library` | `false` | Verify the published tarball |
| `run-build` | `true` | Set `false` when the build needs live services (the site prerenders from MongoDB) |

## Setup

This repo is public so every org repo can call it; it holds no secrets.

1. **Org secret `PACKAGES_READ_TOKEN`**: a fine-grained PAT (or classic PAT with `read:packages`) that can read every `@comfmunity` package. Org Settings → Secrets and variables → Actions → New organization secret, visible to all repositories (`gh secret set PACKAGES_READ_TOKEN --org comfmunity --visibility all`). Repos that still have the old `NPM_GITHUB_TOKEN` secret keep working as a fallback.

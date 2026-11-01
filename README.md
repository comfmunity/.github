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

### `release.yml`

Runs [release-please](https://github.com/googleapis/release-please) on every push to `main`. It keeps a release PR open that bumps the version and updates `CHANGELOG.md` from conventional commits (`feat:` → minor, `fix:` → patch, `feat!:` → major). Merging that PR tags the release, creates a GitHub Release and publishes the package to GitHub Packages.

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
permissions:
  contents: write
  pull-requests: write
  packages: write
jobs:
  release:
    uses: comfmunity/.github/.github/workflows/release.yml@main
    secrets: inherit
```

The repo also needs `release-please-config.json` and `.release-please-manifest.json` at its root.

Release PRs are opened with `GITHUB_TOKEN`, so CI doesn't run on them. They only change the version and changelog, and the publish job builds from scratch anyway.

## Setup

This repo is public so every org repo can call it; it holds no secrets.

1. **Repo secret `PACKAGES_READ_TOKEN`** in every repo that installs `@comfmunity` packages: a classic PAT with `read:packages` (fine-grained tokens don't work with the npm registry). It has to be a repo secret because private repos on the Free plan can't read organization secrets.
2. **Actions access for publishing**: on each package's settings page (Manage Actions access), give its source repo the Write role so `GITHUB_TOKEN` can publish.
3. **Repo Dependabot secret `DEPENDABOT_GITHUB_PACKAGES_TOKEN`** (same token, `gh secret set -a dependabot`) so Dependabot can see `@comfmunity` releases. The org-level one doesn't reach private repos on Free.
4. **Allow Actions to open PRs**: org and repo Settings → Actions → General → Workflow permissions → *Allow GitHub Actions to create and approve pull requests*.

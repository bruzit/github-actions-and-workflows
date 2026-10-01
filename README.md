# GitHub Actions and Workflows

Reusable GitHub Actions workflows forming the CI baseline for every BruzIT repository: semantic-release versioning and MegaLinter linting.

## Features

- [Semantic Release Action](#semantic-release-composite-action)
- [Semantic Release Workflow](#reusable-semantic-release-workflow)
- [MegaLinter Workflow](#reusable-megalinter-workflow)

### Semantic Release Composite Action

[Semantic Release composite action](semantic-release/action.yaml) with the same steps as the reusable workflow. It checks out the repository with the GitHub App token, so the changelog commit and the major tags are pushed as the GitHub App.

### Reusable Semantic Release Workflow

Reusable [Semantic Release workflow](.github/workflows/semantic-release.yaml) using the Conventional Commits preset to automate versioning, tags with SemVer and major tag, generates [GitHub releases](https://github.com/bruzit/github-actions-and-workflows/releases), and updates the [CHANGELOG](CHANGELOG.md).

### Reusable MegaLinter Workflow

Reusable [MegaLinter workflow](.github/workflows/megalinter.yaml) linting pull requests with the `terraform` flavor, auto-committing fixable findings. Linters run with MegaLinter's default rules, except zizmor, whose [`zizmor.yaml`](zizmor.yaml) allows tag-pinned actions.

## Usage

### Use Semantic Release Action

Create a workflow, for example, `.github/workflows/semantic-release.yaml`; the job's `release` environment holds `GH_SEM_REL_APP_ID` and `GH_SEM_REL_APP_PEM_FILE`:

```yaml
---
name: Semantic Release

on:
  push:
    branches:
      - main

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    environment: release
    permissions:
      contents: write
      issues: write
      pull-requests: write
    concurrency:
      group: release-${{ github.ref }}
      cancel-in-progress: false
    steps:
      - name: Semantic Release
        uses: bruzit/github-actions-and-workflows/semantic-release@v0
        with:
          app-id: ${{ vars.GH_SEM_REL_APP_ID }}
          app-private-key: ${{ secrets.GH_SEM_REL_APP_PEM_FILE }}
          plugins: "@semantic-release/exec" # OPTIONAL Space-separated list of additional semantic-release plugins to install.
```

The action checks out the repository itself. A local `uses: ./semantic-release` needs a prior `actions/checkout` with `persist-credentials: false`.

### Use Semantic Release Workflow

Create a workflow, for example, `.github/workflows/semantic-release.yaml`:

```yaml
---
name: Semantic Release

on:
  push:
    branches:
      - main

jobs:
  release:
    name: Release
    uses: bruzit/github-actions-and-workflows/.github/workflows/semantic-release.yaml@v0
    permissions:
      contents: write
      issues: write
      pull-requests: write
    with:
      GH_SEM_REL_APP_ID: ${{ vars.GH_SEM_REL_APP_ID }}
      semantic_release_plugins: "@semantic-release/exec" # OPTIONAL Space-separated list of additional semantic-release plugins to install.
    secrets:
      GH_SEM_REL_APP_PEM_FILE: ${{ secrets.GH_SEM_REL_APP_PEM_FILE }}
```

To create a GitHub App and a GitHub App Installation:

- GitHub
  - _Organization_ / Settings / Developer settings / GitHub Apps
    - **New GitHub App**
      - Create GitHub App
        - GitHub App name: _name_
        - Description: _description_
        - Homepage URL: _homepage URL_
      - Webhook
        - Active: off
      - Permissions
        - Organization permissions
          - Contents: Read and write
          - Issues: Read and write
          - Pull requests: Read and write
        - Where can this GitHub App be installed?: _choose what suits you best_
      - **Create GitHub App**
    - _your app_
      - General
        - **Generate a private key**
      - Install App
        - _your organization_: **Install**
  - _Repository_ / Settings / Secrets and variables / Actions
    - Secrets
      - Repository secrets / **New repository secret**
        - Name: `GH_SEM_REL_APP_PEM_FILE`
        - Secret: _content of the PEM file_
        - **Add secret**
    - Variables
      - Repository variables / **New repository variable**
        - Name: `GH_SEM_REL_APP_ID`
        - Value: _GitHub App ID_
        - **Add variable**

Configure Semantic Release in the repository, for example like this repository's [`.releaserc.yaml`](.releaserc.yaml).

### Use MegaLinter Workflow

Create `.github/workflows/megalinter.yaml`:

```yaml
---
name: MegaLinter

on:
  pull_request:

jobs:
  megalinter:
    name: MegaLinter
    uses: bruzit/github-actions-and-workflows/.github/workflows/megalinter.yaml@v0
    permissions:
      contents: write
      pull-requests: write
    # with:
    #   validate_all_codebase: true # OPTIONAL Lint the whole repository, not only the changed files.
```

Create `.mega-linter.yml` listing the linters for the repository, for example:

```yaml
---
ENABLE:
  - ACTION
  - MARKDOWN
  - YAML
```

Add `ANSIBLE`, `BASH` or `TERRAFORM` to `ENABLE` as needed; ansible-lint additionally requires an `.ansible-lint` file. Copy [`zizmor.yaml`](zizmor.yaml) into the repository root and add `megalinter-reports/` to `.gitignore`.

Pull requests lint only changed files. To also lint the whole repository weekly, for example to catch newly published advisories for pinned action tags, create `.github/workflows/megalinter-scheduled.yaml`:

```yaml
---
name: MegaLinter Scheduled

on:
  schedule:
    - cron: "0 6 * * 1"
  workflow_dispatch:

jobs:
  megalinter:
    name: MegaLinter
    uses: bruzit/github-actions-and-workflows/.github/workflows/megalinter.yaml@v0
    permissions:
      contents: write
      pull-requests: write
    with:
      validate_all_codebase: true
```

Fixes are not committed outside pull requests; findings fail the run.

## Linting

This repository is linted by its own [MegaLinter workflow](.github/workflows/megalinter.yaml). Run locally (needs Docker):

```bash
# report issues
docker run --rm -v "$PWD":/tmp/lint oxsecurity/megalinter-terraform:v10

# auto-fix where possible
docker run --rm -e APPLY_FIXES=all -v "$PWD":/tmp/lint oxsecurity/megalinter-terraform:v10
```

## Copyright and Licensing

[MIT License](LICENSE)  
Copyright © 2026 Martin Bružina

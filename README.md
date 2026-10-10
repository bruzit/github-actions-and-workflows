# GitHub Actions and Workflows

Reusable GitHub Actions and workflows forming the CI baseline for every BruzIT repository: semantic-release versioning and MegaLinter linting.

## Features

- [Semantic Release Action](#semantic-release-composite-action)
- [MegaLinter Workflow](#reusable-megalinter-workflow)

### Semantic Release Composite Action

[Semantic Release composite action](semantic-release/action.yaml) using the Conventional Commits preset to automate versioning, tags with SemVer and major tag, generates [GitHub releases](https://github.com/bruzit/github-actions-and-workflows/releases), and updates the [CHANGELOG](CHANGELOG.md). It checks out the repository with the GitHub App token, so the changelog commit and the major tags are pushed as the GitHub App.

### Reusable MegaLinter Workflow

Reusable [MegaLinter workflow](.github/workflows/megalinter.yaml) linting pull requests with the `terraform` flavor, auto-committing fixable findings, then scanning the full git history for secrets with [gitleaks](https://github.com/gitleaks/gitleaks). Linters run with MegaLinter's default rules, except markdownlint, whose [`.markdownlint.yaml`](.markdownlint.yaml) applies markdownlint's default rules without line length, and zizmor, whose [`zizmor.yaml`](zizmor.yaml) allows tag-pinned actions.

## Usage

The callers below are those of [bruzit/.github](https://github.com/bruzit/.github/tree/main/.github/workflows).

### Use Semantic Release Action

Create `.github/workflows/semantic-release.yaml`:

```yaml
---
name: Semantic Release

on:
  push:
    branches:
      - main

concurrency:
  group: ${{ github.workflow }}

jobs:
  release:
    name: Release
    environment: release
    runs-on: ubuntu-slim
    permissions: {}
    steps:
      - name: Semantic Release
        uses: bruzit/github-actions-and-workflows/semantic-release@v0
        with:
          app-id: ${{ vars.GH_SEM_REL_APP_ID }}
          app-private-key: ${{ secrets.GH_SEM_REL_APP_PEM_FILE }}
```

The optional `plugins` input takes a space-separated list of additional semantic-release plugins to install, e.g. `"@semantic-release/exec"`.

The action checks out the repository itself. A local `uses: ./semantic-release` needs a prior `actions/checkout` with `persist-credentials: false`.

The job needs no `GITHUB_TOKEN` permissions. The action releases with the GitHub App token only: it creates an installation token for the calling repository with `contents`, `issues` and `pull-requests` write, and uses it for the checkout, the changelog commit and tag pushes, the GitHub release, and the release comments on issues and pull requests. There is no `GITHUB_TOKEN` fallback; `app-id` and `app-private-key` are required.

The `release` environment, its `GH_SEM_REL_APP_ID` variable and `GH_SEM_REL_APP_PEM_FILE` secret are provisioned by [GitHub Organization as Code](https://github.com/bruzit/github-organization-as-code) from [`bruzit.yaml`](https://github.com/bruzit/.github/blob/main/bruzit.yaml), not created by hand and not as repository variables or secrets. The organization environment is added to every repository, deployable from the default branch only, and the App bypasses the default-branch ruleset to push the changelog commit:

```yaml
---
organization:
  environments:
    release:
      deployment_branches:
        - ~DEFAULT_BRANCH
      variables:
        GH_SEM_REL_APP_ID: "123456"
      secrets:
        - GH_SEM_REL_APP_PEM_FILE
  rulesets:
    default-branch:
      bypass_apps:
        - 123456
```

GitHub Organization as Code creates the secret with a placeholder value only; set the App's private key once per repository:

```shell
gh secret set GH_SEM_REL_APP_PEM_FILE --env release --repo OWNER/REPOSITORY < private-key.pem
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
        - Repository permissions
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
```

The caller grants `contents: write` for the push of the auto-fix commit to the pull request branch and `pull-requests: write` for MegaLinter's pull request comments, both with `GITHUB_TOKEN`; a called workflow's `GITHUB_TOKEN` cannot exceed the caller's permissions. The workflow needs no secrets.

Create `.mega-linter.yaml` listing the linters for the repository, for example:

```yaml
---
ENABLE:
  - ACTION
  - MARKDOWN
  - YAML
```

Add `ANSIBLE`, `BASH` or `TERRAFORM` to `ENABLE` as needed; ansible-lint additionally requires an `.ansible-lint` file. Copy [`.markdownlint.yaml`](.markdownlint.yaml) and [`zizmor.yaml`](zizmor.yaml) into the repository root and add `megalinter-reports/` to `.gitignore`.

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

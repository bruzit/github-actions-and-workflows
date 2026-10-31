# GitHub Actions and Workflows

[![License](https://img.shields.io/github/license/bruzit/github-actions-and-workflows)](LICENSE)
[![Last commit](https://img.shields.io/github/last-commit/bruzit/github-actions-and-workflows)](https://github.com/bruzit/github-actions-and-workflows/commits)
[![MegaLinter](https://github.com/bruzit/github-actions-and-workflows/actions/workflows/megalinter-pull-request.yaml/badge.svg)](https://github.com/bruzit/github-actions-and-workflows/actions/workflows/megalinter-pull-request.yaml)

Reusable GitHub Actions workflows forming the CI baseline for every BruzIT repository: semantic-release versioning and MegaLinter linting.

## Features

### Semantic Release Composite Action

[`semantic-release/action.yaml`](semantic-release/action.yaml) composite action using the Conventional Commits preset to automate versioning, tags with SemVer and major tag, generates [GitHub releases](../../releases), and updates [`CHANGELOG.md`](CHANGELOG.md). It checks out the repository with the GitHub App token, so the changelog commit and the major tags are pushed as the GitHub App.

### Reusable MegaLinter Workflow

Reusable [`.github/workflows/megalinter.yaml`](.github/workflows/megalinter.yaml) workflow linting pull requests with the `terraform` flavor, auto-committing fixable findings, then scanning the full git history for secrets with gitleaks ([gitleaks/gitleaks GitHub repository](https://github.com/gitleaks/gitleaks)). Linters run with MegaLinter's default rules, except markdownlint, whose [`.markdownlint.yaml`](.markdownlint.yaml) applies markdownlint's default rules without line length, and zizmor, whose [`zizmor.yaml`](zizmor.yaml) allows tag-pinned actions.

## Installation and Setup

Nothing to install: callers reference the action and the workflow at `@v0`. Each needs a GitHub App, an environment and a caller workflow; see Usage.

## Usage

The callers below are those of [bruzit/.github workflows](https://github.com/bruzit/.github/tree/main/.github/workflows).

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

The `release` environment, its `GH_SEM_REL_APP_ID` variable and `GH_SEM_REL_APP_PEM_FILE` secret are provisioned by [BruzIT GitHub Organization as Code](https://github.com/bruzit/github-organization-as-code) from [`bruzit.yaml` of bruzit/.github](https://github.com/bruzit/.github/blob/main/bruzit.yaml), not created by hand and not as repository variables or secrets. The organization environment is added to every repository, deployable from the default branch only, and the App bypasses the default-branch ruleset to push the changelog commit:

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

BruzIT GitHub Organization as Code creates the secret with a placeholder value only; set the App's private key once per repository:

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

Configure semantic-release in the repository, for example like this repository's [`.releaserc.yaml`](.releaserc.yaml).

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
      contents: read
      pull-requests: write
    secrets:
      GH_AUTOFIX_APP_PEM_FILE: ${{ secrets.GH_AUTOFIX_APP_PEM_FILE }}
```

The caller grants `contents: read` for the checkout and `pull-requests: write` for MegaLinter's pull request comments, both with `GITHUB_TOKEN`; a called workflow's `GITHUB_TOKEN` cannot exceed the caller's permissions. Linters run without write credentials.

Fixes are pushed as a GitHub App, because commits pushed with `GITHUB_TOKEN` trigger no workflows and would leave the pull request head without the required checks. Only after MegaLinter succeeds on a pull request from the same repository with applied fixes, the workflow creates an installation token for the calling repository with `contents: write`, commits `style: apply megalinter fixes` as the App and pushes it to the pull request branch; the push re-runs the checks. The job uses the `autofix` environment without creating deployments; its `GH_AUTOFIX_APP_ID` variable and `GH_AUTOFIX_APP_PEM_FILE` secret are provisioned by BruzIT GitHub Organization as Code like the `release` environment above. The caller must pass the secret explicitly: a called workflow reads an environment secret only under a name its caller passes, and `secrets: inherit` fails zizmor. The App needs only `Contents: write` and no ruleset bypass. Pull requests from forks and Dependabot get no secrets, so MegaLinter only reports their findings.

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
      contents: read
      pull-requests: write
    with:
      validate_all_codebase: true
```

Fixes are not committed outside pull requests; findings fail the run.

## Support

Community, best effort; [open a GitHub issue](../../issues).

## Development

### Linting

This repository is linted by its own [`.github/workflows/megalinter.yaml`](.github/workflows/megalinter.yaml) workflow. Run locally (needs Docker):

```bash
# report issues
docker run --rm -v "$PWD":/tmp/lint oxsecurity/megalinter-terraform:v10

# auto-fix where possible
docker run --rm -e APPLY_FIXES=all -v "$PWD":/tmp/lint oxsecurity/megalinter-terraform:v10
```

## Contributing

Open a pull request to `main`. Commits follow the [Conventional Commits 1.0.0 specification](https://www.conventionalcommits.org/en/v1.0.0/), from which semantic-release sets versions and the changelog automatically.

## License and Copyright

MIT License: [`LICENSE`](LICENSE)  
Copyright © 2026 Martin Bružina

## Stats

| Metric           | Badges                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Commits          | [![Commits total](https://img.shields.io/github/commit-activity/t/bruzit/github-actions-and-workflows?label=commits%20total)](https://github.com/bruzit/github-actions-and-workflows/commits/main)                                                                                                                                                                                                                                                                                                                     |
| Issues           | [![Issues open](https://img.shields.io/github/issues/bruzit/github-actions-and-workflows?label=issues)](https://github.com/bruzit/github-actions-and-workflows/issues) [![Issues total](https://img.shields.io/github/issues-search?query=repo%3Abruzit%2Fgithub-actions-and-workflows%20is%3Aissue&label=issues%20total)](https://github.com/bruzit/github-actions-and-workflows/issues?q=is%3Aissue)                                                                                                                 |
| Pull requests    | [![Pull requests open, not draft](https://img.shields.io/github/issues-search?query=repo%3Abruzit%2Fgithub-actions-and-workflows%20is%3Apr%20is%3Aopen%20draft%3Afalse&label=PRs%20open)](https://github.com/bruzit/github-actions-and-workflows/pulls?q=is%3Apr+is%3Aopen+draft%3Afalse) [![Pull requests total](https://img.shields.io/github/issues-search?query=repo%3Abruzit%2Fgithub-actions-and-workflows%20is%3Apr&label=PRs%20total)](https://github.com/bruzit/github-actions-and-workflows/pulls?q=is%3Apr) |
| Checks on `main` | [![Checks on main](https://img.shields.io/github/check-runs/bruzit/github-actions-and-workflows/main?label=checks%20%28main%29)](https://github.com/bruzit/github-actions-and-workflows/actions?query=branch%3Amain)                                                                                                                                                                                                                                                                                                   |
| Releases         | [![Latest release](https://img.shields.io/github/v/release/bruzit/github-actions-and-workflows)](https://github.com/bruzit/github-actions-and-workflows/releases/latest) [![Commits since latest release](https://img.shields.io/github/commits-since/bruzit/github-actions-and-workflows/latest)](https://github.com/bruzit/github-actions-and-workflows/releases)                                                                                                                                                    |
| Community        | [![Stargazers](https://img.shields.io/github/stars/bruzit/github-actions-and-workflows)](https://github.com/bruzit/github-actions-and-workflows/stargazers) [![Forks](https://img.shields.io/github/forks/bruzit/github-actions-and-workflows)](https://github.com/bruzit/github-actions-and-workflows/forks) [![Contributors](https://img.shields.io/github/contributors/bruzit/github-actions-and-workflows)](https://github.com/bruzit/github-actions-and-workflows/graphs/contributors)                            |

# GitHub Actions and Workflows

A collection of GitHub composite actions and reusable workflows

## Features

- [Semantic Release Workflow](#semantic-release-workflow)

### Reusable Semantic Release Workflow

Reusable [Semantic Release workflow](.github/workflows/semantic-release.yaml) using the Conventional Commits preset to automate versioning, tags with SemVer and major tag, generates [GitHub releases](https://github.com/bruzit/github-actions-and-workflows/releases), and updates the [CHANGELOG](CHANGELOG.md).

## Usage

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
      GH_APP_ID: ${{ vars.GH_APP_ID }}
      semantic_release_plugins: "@semantic-release/exec" # OPTIONAL Space-separated list of additional semantic-release plugins to install.
    secrets:
      GH_APP_PEM_FILE: ${{ secrets.GH_APP_PEM_FILE }}
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
        - Name: `GH_APP_PEM_FILE`
        - Secret: _content of the PEM file_
        - **Add secret**
    - Variables
      - Repository variables / **New repository variable**
        - Name: `GH_APP_ID`
        - Value: _GitHub App ID_
        - **Add variable**

Configure Semantic Release in the repository, for example, `.releaserc.yaml`:

```yaml
---
branches:
  - main
plugins:
  - - "@semantic-release/commit-analyzer"
    - preset: conventionalcommits
  - - "@semantic-release/release-notes-generator"
    - preset: conventionalcommits
  - "@semantic-release/github"
  - - "@semantic-release/changelog"
    - changelogTitle: '# Changelog'
  - - "@semantic-release/git"
    - assets:
      - CHANGELOG.md
  - semantic-release-major-tag
```

## Copyright and Licensing

[MIT License](LICENSE)  
Copyright © 2026 Martin Bružina

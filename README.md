# Arbeidstilsynet/action-check-changes

Action to check for changes to files. The main mode is automatically comparing to last successful workflow run in the same branch. This action can also be used for basic comparison to a chosen `base-ref`, skipping the lookup of workflow runs.

This action can be used for safe continuous delivery/deployment taking into consideration previous runs.

Resolution order for base commit:

1. `base-ref` if provided.
2. Last successful run of `workflow-file` (explicit or auto-detected).
3. Repository root (first) commit.

## Versioning

This repository uses a simple versioning system based on the `VERSION` file.
When you update the `VERSION` file and push to `main`, a Git tag with that version is created or updated automatically by the workflow.
If you have to make breaking changes to the action, bump the version.

## Requirements

Requires full commit history. When configuring [actions/checkout](https://github.com/actions/checkout), make sure to set `fetch-depth: 0`.

The job must have permission for `actions: read` for the action to retrieve run history through the GitHub API.

## Inputs

| Name            | Required | Default       | Description                                                                                                                                                                 |
|-----------------|----------|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `include`       | Yes      |               | Newline separated Git pathspecs (globs) to include in the diff scope (e.g. `src/`, `app/**/*.ts`).                                                                          |
| `exclude`       | No       | (empty)       | Newline separated Git pathspecs to exclude. Each is applied as `:(exclude)<pattern>`.                                                                                       |
| `workflow-file` | No       | (auto-detect) | Workflow filename whose last successful run determines the base commit. If omitted, auto-detected from `GITHUB_WORKFLOW_REF`; if none found falls back to repo root commit. |
| `base-ref`      | No       | (empty)       | Explicit Git ref (commit SHA, branch, tag, or remote ref like `origin/main`) to use as base. When set, workflow run lookup is skipped.                                      |

## Outputs

| Name               | Description                                                                                           |
|--------------------|-------------------------------------------------------------------------------------------------------|
| `changes_detected` | `true` if any included (and not excluded) paths changed between base and head.                        |
| `base_sha`         | The base commit SHA used for the diff (last successful run’s head, or fallback).                      |
| `head_sha`         | The current commit SHA.                                                                               |
| `changed_files`    | Newline separated list of changed files after exclusions. URL-escaped newlines in raw output context. |

## Usage

```yaml
on:
  push:
    branches:
      - main

jobs:
  check:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      actions: read
    outputs:
      changed: ${{ steps.changes.outputs.changes_detected }}
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0

      - uses: Arbeidstilsynet/action-check-changes@v1
        id: changes
        with:
          include: |
            apps/web
            .github/workflows/deploy.yml
          exclude: |
            **/README.md
            apps/web/docs/

  deploy:
    needs: check
    if: needs.check.outputs.changed == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "insert deploy here"
```

### Using an explicit base ref (e.g. always compare to main)

```yaml
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0

      - uses: Arbeidstilsynet/action-check-changes@v1
        id: changes
        with:
          base-ref: origin/main
          include: |
            apps/web
            .github/workflows/ci.yml
```

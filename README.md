# Arbeidstilsynet/action-noop

> **Note:** This is a template repository for creating simple composite GitHub Actions.
>
> After creating the new repo you should enable "Allow auto-merge" and "Automatically delete head branches" in Settings -> General -> Pull Requests.

A no-op GitHub Action that echoes inputs and sets outputs.

## Versioning

This repository uses a simple versioning system based on the `VERSION` file.
When you update the `VERSION` file and push to `main`, a Git tag with that version is created or updated automatically by the workflow.
If you have to make breaking changes to the action, bump the version.

## Requirements

- None

## Inputs

| Name         | Description        | Required | Default         |
|--------------|--------------------|----------|-----------------|
| `input-one`  | First input value  | Yes      |                 |
| `input-two`  | Second input value | No       | `default-value` |

## Outputs

| Name        | Description           |
|-------------|-----------------------|
| `output-one`| Echo of input-one     |
| `output-two`| Echo of input-two     |

## Usage

```yaml
name: Example No-op Action Usage

on:
  push:
    branches:
      - main

jobs:
  noop-job:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: Arbeidstilsynet/action-noop@v1
        id: noop
        with:
          input-one: "Hello"
          input-two: "World"

      - name: Show outputs
        run: |
          echo "Output one: ${{ steps.noop.outputs.output-one }}"
          echo "Output two: ${{ steps.noop.outputs.output-two }}"
```

# skolar-action

Run [`skolar update`](https://github.com/h3y6e/skolar) and create a pull request if needed.

## Usage

```yaml
name: Update Skills

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  update-skills:
    runs-on: ubuntu-slim
    timeout-minutes: 5

    steps:
      - uses: actions/checkout@v7

      - uses: h3y6e/skolar-action@v0.0.2
        id: skolar

      - name: Enable auto-merge
        if: steps.skolar.outputs.pr-number != ''
        run: gh pr merge "${{ steps.skolar.outputs.pr-number }}" --auto --merge
        env:
          GH_TOKEN: ${{ github.token }}
```

The action downloads the [skolar](https://github.com/h3y6e/skolar) for the runner, and runs `skolar update -y`.
Changes are pushed to `branch` and opened as a pull request against the checked-out branch, or update the existing one.

Pull requests created with the workflow `GITHUB_TOKEN` do not trigger other workflows. To run CI on them, pass a GitHub App installation token with `token`.

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `token` | no | workflow `github.token` | GitHub token used for `gh`, `git push`, and pull request creation. |
| `version` | no | latest release | skolar release tag to install, e.g. `v2026.10.0`. |
| `dir` | no | | Destination directory for installed skills. When empty, skolar uses its default. |
| `branch` | no | `skolar-action/patch` | The pull request branch name. |
| `commit-message` | no | `[skolar-action] automated change` | The message to use when committing changes. |
| `title` | no | `Changes by skolar-action` | The title of the pull request. |
| `body` | no | `Automated changes by [skolar-action](https://github.com/h3y6e/skolar-action)` | The body of the pull request. |

## Outputs

| Name | Description |
| --- | --- |
| `pr-number` | The pull request number. Empty when there were no changes. |
| `pr-url` | The pull request URL. Empty when there were no changes. |

## Requirements

- Linux or macOS runner with GitHub CLI installed.
- The token must be able to push and create pull requests (`contents: write`, `pull-requests: write`).

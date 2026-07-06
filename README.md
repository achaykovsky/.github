# Central GitHub configuration

Shared CI workflows, ruleset templates, and adoption docs for Python/Poetry repositories under [achaykovsky](https://github.com/achaykovsky).

## Quick start

1. Add a thin [CI wrapper](docs/ADOPT.md#1-add-the-ci-workflow) to your repo.
2. Add [dev dependencies](docs/ADOPT.md#2-add-dev-dependencies) (`ruff`, `mypy`, `pytest`, `pip-audit`).
3. Open a PR and wait for CI to run once (registers check names in GitHub).
4. [Import the ruleset](docs/REQUIRED_CHECKS.md#enable-the-ruleset) to block merges until all checks pass.

Full walkthrough: [docs/ADOPT.md](docs/ADOPT.md).

## Mandatory checks

When a repo adopts the shared workflow with caller job name `ci`, these status checks must pass before merging to the default branch:

| Check name | What it runs |
|---|---|
| `ci / poetry-check` | `poetry check` + lockfile present |
| `ci / lint` | `ruff check` + `ruff format --check` |
| `ci / typecheck` | `mypy` |
| `ci / test` | `pytest` |
| `ci / security` | `pip-audit` |

Details and ruleset import: [docs/REQUIRED_CHECKS.md](docs/REQUIRED_CHECKS.md).

## Contents

| Path | Purpose |
|---|---|
| [.github/workflows/python-ci.yml](.github/workflows/python-ci.yml) | Reusable `workflow_call` CI (five jobs) |
| [.github/actions/setup-poetry/](.github/actions/setup-poetry/) | Composite action: checkout, Poetry, cache, install |
| [rulesets/main-protection.json](rulesets/main-protection.json) | Importable ruleset template |
| [docs/ADOPT.md](docs/ADOPT.md) | Per-repo onboarding |
| [docs/REQUIRED_CHECKS.md](docs/REQUIRED_CHECKS.md) | Canonical check names + ruleset steps |

## Account note

This is a **personal GitHub account**, not an Organization. Rulesets do **not** propagate automatically — import [rulesets/main-protection.json](rulesets/main-protection.json) in each repo after CI has run at least once. If you later move to a GitHub Organization, the same workflow and check names work with a single org-level ruleset.

## Consumer example

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    uses: achaykovsky/.github/.github/workflows/python-ci.yml@main
    with:
      python-version: "3.11"
      pytest-args: '-m "not integration"'
```

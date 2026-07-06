# Adopt shared CI in a Python/Poetry repo

Estimated time: 10–15 minutes per repo.

## Prerequisites

- Poetry project with `pyproject.toml` and `poetry.lock` committed
- Default branch is `main` (or adjust the ruleset — see [REQUIRED_CHECKS.md](REQUIRED_CHECKS.md))
- GitHub Actions enabled on the repo

## 1. Add the CI workflow

Create `.github/workflows/ci.yml` in the target repo:

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

**Caller job id must be `ci`** so required check names match [REQUIRED_CHECKS.md](REQUIRED_CHECKS.md).

### Optional inputs

| Input | When to change |
|---|---|
| `python-version` | Repo requires a different Python |
| `pytest-args` | Different markers or test path |
| `run-mypy: false` | Legacy repo without type hints yet |
| `working-directory` | Poetry project in a monorepo subfolder |

Example for untyped legacy repo:

```yaml
jobs:
  ci:
    uses: achaykovsky/.github/.github/workflows/python-ci.yml@main
    with:
      run-mypy: false
```

When `run-mypy` is `false`, remove `ci / typecheck` from the ruleset for that repo only.

## 2. Add dev dependencies

Add to `pyproject.toml`:

```toml
[tool.poetry.group.dev.dependencies]
pytest = "^8.0"
ruff = "^0.8"
mypy = "^1.13"
pip-audit = "^2.7"
```

Then regenerate the lockfile:

```bash
poetry lock
poetry install
```

### Minimal tool configuration

Add to `pyproject.toml` if not already present:

```toml
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "UP"]

[tool.mypy]
python_version = "3.11"
strict = false
warn_return_any = true
warn_unused_configs = true
```

Adjust `mypy` paths or `packages` if the project layout requires it.

## 3. Open a PR and wait for CI

1. Commit `ci.yml`, `pyproject.toml`, and `poetry.lock`.
2. Push a branch and open a PR to `main`.
3. Wait for all five jobs to complete (fix failures before enabling the ruleset).

## 4. Enable the ruleset

Follow [REQUIRED_CHECKS.md — Enable the ruleset](REQUIRED_CHECKS.md#enable-the-ruleset).

Import [rulesets/main-protection.json](../rulesets/main-protection.json) after the first CI run.

## 5. Confirm merge gate

- Push a commit that fails lint or tests → merge should be blocked.
- Fix and push → all `ci / …` checks green → merge allowed.

## Rollout checklist

- [ ] `.github/workflows/ci.yml` added (caller job `ci`)
- [ ] Dev deps in `pyproject.toml` + updated `poetry.lock`
- [ ] `ruff` / `mypy` config present (or `run-mypy: false`)
- [ ] PR opened; all five CI jobs green
- [ ] Ruleset imported and active
- [ ] Merge blocked verified with a failing check

## Non-Poetry repos

This workflow is Poetry-only. Java or other stacks need a separate reusable workflow (not in v1). Skip or fork for outliers like `java_exercises`.

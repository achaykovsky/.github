---
applyTo: "**/*.py,**/pyproject.toml,**/poetry.lock,.github/workflows/python-ci.yml,.github/actions/setup-poetry/**"
---

# Python / Poetry review guidance

Shared rules (priorities, rulesets, review style) live in `.github/copilot-instructions.md`. This file owns **Python-only** CI and code review rules.

## Required check names (critical)

Canonical names for `python-ci.yml` consumers (caller job id `ci`) are defined in [docs/REQUIRED_CHECKS.md](../../docs/REQUIRED_CHECKS.md#check-names). Do not copy that table here — link and keep rulesets in sync with it.

Review notes:

- If `run-mypy: false` is introduced or documented, call out that `ci / typecheck` may be skipped and the repo ruleset must drop that check (see REQUIRED_CHECKS).
- Job `name:` fields in `python-ci.yml` must stay aligned with the REQUIRED_CHECKS table.

## Consumer expectations

- `poetry.lock` committed
- dev deps: `pytest`, `ruff`, `mypy`, `pip-audit`
- optional `run-mypy: false` for untyped legacy repos

When workflow commands change (e.g., ruff/mypy invocations), verify [docs/ADOPT.md](../../docs/ADOPT.md) tool configuration examples still match.

## Composite action (`setup-poetry`)

- `install-dependencies: "false"` must skip `poetry install` but still checkout, install Poetry, and restore cache.
- Cache path must use `inputs.working-directory` so monorepo subpaths work.
- Cache key must include OS, Python version, and lockfile hash.
- All run steps that execute Poetry commands need `working-directory: ${{ inputs.working-directory }}`.

## Job design (`python-ci.yml`)

- `poetry-check` should validate lockfile presence and run `poetry check` without installing deps when possible.
- Conditional jobs (e.g., `if: ${{ inputs.run-mypy }}`) should document consumer impact on required checks.
- Reuse `achaykovsky/.github/.github/actions/setup-poetry@main` consistently across jobs.

## Security

- Do not disable `pip-audit` or lockfile checks without documented legacy opt-out.
- Flag hardcoded credentials in example code or workflow env.

## Application code (when reviewing `*.py`)

- Use parameterized queries / ORM — no SQL string concatenation with user input.
- Narrow exception handling; preserve cause chains; no bare `except:`.
- New behavior should have pytest coverage; edge cases and failure paths included.
- Type hints on public function signatures when the consumer repo runs mypy.

## Examples

```yaml
# Avoid — cache key ignores Python version bumps
key: venv-${{ hashFiles('poetry.lock') }}

# Prefer — invalidate on Python or lockfile change
key: venv-${{ runner.os }}-${{ inputs.python-version }}-${{ hashFiles(format('{0}/poetry.lock', inputs.working-directory)) }}
```

```python
# Avoid — broad catch, lost cause
try:
    result = fetch(id)
except Exception:
    return None

# Prefer — specific handling, chained cause
try:
    result = fetch(id)
except ValidationError as exc:
    raise ValueError(f"invalid id {id!r}") from exc
```

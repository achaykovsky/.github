# Required checks

Canonical status check names for repos that call [python-ci.yml](../.github/workflows/python-ci.yml) with caller job id **`ci`**.

## Check names

Copy these **exactly** when configuring a ruleset (spacing and slashes matter):

```
ci / poetry-check
ci / lint
ci / typecheck
ci / test
ci / security
```

| Check | Job | Command |
|---|---|---|
| `ci / poetry-check` | `poetry-check` | `poetry check`; `poetry.lock` must exist |
| `ci / lint` | `lint` | `ruff check .`; `ruff format --check .` |
| `ci / typecheck` | `typecheck` | `mypy .` (skipped when `run-mypy: false`) |
| `ci / test` | `test` | `pytest` with caller `pytest-args` |
| `ci / security` | `security` | `pip-audit` |

If you rename the caller job from `ci`, every check name changes (`my-job / lint`, etc.). Keep the caller job id as `ci` across all repos for consistent rulesets.

## Enable the ruleset

CI must run **at least once** on a pull request before GitHub lists these checks in the ruleset UI.

### Option A — GitHub UI

1. Open the repo → **Settings** → **Rules** → **Rulesets** → **New ruleset** → **Import a ruleset**.
2. Upload [rulesets/main-protection.json](../rulesets/main-protection.json).
3. Confirm all five check names appear under **Required status checks**.
4. Save with **Enforcement status: Active**.

### Option B — GitHub CLI

```bash
gh api repos/OWNER/REPO/rulesets \
  --method POST \
  --input rulesets/main-protection.json
```

Run from a clone of this repo, or pass the full path to `main-protection.json`.

## Ruleset behavior

The template enforces:

- Pull requests required before merging to the default branch
- All five status checks must pass
- Branch must be up to date with the base branch (`strict_required_status_checks_policy: true`)
- Force pushes and branch deletion blocked
- No bypass actors (admins cannot skip checks)

Merge method is not restricted (merge commits are allowed).

## Legacy `master` branch

The template targets `~DEFAULT_BRANCH` only. If a repo still uses `master`, either rename to `main` or duplicate the ruleset with an explicit `refs/heads/master` include.

## Verify enforcement

1. Open a PR with a deliberate lint or test failure.
2. Confirm merge is blocked with missing/failing required checks.
3. Fix the failure, push, and confirm all five checks pass and merge is allowed.

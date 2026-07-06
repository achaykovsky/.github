## Summary

<!-- What changed and why -->

## Test plan

- [ ] `poetry check` passes locally
- [ ] `poetry run ruff check .` and `poetry run ruff format --check .` pass
- [ ] `poetry run mypy .` passes (or N/A with `run-mypy: false` in CI)
- [ ] `poetry run pytest` passes with the same args as CI
- [ ] `poetry run pip-audit` reports no blocking vulnerabilities
- [ ] All five CI checks are green on this PR before merge

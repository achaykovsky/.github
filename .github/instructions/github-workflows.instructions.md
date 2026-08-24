---
applyTo: ".github/workflows/**,.github/actions/**"
---

# GitHub Actions review guidance (stack-agnostic)

Shared rules (priorities, rulesets, review style) live in `.github/copilot-instructions.md`. Stack-specific job and composite-action rules live in:

- `.github/instructions/python.instructions.md`
- `.github/instructions/go.instructions.md`

Do not duplicate stack-specific check names, cache keys, or job commands here. Canonical Python check names: [docs/REQUIRED_CHECKS.md](../../docs/REQUIRED_CHECKS.md).

## Contract stability

- Treat `workflow_call` inputs and composite action inputs as a public API.
- Breaking renames or removed inputs require simultaneous updates to `docs/ADOPT.md`, `docs/REQUIRED_CHECKS.md`, and consumer examples in `README.md`.
- Job `name:` fields define GitHub check names — treat them as immutable unless migrating rulesets.

## Permissions and pinning

- Default to least privilege (`contents: read`). Flag `write`, `id-token`, or `packages` unless clearly required.
- Prefer pinned major versions (`actions/checkout@v4`, `actions/setup-python@v5`, `actions/setup-go@v5`).
- Flag unpinned `@main` on third-party actions unless this repo intentionally dogfoods its own `@main` refs.

## Reusable workflow structure

- New or changed `workflow_call` inputs must have descriptions, sensible defaults, and backward-compatible defaults when possible.
- Concurrency groups should be scoped to workflow + ref with `cancel-in-progress: true`.
- Thread `working-directory` through checkout, cache paths, and run steps for monorepo support.
- Use explicit `shell: bash` in composite actions.

## Examples

```yaml
# Avoid — overly broad permissions
permissions:
  contents: write
  pull-requests: write

# Prefer — least privilege for CI
permissions:
  contents: read
```

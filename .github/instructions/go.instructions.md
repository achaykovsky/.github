---
applyTo: "**/*.go,**/go.mod,**/go.sum,.github/workflows/go-ci.yml,.github/actions/setup-go/**"
---

# Go review guidance

Shared rules (priorities, rulesets, review style) live in `.github/copilot-instructions.md`. This file owns **Go-only** CI and code review rules.

**Publication status:** `go-ci.yml` and `setup-go` are not published in this repo yet. Treat the check names and job design below as the **draft contract** — when the workflow ships, move canonical check names into `docs/REQUIRED_CHECKS.md` (Go section) and a Go ruleset template, then link here instead of duplicating tables.

## Required check names (critical)

Draft canonical names for `go-ci.yml` consumers (caller job id `ci`):

```
ci / mod-check
ci / lint
ci / vet
ci / test
ci / security
```

| Check | Typical command |
|---|---|
| `ci / mod-check` | `go mod verify`; `go.sum` committed; optional tidy diff check |
| `ci / lint` | `golangci-lint run` |
| `ci / vet` | `go vet ./...` |
| `ci / test` | `go test` with caller `go-test-args` (e.g. `-race`, `-short`) |
| `ci / security` | `govulncheck ./...` |

Review notes:

- If a job is made conditional (e.g., skip `vet` for legacy repos), document ruleset impact the same way as `run-mypy: false` for Python.
- Job `name:` fields in `go-ci.yml` must match this table until REQUIRED_CHECKS owns Go check names.

## Consumer expectations

- `go.mod` and `go.sum` committed
- `.golangci.yml` (or equivalent) for lint job
- tests runnable with `go test ./...`
- `govulncheck` available for the security job
- Go version pinned via `go` directive and workflow `go-version` input

When Go workflow commands change (e.g., lint flags, race detector defaults), verify adoption docs and example caller YAML still match.

## Composite action (`setup-go`)

- Install Go via `actions/setup-go@v5` with `go-version` / `go-version-file` from inputs.
- Cache module and build caches under `~/go/pkg/mod` and `~/.cache/go-build` (or action defaults).
- Cache key must include OS, Go version, and `go.sum` hash from `inputs.working-directory`.
- Thread `working-directory` through checkout, cache paths, and all `go` / `golangci-lint` / `govulncheck` steps.

## Job design (`go-ci.yml`)

- `mod-check` should run `go mod verify` and fail when `go.sum` is missing or stale vs `go mod tidy`.
- `lint` should run `golangci-lint run` with repo config (`.golangci.yml`).
- `vet` should run `go vet ./...` from the module root.
- `test` should accept caller `go-test-args` (e.g. `-race -count=1 ./...`).
- `security` should run `govulncheck ./...`.
- Reuse `achaykovsky/.github/.github/actions/setup-go@main` consistently across jobs.

## Security

- Do not disable `govulncheck` or `go mod verify` without documented legacy opt-out.
- SQL must use parameterized queries — never `fmt.Sprintf` for query text.
- No secrets, tokens, or PII in logs.
- Flag `math/rand` for security-sensitive tokens; prefer `crypto/rand`.

## Application code (when reviewing `*.go`)

- Never ignore errors (`_, err :=` without check).
- Wrap errors with `%w` when callers need `errors.Is` / `errors.As`.
- Use sentinel or typed errors — avoid string matching on error messages.
- No panics in library or HTTP handler code paths.
- Pass `context.Context` as the first parameter to I/O and RPC functions.
- Flag goroutine leaks (missing `WaitGroup`, unbounded channel sends).
- Prefer table-driven tests with `t.Run` subtests.
- `go.mod` / `go.sum` must stay in sync; flag manual dependency edits without `go mod tidy`.

## Examples

```yaml
# Avoid — Go cache key ignores go.sum / version changes
key: go-${{ runner.os }}

# Prefer — invalidate on Go version or dependency changes
key: go-${{ runner.os }}-${{ inputs.go-version }}-${{ hashFiles(format('{0}/go.sum', inputs.working-directory)) }}
```

```go
// Avoid — ignored error, lost cause
user, _ := repo.Find(ctx, id)

// Prefer
user, err := repo.Find(ctx, id)
if errors.Is(err, ErrNotFound) {
    return ErrNotFound
}
if err != nil {
    return fmt.Errorf("find user %q: %w", id, err)
}
```

```go
// Avoid — SQL injection
query := fmt.Sprintf("SELECT * FROM users WHERE id = %s", id)

// Prefer
row := db.QueryRowContext(ctx, "SELECT * FROM users WHERE id = ?", id)
```

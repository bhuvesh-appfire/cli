---
name: acceptance-testing
description: Conventions for the testscript-based acceptance suite in acceptance/ - distinct from AGENTS.md's unit-test guidance
type: convention
scope:
  - acceptance/**
updated: '2026-08-04'
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - acceptance/README.md
  - acceptance/acceptance_test.go
---

AGENTS.md's testing section (HTTP mocking, `testify`, table-driven tests) is about `pkg/cmd/**` unit tests. `acceptance/` is a different animal: `.txtar`-based blackbox tests built on `github.com/cli/go-internal/testscript`, gated behind the `acceptance` build tag so `go test ./...` skips them by default.

Run them with:
```bash
GH_ACCEPTANCE_HOST=<host> GH_ACCEPTANCE_ORG=<org> GH_ACCEPTANCE_TOKEN=<token> go test -tags=acceptance ./acceptance
```
They hit a *real* GitHub instance and create real resources - `GH_ACCEPTANCE_TOKEN` needs a classic PAT (fine-grained PATs lack required scopes) with permissions in `GH_ACCEPTANCE_ORG`.

Not strictly blackbox: `exec gh` inside a script delegates to a test binary that calls `ghcmd.Main()` directly (see `acceptance/acceptance_test.go`), avoiding a real build step while keeping coverage instrumentation.

## Custom testscript commands (defined in `acceptance_test.go`)

- `defer <cmd>` - registers cleanup to run at script end, even on failure. Always pair resource creation with a `defer ... delete ...` immediately after creation.
- `env2upper VAR=value` - uppercases a value into an env var (needed because GitHub Actions uppercases secret names).
- `replace <file> KEY=value` - interpolates placeholders into a file.
- `stdout2env VAR` - captures previous command's stdout into an env var.
- `jq-assert VAR '<jq-expr>' '<regexp>'` - evaluates jq against a JSON env var and regex-matches the result.
- `jq2env VAR '<jq-expr>' TARGET` - same but stores the result instead of asserting.

## Test isolation rules (from `acceptance/README.md`)

- Favor duplicating setup across scripts over introducing a new shared `testscript` command.
- Each script should own its whole resource lifecycle rather than sharing fixtures across scripts.
- Use the `RANDOM_STRING` env var (length-10 random string, injected per script) to avoid collisions on globally-visible resource names.

## Debugging

`GH_ACCEPTANCE_PRESERVE_WORK_DIR=true` and `GH_ACCEPTANCE_SKIP_DEFER=true` skip cleanup so you can inspect the working directory after a failed run. `-v` dumps full `testscript` env including tokens - only use it locally, never paste that output anywhere shared (`acceptance/README.md` notes token redaction in logs is not guaranteed comprehensive).

## Gotcha: `test.ExpectLines` is deprecated in favor of exact matches

Unrelated to `acceptance/` itself but adjacent: the top-level `test` package (`test/helpers.go`, still imported by 23+ `pkg/cmd/**` unit tests) marks `ExpectLines` as `// Deprecated: prefer exact matches for command output`. New unit tests should assert exact output strings (per AGENTS.md's `assert.Equal(t, "expected", actual)` pattern), not regex-line matching via `test.ExpectLines`.

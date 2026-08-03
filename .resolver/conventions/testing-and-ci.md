---
name: testing-and-ci
description: How CI actually invokes tests, beyond the `go test ./...` shown in AGENTS.md
type: convention
scope: global
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - .github/workflows/go.yml
  - acceptance/README.md
  - test/integration/attestation-cmd/run-all-tests.sh
---

`AGENTS.md` covers the day-to-day test/lint commands and the Options+Factory unit test pattern in depth;
this page only records what CI runs that AGENTS.md's examples don't show.

## Three distinct test surfaces in CI (`.github/workflows/go.yml`)

1. **`build` job** — `go test -race -tags=integration ./...` across ubuntu/windows/macos, then
   `go build -v ./cmd/gh`. Note the `-race` flag and `-tags=integration`, which AGENTS.md's plain
   `go test ./...` example doesn't include. `-tags=integration` gates a small set of real-network tests
   under `pkg/cmd/attestation/**` (files named `*_integration_test.go`) that are otherwise excluded from a
   default `go test ./...` run.
2. **`integration-tests` job** — builds the real binary with `make`, then runs
   `test/integration/attestation-cmd/run-all-tests.sh` against `GH_TOKEN: ${{ github.token }}`. This is a
   shell-driven black-box suite distinct from both unit tests and the `acceptance/` package.
3. **Acceptance tests** (`go:build acceptance`, run via `make acceptance` / `go test -tags acceptance
   ./acceptance`) — fully documented in [`acceptance/README.md`](../../acceptance/README.md); they need
   `GH_ACCEPTANCE_HOST`/`GH_ACCEPTANCE_ORG`/`GH_ACCEPTANCE_TOKEN` against a real GitHub instance and are
   not run in the standard `go.yml` workflow at all. If asked to "run the tests" for a change touching
   `acceptance/testdata/*.txtar`, don't assume `go test ./...` exercises it — it's excluded by the build
   tag, and running it for real requires credentials this environment won't have.

## Practical implication for changes

If a change touches `pkg/cmd/attestation/`, check whether existing tests are `_integration_test.go` (real
network) versus plain `_test.go` (mocked) before assuming `go test ./...` covers them.

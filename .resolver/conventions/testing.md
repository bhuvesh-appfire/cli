---
name: testing
description: Unit vs acceptance test split and where acceptance-test conventions live
type: convention
scope: global
updated: 2026-08-03 (IONE-959)
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - AGENTS.md
  - acceptance/README.md
  - acceptance/acceptance_test.go
---

`AGENTS.md`'s "Testing" section covers unit-test structure for commands (Options/run-function table tests, `httpmock.Registry`, `iostreams.Test()`, `testify` with `require` for errors). That's the pattern for the vast majority of tests in this repo.

A second, separate test suite exists for real-GitHub blackbox tests: `acceptance/`, gated behind `//go:build acceptance` so `go test ./...` never runs it. It's built on `go-internal/testscript` (`.txtar` scripts), requires `GH_ACCEPTANCE_HOST`/`GH_ACCEPTANCE_ORG`/`GH_ACCEPTANCE_TOKEN` env vars, and creates/deletes real resources on a live host. Full conventions (custom `testscript` commands like `defer`, `jq-assert`, `stdout2env`; test-isolation rules; debugging flags `GH_ACCEPTANCE_PRESERVE_WORK_DIR`/`GH_ACCEPTANCE_SKIP_DEFER`) are documented in [`acceptance/README.md`](../../acceptance/README.md) — read that before touching anything under `acceptance/`, don't try to reverse-engineer it from the `.txtar` files alone.

Do not add acceptance-style live-API tests as regular `_test.go` files outside `acceptance/`; regular unit tests must stub HTTP/git per `docs/project-layout.md`'s "How to write tests" section, never hit real services.

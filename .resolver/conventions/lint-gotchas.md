---
name: lint-gotchas
description: golangci-lint specifics not covered by AGENTS.md - which linters are intentionally off and why
type: convention
scope: global
updated: '2026-08-04'
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - .golangci.yml
---

AGENTS.md says to run `make lint` before committing but doesn't detail the config. `.golangci.yml` uses `linters.default: none` plus an explicit enable-list, so a linter not on this list produces zero findings regardless of what its default preset would normally check.

Enabled: `asasalint`, `asciicheck`, `bidichk`, `bodyclose`, `copyloopvar`, `durationcheck`, `exptostd`, `fatcontext`, `gocheckcompilerdirectives`, `gochecksumtype`, `gocritic`, `gomoddirectives`, `goprintffuncname`, `govet` (with `httpresponse` check explicitly turned on), `ineffassign`, `nilerr`, `nolintlint`, `nosprintfhostport`, `reassign`, `unused`.

Explicitly **not** enabled yet (per an inline comment "too many issues, and confirm we need them"): `gosec`, `staticcheck`, `errcheck`. Do not assume a clean `make lint` run means there are no unchecked errors or no `staticcheck`-class issues - those categories aren't being linted at all right now. When adding new code, apply that judgment manually rather than relying on the linter to catch it.

`bodyclose` and `gosec` are excluded specifically for `_test\.go$` files (`.golangci.yml:31-36`) - test files don't need to close HTTP response bodies. `gocritic` has `appendAssign` and the whole `style` tag disabled.

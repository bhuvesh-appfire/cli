---
name: linting
description: Which golangci-lint linters are actually enabled - notably errcheck/staticcheck/gosec are not
type: convention
scope: global
updated: '2026-08-05'
captured_sha: 3ef13bbb2374fab317fc5c26f65b9be93fa9cb40
sources:
  - .golangci.yml
---

`make lint` runs `golangci-lint run ./...` per [`AGENTS.md`](../../AGENTS.md), configured by `.golangci.yml`. That config uses `linters.default: none` and enables an explicit allowlist (asasalint, asciicheck, bidichk, bodyclose, copyloopvar, durationcheck, exptostd, fatcontext, gocheckcompilerdirectives, gochecksumtype, gocritic, gomoddirectives, goprintffuncname, govet, ineffassign, nilerr, nolintlint, nosprintfhostport, reassign, unused).

`errcheck`, `staticcheck`, and `gosec` are deliberately **not** enabled - the config comments them as "to enable later due to too many issues". Don't assume unchecked errors or common staticcheck-class issues will be caught by CI; don't add `//nolint:errcheck`-style suppressions expecting a linter that isn't running, and don't turn these linters on as a "bonus improvement" in an unrelated PR since that would surface a large pre-existing backlog.

`gocritic` has the `style` tag and `appendAssign` check disabled. Test files (`_test.go`) are exempt from `bodyclose` and `gosec`.
